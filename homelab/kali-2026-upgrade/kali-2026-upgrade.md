# Kali Linux 2026.2 Upgrade — Bare Metal Lab Documentation

**Date:** June 14, 2026  
**Machine:** Bare metal laptop (hostname: MOUSE)  
**Previous Version:** Kali 2025.x (kali-rolling)  
**Target Version:** Kali 2026.2  
**Kernel Before:** Unknown  
**Kernel After:** `6.19.14+kali-amd64`

---

## Overview

This documents a full in-place upgrade of a bare metal Kali Linux installation to version 2026.2, including issues encountered and how they were resolved. This machine serves as my primary cybersecurity home lab for TryHackMe, CTF practice, and tool exploration.

---

## Pre-Upgrade Checklist

- Verified available disk space with `df -h` (811 GB available)
- Confirmed current OS version with `cat /etc/os-release`
- Noted this is a bare metal install — no VM snapshot available, so backed up home directory via rsync to external drive before proceeding

---

## Upgrade Steps

### 1. Update sources.list

```bash
echo "deb http://http.kali.org/kali kali-rolling main contrib non-free non-free-firmware" | sudo tee /etc/apt/sources.list
```

### 2. Run the upgrade

```bash
sudo apt update && sudo apt -y full-upgrade
```

This touched approximately 1,800+ packages including:
- `kali-linux-default 2026.2.11`
- `kali-desktop-xfce 2026.2.11`
- `kali-tools-top10 2026.2.11`
- `burpsuite 2026.3.2`
- `kali-menu 2026.2.6`
- New kernel initramfs generated: `6.19.14+kali-amd64`

### 3. Update shell config files

```bash
cp -vrbi /etc/skel/. ~/
```

Prompted to overwrite `/home/kevin/.bash_logout` — confirmed with `y`.

### 4. Reboot

```bash
[ -f /var/run/reboot-required ] && sudo reboot -f
```

---

## Issues Encountered

### Issue 1: Black screen after reboot
**What happened:** After the reboot command completed, the screen went black and stayed black.  
**Root cause:** Normal behavior — system was shutting down and restarting.  
**Resolution:** Waited ~60 seconds for BIOS splash and Kali boot sequence.

### Issue 2: Machine did not power back on
**What happened:** After the black screen, the machine appeared fully powered off. Pressing the power button had no effect.  
**Root cause:** Residual charge preventing POST (common on laptops after forced shutdown).  
**Resolution:** Held power button for 15 seconds to drain residual charge, then pressed power button normally. Machine booted successfully.

---

## Post-Upgrade Verification

```bash
cat /etc/os-release
```

```
PRETTY_NAME="Kali GNU/Linux Rolling"
NAME="Kali GNU/Linux"
VERSION_ID="2026.2"
VERSION="2026.2"
VERSION_CODENAME=kali-rolling
ID=kali
ID_LIKE=debian
```

```bash
uname -r
```

```
6.19.14+kali-amd64
```

✅ Upgrade confirmed successful.

---

## Additional Tools Installed Post-Upgrade

### XSStrike 3.1.6
Advanced XSS detection and exploitation tool. Added to Kali repos in 2026.1.

```bash
sudo apt install xsstrike
```

**Use case:** Web application penetration testing, particularly XSS vulnerability discovery and WAF bypass. Relevant to OWASP Top 10 TryHackMe rooms.

### Fluxion 6.28
WiFi social engineering and WPA2 auditing tool. Added to Kali repos in 2026.1.

```bash
sudo apt install fluxion
```

**Use case:** Wireless network security auditing via evil twin access points. Requires a wireless adapter with monitor mode and packet injection support. Noted for future lab use.

### Atomic-Operator (attempted — removed)
Purple team tool for running Atomic Red Team tests.

```bash
pip install atomic-operator --break-system-packages
# Installed but failed to run — ModuleNotFoundError: No module named 'pipes'
# Also caused dependency conflicts with theharvester, netexec, and other Kali tools
pip uninstall atomic-operator --break-system-packages
```

**Outcome:** Incompatible with current Python version. Removed to avoid breaking existing tools. Will revisit in a future release.

---

## What's New in Kali 2026.2

- Kernel upgraded to 6.19.14
- 2026 annual theme refresh (boot menu, login screen, desktop wallpapers)
- BackTrack Mode added to `kali-undercover` (20th anniversary of BackTrack Linux)
- Burp Suite updated to 2026.3.2
- 8 new tools added to repos: AdaptixC2, Fluxion, XSStrike, Atomic-Operator, and others
- 183+ existing tools updated

**Known issue:** SDR tools (gqrx, gr-air-modes) are broken in this release. Fix expected in 2026.3.

---

## Key Lessons Learned

- On bare metal, always `rsync` home directory to external drive before a major upgrade — no snapshot fallback available
- Large upgrade gaps (several months) touch 1,800+ packages; weekly updates keep this to 20–50 at a time
- `polkitd` upgrade requires a reboot to fully apply — don't skip it
- Residual charge issues on laptops after forced shutdowns can be cleared by holding the power button for 15 seconds
- Verify pip installs for dependency conflicts before committing — `--break-system-packages` can conflict with existing Kali tools

---

## Lab Environment

| Component | Details |
|-----------|---------|
| OS | Kali Linux 2026.2 (kali-rolling) |
| Kernel | 6.19.14+kali-amd64 |
| Desktop | XFCE |
| Install type | Bare metal |
| Primary use | TryHackMe, CTF practice, tool exploration |
| Certifications in progress | CompTIA Security+ (SY0-701) |

---

## References

- [Official Kali 2026.1 Release Notes](https://www.kali.org/blog/kali-linux-2026-1-release/)
- [Kali Linux Updating Documentation](https://www.kali.org/docs/general-use/updating-kali/)
- [XSStrike GitHub](https://github.com/s0md3v/XSStrike)
