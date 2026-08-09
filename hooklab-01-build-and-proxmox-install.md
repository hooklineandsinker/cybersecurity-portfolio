# HOOKLAB-01: Home Lab Virtualization Host — Build & Proxmox VE Deployment

**Completed:** August 2026
**Purpose:** Dedicated, always-on type-1 hypervisor host for Active Directory, SIEM, and blue-team lab work — separated from my daily-driver Kali laptop.

---

## Why Build This

My Active Directory lab (`hooklab.local` — HookDC + HookClient) and Splunk instance were running in VirtualBox on my daily-driver Kali laptop. That worked for one lab at a time, but hit a hard ceiling:

- **16GB RAM ceiling** — running the AD domain, a SIEM, and a Kali attack box concurrently left nothing for the host OS
- **Type-2 hypervisor overhead** — VirtualBox running on top of a full desktop OS wastes resources that should go to the guests
- **No persistence** — I couldn't leave a domain controller running indefinitely without tying up the machine I use for daily work and job applications
- **Laptop thermals** — sustained multi-VM load throttles in a laptop chassis

The goal was a dedicated box I could break, rebuild, and leave running 24/7, managed headlessly over the network.

---

## Hardware

| Component | Model | Key Specs | Price |
|---|---|---|---|
| CPU | AMD Ryzen 5 5600G | 6C/12T, 3.9GHz base / 4.4GHz boost, Zen 3, 7nm, 16MB L3, integrated Radeon graphics, AMD-V, 65W TDP | $191.95 |
| Motherboard | GIGABYTE B550M K | AM4, B550 chipset, Micro-ATX, 4× DDR4 (128GB max), dual M.2, PCIe 4.0 x16, Realtek GbE | $74.99 |
| Memory | Timetec Pinnacle Konduit 32GB | 2×16GB DDR4-3200, CL16-18-18-38, dual rank | $189.99 |
| Storage | Patriot P400 Lite 1TB | M.2 2280, PCIe Gen4 x4, NVMe 2.0, 3,500MB/s read / 2,700MB/s write | $149.99 |
| PSU | ASRock Challenger CL-550B | 550W, 80+ Bronze, non-modular, 5-yr warranty | $49.99 |
| Case | Cooler Master MasterBox Q300L (refurb) | Micro-ATX mini tower, 387×230×381mm, 159mm cooler clearance, magnetic dust filters | $49.99 |
| **Total** | | | **~$707** |

### Component Selection Rationale

**RAM over cores.** For concurrent VM workloads at this scale, memory is the binding constraint before CPU. 32GB across two dual-rank sticks in dual-channel (slots A2/B2) was the priority; 6C/12T is sufficient headroom for 4-6 concurrent guests.

**AM4/DDR4 over AM5/DDR5.** I priced both platforms during a 2026 DRAM supply crunch. DDR4-3200 32GB kits were running ~$190 while equivalent DDR5 was $399-479 — the shortage hit DDR5 significantly harder. AM4 was the clear value platform at the time of purchase.

**G-series CPU, no discrete GPU.** The 5600G's integrated graphics provides display output for BIOS access and initial setup. A GPU-less build eliminates the largest heat and power variable, keeps the PSU running at low load, and removes case clearance concerns entirely. The non-G Ryzen 5 5600 would have produced no video output on this board without adding a discrete card.

**PSU headroom.** 550W against a 65W CPU with no GPU means the unit runs well under a third of rated capacity — appropriate margin for a machine intended for 24/7 uptime.

---

## Assembly Notes

Standard build order: CPU, cooler, RAM, and M.2 SSD installed on the board outside the case, then board mounted and cabled.

**Issues encountered and resolved:**

- **Cooler mounting style.** The stock Wraith Stealth shipped in the screw-down configuration rather than the more common clip-on style. This required removing the two black plastic AM4 retention brackets (4 screws) before mounting, tightening the four spring screws in a diagonal X pattern to maintain even die pressure.
- **Front panel header pinout.** The case manual only illustrated the connector side without motherboard pin assignments. Resolved by reading the silkscreen printed directly on the PCB adjacent to the F_PANEL header: top row `+PLED−` / `+PW−` / `+SPEAK−`, bottom row `+HD−` / `−RES+` / `−CI+` / `+PWR LED−`. Worth noting the `−RES+` group is printed in reverse polarity order relative to its neighbors — irrelevant for a momentary switch, but a good reminder to read rather than assume.
- **3-pin case fan into SYS_FAN1.** No PWM control, so the fan runs at a fixed speed. Acceptable for a lab host; can be tuned to DC/voltage mode in BIOS if noise becomes an issue.

