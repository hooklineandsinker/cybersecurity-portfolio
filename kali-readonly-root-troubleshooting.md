# Incident Write-Up: Kali Linux Root Filesystem Mounting Read-Only at Boot

**Host:** `MOUSE` (bare-metal Kali GNU/Linux Rolling, kernel 6.19.14-kali-amd64)
**Date:** June 19, 2026
**Category:** Systems Troubleshooting / Linux Boot Process / systemd

## Summary

After logging into a bare-metal Kali install, the root filesystem (`/`) was mounted read-only, breaking history logging, networking, the display manager, and other core services. Root cause was traced to a missing `systemd` unit symlink that normally remounts root read-write during boot. The fix restored normal boot behavior without requiring a filesystem repair or reinstall.

## Initial Symptom

On login, zsh reported:

```
zsh: locking failed for /home/kevin/.zsh_history: read-only file system: reading anyway
```

## Investigation

**Step 1 — Confirm the mount state**

```bash
mount | grep ' / '
# /dev/nvme0n1p2 on / type ext4 (ro,relatime,stripe=128)
```

Confirmed root was mounted `ro`.

**Step 2 — Rule out filesystem corruption**

```bash
sudo mount -o remount,rw /
dmesg | grep -i -E 'error|ext4|remount'
```

The remount succeeded cleanly, and `dmesg` showed no ext4 errors, no failed journal recovery, and no fsck warnings — ruling out disk corruption as the cause. A `systemd-sslh-generator` segfault appeared in the log but was determined to be a downstream symptom (it failed because the filesystem was already read-only), not the root cause.

**Step 3 — Review `journalctl` for the full boot sequence**

```bash
journalctl -b | grep -i -E 'fsck|clean|recover|mount' | head -40
```

This showed root mounting `ro` directly with no `systemd-fsck-root.service` activity at all — fsck never ran or attempted to run.

**Step 4 — Check fstab and kernel boot parameters**

```bash
cat /etc/fstab
cat /proc/cmdline
```

`/etc/fstab` was correctly configured (`errors=remount-ro`, `pass=1` on root). `/proc/cmdline` showed `ro quiet splash` — standard kernel behavior, since the kernel always mounts root `ro` initially and expects `systemd` to remount it `rw` later in boot via `systemd-remount-fs.service`.

**Step 5 — Isolate the actual failure point**

```bash
systemctl status systemd-remount-fs.service
journalctl -b -u systemd-remount-fs.service
```

`systemd-remount-fs.service` was `inactive (dead)` with **no journal entries at all** — it had never run this boot.

```bash
ls -la /usr/lib/systemd/system/sysinit.target.wants/ | grep -i remount
ls -la /etc/systemd/system/sysinit.target.wants/ | grep -i remount
```

Neither directory contained a symlink for `systemd-remount-fs.service`. Without that symlink, `sysinit.target` never pulls the unit into the boot sequence, so it silently never runs — leaving root permanently read-only after the kernel's initial `ro` mount.

## Root Cause

The `systemd-remount-fs.service` unit (a stock, vendor-shipped "static" unit with no `[Install]` section) relies on a symlink in `sysinit.target.wants/` to be activated at boot. That symlink was missing from this install, so the service that's responsible for flipping root from `ro` to `rw` during early boot never executed.

## Fix

Re-created the missing symlink manually (no internet access was available at the time to reinstall the `systemd` package, which would have restored it automatically):

```bash
sudo ln -s /usr/lib/systemd/system/systemd-remount-fs.service \
  /etc/systemd/system/sysinit.target.wants/systemd-remount-fs.service
sudo systemctl daemon-reload
sudo reboot
```

## Verification

After reboot:

```bash
mount | grep ' / '
# /dev/nvme0n1p2 on / type ext4 (rw,relatime,errors=remount-ro,stripe=128)
```

Root mounted `rw` automatically with no manual intervention. GUI/display manager came up normally.

As a follow-up integrity check, confirmed no other core `sysinit.target` units were affected:

```bash
ls /usr/lib/systemd/system/sysinit.target.wants/ | grep -E 'tmpfiles-setup|sysctl|udev-trigger|modules-load|random-seed'
```

All expected units (`systemd-modules-load`, `systemd-random-seed`, `systemd-sysctl`, `systemd-tmpfiles-setup`, `systemd-tmpfiles-setup-dev`, `systemd-udev-trigger`) were present and correctly linked. Only `systemd-remount-fs.service` was affected.

## Key Takeaways

- A read-only root filesystem doesn't always mean disk corruption — check `dmesg` for ext4 errors before assuming a hardware/fsck issue.
- The Linux boot sequence mounts root `ro` by design; a working `systemd-remount-fs.service` is what flips it to `rw`. If that unit's `sysinit.target.wants/` symlink is missing, the remount silently never happens.
- `journalctl -b -u <unit>` showing **zero entries** is a strong signal the unit never ran at all this boot — worth checking before assuming a service merely failed.
- When troubleshooting, downstream symptoms (segfaults, failed services, failed ACL writes) can be red herrings caused by the same upstream condition rather than separate problems.
