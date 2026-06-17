# Home Lab Build: Active Directory Domain Controller + Client

## Overview

This write-up documents the build of a two-machine Active Directory lab environment using VirtualBox, running on a Kali Linux host. The goal was to create an isolated, snapshot-able network for practicing AD administration, log generation, and (in a future phase) SOC-style monitoring with a SIEM.

**Environment:**
- Host OS: Kali Linux (bare metal, Dell laptop, i7-1355U / 16GB RAM)
- Hypervisor: Oracle VirtualBox
- Domain Controller: `HookDC` — Windows Server 2025 Standard Evaluation
- Client: `HookClient` — Windows 11 Pro
- Domain: `hooklab.local`
- Network: Isolated VirtualBox Host-only network (`vboxnet0`, 192.168.56.0/24)

## Architecture

The lab uses a host-only network rather than NAT or bridged networking, which keeps the domain traffic fully isolated from the home network while still allowing the Kali host to reach both VMs directly (useful for future attack/detection exercises). Each VM also retained a temporary NAT adapter during OS installation to handle activation and downloading prerequisites, which was later left in place on the DC but skipped entirely on the client once host-only connectivity was confirmed.

| Machine | Role | IP | OS |
|---|---|---|---|
| HookDC | Domain Controller / DNS | 192.168.56.10 (static) | Windows Server 2025 |
| HookClient | Domain-joined workstation | DHCP (vboxnet0) | Windows 11 Pro |
| Kali (host) | Attacker/management box | 192.168.56.1 | Kali Linux |

## Build Steps

### 1. Host-only network setup
Created a Host-only Network adapter (`vboxnet0`) in VirtualBox's Network Manager, configured with a `192.168.56.1/24` range and DHCP enabled for `.101–.254`, leaving `.2–.100` free for static assignments.

### 2. Domain Controller (HookDC)
- Created a VM with 4096 MB RAM, 2 CPUs, 71GB dynamically allocated disk
- Installed Windows Server 2025 Standard Evaluation (Desktop Experience)
- Added a second network adapter for the host-only network alongside the temporary NAT adapter used during install
- Assigned a static IP (`192.168.56.10`) to the host-only adapter
- Installed the Active Directory Domain Services role via Server Manager
- Promoted the server to a new forest, `hooklab.local`, with DNS automatically configured on the same box

### 3. Client (HookClient)
- Created a VM with 4096 MB RAM, 2 CPUs, EFI/TPM 2.0/Secure Boot enabled (required for Windows 11)
- Installed Windows 11 Pro with no NAT adapter — host-only only, by design, to confirm the lab network alone was sufficient
- Set the client's DNS manually to `192.168.56.10` so it could resolve the domain
- Joined the client to `hooklab.local` via System Properties → Change → Domain
- Verified the join by logging in with `HOOKLAB\Administrator`

### 4. Snapshots
Took a baseline snapshot of both VMs immediately after a confirmed working domain join, giving a safe restore point before any further configuration or intentional breakage for practice scenarios.

## Troubleshooting Notes

A few issues came up during the build that are worth recording, since they're common gotchas with this kind of setup:

- **VirtualBox "Finish" button greyed out during VM creation.** Caused by VirtualBox auto-re-checking "Proceed with Unattended Installation" after an ISO is selected, even when it was previously unchecked. Fix: re-uncheck it under the VM name/OS section.
- **"Same UUID as an existing virtual machine" error.** Triggered by trying to re-add a VM through File → Add that VirtualBox had already auto-registered after clicking Finish. The VM was already present in the main list; no action was needed beyond cancelling the duplicate add.
- **Windows 11 EFI boot not starting automatically.** With EFI enabled, VirtualBox requires a keypress during the "Press any key to boot from CD or DVD" prompt within a short window. Missing it drops into the EFI Boot Manager, from which the UEFI CD-ROM entry can be selected manually to continue.
- **Windows 11 OOBE blocking on "Let's connect you to a network."** Since the client had no NAT/internet adapter, setup demanded a network connection with no visible bypass option. Resolved via Shift+F10 → `oobe\bypassnro`, which reboots into OOBE with a working "I don't have internet" path, enabling local account setup.
- **Server Manager dashboard showing "Roles: 0" after DC promotion and reboot.** A stale cache issue — manually refreshing Server Manager (the circular arrow icon) revealed the AD DS and DNS roles were in fact installed correctly.
- **Domain join "restart" not triggering automatically.** Clicking OK on the welcome-to-domain message only dismisses that dialog; System Properties still needs to be closed, and the resulting prompt explicitly answered with "Restart Now."

## Skills Demonstrated

- VirtualBox network configuration (host-only networking, multi-adapter VMs)
- Windows Server 2025 installation and Active Directory Domain Services deployment
- Active Directory forest creation and DNS role configuration
- Windows 11 client deployment and domain join
- Troubleshooting EFI/UEFI boot behavior, Windows OOBE restrictions, and VirtualBox VM registration quirks
- Snapshot-based environment management for safe, repeatable lab work

## Next Steps

With the domain controller and client confirmed working, the next phase of this lab is adding a SIEM (Wazuh or Security Onion) as a third VM to begin collecting and analyzing logs from both machines — moving from infrastructure setup into active blue-team monitoring practice.