---

## First Boot & BIOS Configuration

The system POSTed on the first power-on attempt.

**Boot sequence observations:**

1. **fTPM/PSP NV prompt** — AMI BIOS detected a new CPU and offered to reset the firmware TPM. Selected `Y` (reset). No existing OS or encrypted volumes, so no recovery-key risk.
2. **CMOS reset notice** — expected consequence of the fTPM reset.
3. **"Boot failure detected"** — this looks alarming but was a configuration inconsistency following the CMOS clear, not a hardware fault. The dialog itself reported CPU at 3900.0MHz and memory at 3204.51MHz, both correct. Selected "Enter BIOS" rather than loading defaults, to verify and configure in a single pass.

**Verification in BIOS (Easy Mode):**

| Check | Result |
|---|---|
| CPU | AMD Ryzen 5 5600G with Radeon Graphics ✓ |
| RAM total | 32GB / 32768MB ✓ |
| DIMM population | A2: 16GB, B2: 16GB — correct dual-channel config ✓ |
| Memory frequency | 3209.92MHz (rated speed without XMP) ✓ |
| Storage | Patriot P400L 1000GB detected ✓ |
| CPU_FAN | 1679 RPM ✓ |
| SYS_FAN1 | 497 RPM ✓ |
| CPU temp (idle) | 41.0°C ✓ |
| BIOS version | F7 (10/28/2025) ✓ |

**Configuration change — the one that matters:**

`Tweaker → Advanced CPU Settings → SVM Mode → Enabled`

SVM Mode is AMD's CPU virtualization extension (help text: *"Enable/Disable CPU Virtualization"*). Without it, KVM cannot use hardware-assisted virtualization and guest performance degrades severely. This is disabled by default on this board.

XMP was left disabled — the memory already trains to its rated 3200MHz, so enabling the profile offered no benefit.

**BIOS compatibility note:** I checked GIGABYTE's CPU support list before ordering, since G-series (Cezanne) APUs historically required BIOS updates on some AM4 boards. The B550M K's first BIOS release (F1, Jan 2023) already shipped with AGESA V2 1.2.0.7, which postdates Cezanne's 2021 launch — meaning every BIOS this board has ever shipped with supports the 5600G natively. The board arrived on F7 and recognized the CPU immediately. Q-Flash Plus was available as a fallback but not needed.

---

## Proxmox VE 9.2 Installation

### Preparing Installation Media

1. Downloaded the Proxmox VE 9.2 ISO from the official downloads page.
2. **Verified the SHA256 checksum before flashing:**
   ```powershell
   Get-FileHash proxmox-ve_9.2-1.iso -Algorithm SHA256
   ```
   Compared against the hash published alongside the download. This step caught a real mistake: my first download was the **ARM64** build (`proxmox-ve_9.2-1-arm64.iso`, 1.50GB). The checksum verified correctly — but against the wrong file. The ARM64 image will not boot on x86-64 hardware. The correct image is the plain `Proxmox VE 9.2 ISO Installer` (1.71GB, SHA256 beginning `4e88fe41...`).

   *Lesson: checksum verification confirms integrity, not that you downloaded the right artifact. Verify the architecture too.*
3. Flashed to USB with Balena Etcher. Windows subsequently prompted "You need to format the disk in drive D:" and "The volume does not contain a recognized file system" — both expected. The drive now carries a Linux/ISO filesystem Windows cannot parse. Clicking "Format disk" would have destroyed the installer.

### Installation

Booted via one-time boot menu (F12). Installer confirmed **EFI boot mode** and detected the NVMe drive.

| Setting | Value |
|---|---|
| Filesystem | ext4 |
| Target disk | `/dev/nvme0n1` (931.51GiB, Patriot P400L) |
| Timezone | America/New_York |
| Hostname | `hooklab` |
| IP (CIDR) | `192.168.40.50/24` |
| Gateway | `192.168.40.1` |
| DNS | `192.168.40.1` |

**Network planning:** The installer pre-populates `192.168.100.2/192.168.100.1` as placeholders. These are not real defaults and will produce an unreachable host if accepted blindly. I ran `ipconfig` on another machine on the LAN to identify the actual subnet (`192.168.40.x`) and gateway, then assigned a **static IP well outside the router's DHCP pool** — the DHCP-assigned host I checked had received `.250`, so `.50` sits clear of the dynamic range. A static address means the management interface stays at a predictable URL across reboots.

Install completed and auto-rebooted to:

```
https://192.168.40.50:8006
```

### Post-Install: Repository Configuration

A fresh Proxmox install defaults to the **enterprise** repositories, which require a paid subscription. The first `apt update` fails with `401 Unauthorized`. Switching to the free no-subscription repo is required.

**Proxmox 9.x uses the deb822 `.sources` format**, not the older `.list` files most guides still reference. Confirmed with:

```bash
ls /etc/apt/sources.list.d
# ceph.sources  debian.sources  pve-enterprise.sources
```

**Disable the enterprise PVE repo** — add `Enabled: false` to `/etc/apt/sources.list.d/pve-enterprise.sources`:

```
Enabled: false
Types: deb
URIs: https://enterprise.proxmox.com/debian/pve
Suites: trixie
Components: pve-enterprise
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
```

**Add the no-subscription repo** — create `/etc/apt/sources.list.d/pve-no-subscription.sources`:

```
Types: deb
URIs: http://download.proxmox.com/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
```

**Disable the Ceph enterprise repo** — `ceph.sources` also points at `enterprise.proxmox.com` and throws its own 401. Ceph is distributed storage for multi-node clusters; irrelevant to a single-node host with one drive. Same fix — add `Enabled: false`.

**Update:**

```bash
apt update && apt dist-upgrade -y
```

Completed cleanly to PVE 9.2.10. A new kernel (7.0.14-11-pve) was installed, requiring a reboot.

**Benign messages in the upgrade output** (present on every Proxmox update, not errors):
- `File descriptor 3 leaked on vgs invocation` — cosmetic LVM warning
- `watchdog-mux.service is a disabled or a static unit` — HA clustering component, unused on a single node
- `No /etc/kernel/proxmox-boot-uuids found, skipping ESP sync` — expected on a GRUB/ext4 install; that path applies to ZFS-root systems
- `modprobe: FATAL: Module shpchp not found` (during installer boot) — optional modules absent from this kernel build

---

## Final State

- Proxmox VE 9.2.10 running bare-metal on `/dev/nvme0n1`
- Node `hooklab` reachable at `https://192.168.40.50:8006`
- Hardware virtualization (SVM) enabled and available to KVM
- Fully headless — monitor, keyboard, and mouse disconnected after initial setup; all management via browser
- Repositories on the free no-subscription channel, system current

---

## Lessons Learned

1. **Verify the artifact, not just the hash.** A valid checksum on the wrong architecture is still the wrong file. Confirm you downloaded what you intended before flashing.
2. **Read the board, not the internet.** The case manual and generic pinout tables both failed me on the F_PANEL header. The authoritative source was silkscreened on the PCB two inches from the connector.
3. **Alarming messages are often normal.** "Boot failure detected," Windows demanding to format a freshly-flashed USB, `modprobe: FATAL`, LVM file descriptor leaks — all expected. Reading what the message actually reports (correct clock speeds, correct memory frequency) beats reacting to the headline.
4. **Installer defaults are not your defaults.** The placeholder network config would have produced a fully installed but unreachable host. Two minutes with `ipconfig` beforehand avoided that entirely.
5. **Documentation formats drift.** Most current Proxmox guides still reference `.list` files. The 9.x move to deb822 `.sources` means following a guide verbatim creates an empty file and silently fails to fix anything.

---

## Next Steps

- [ ] Rebuild `hooklab.local` AD environment (HookDC + HookClient) as Proxmox guests
- [ ] Deploy persistent Splunk instance for ongoing detection engineering
- [ ] Stand up an attack range: vulnerable target + C2 + SIEM detection pipeline
- [ ] Evaluate Elastic/Kibana alongside Splunk for comparison
- [ ] Add second 120mm case fan for exhaust (currently intake-only)
- [ ] Consider second NVMe drive to separate OS from VM storage

---

*Part of my [cybersecurity portfolio](https://github.com/hooklineandsinker/cybersecurity-portfolio).*
