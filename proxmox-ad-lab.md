# Building an Isolated Active Directory Lab on Proxmox VE

**Migrating a Windows domain lab from VirtualBox to dedicated bare-metal virtualization**

---

## Overview

This project documents the build of a dedicated, always-on Active Directory lab environment on Proxmox VE, replacing a laptop-hosted VirtualBox setup. The finished environment provides an isolated Windows domain for defensive security practice — log analysis, detection engineering, adversary emulation, and incident response — with snapshot-based rollback so the environment can be intentionally broken and restored in seconds.

**Outcome:** A functioning single-forest AD domain (`hooklab.local`) with a Windows Server 2025 domain controller and a domain-joined Windows 11 Pro workstation, running on an air-gapped virtual bridge with no path to the home network.

---

## Architecture

| Component | Detail |
|---|---|
| **Hypervisor** | Proxmox VE 9.2.10, node `hooklab` |
| **Host hardware** | AMD Ryzen 5 5600G |
| **Management** | 192.168.40.50 on `vmbr0` (home LAN) |
| **Lab network** | `vmbr1` — 10.10.10.0/24, no physical NIC attached |
| **Admin workstation** | Kali Linux laptop, SSH key auth to hypervisor |

### Virtual machines

| VM | ID | OS | IP | Role |
|---|---|---|---|---|
| HookDC | 100 | Windows Server 2025 Datacenter (Desktop Experience) | 10.10.10.10 | Domain controller, DNS, KDC |
| HookClient | 101 | Windows 11 Pro 25H2 | 10.10.10.20 | Domain-joined workstation |

### Network isolation design

The lab bridge is defined with `bridge-ports none`, meaning it exists only in software with no uplink to physical hardware. The Proxmox host holds 10.10.10.1 on that bridge and acts as the gateway.

```
auto vmbr1
iface vmbr1 inet static
    address 10.10.10.1/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0
#   post-up   iptables -t nat -A POSTROUTING -s 10.10.10.0/24 -o vmbr0 -j MASQUERADE
#   post-down iptables -t nat -D POSTROUTING -s 10.10.10.0/24 -o vmbr0 -j MASQUERADE
```

The two commented MASQUERADE rules are a deliberate **air-gap toggle**. Uncommenting them and reloading grants the lab outbound internet access for updates and tool downloads; re-commenting seals it. This matters for any future work involving live malware samples — the default state is isolated, and connectivity is an explicit, reversible decision rather than an assumption.

---

## Design decision: migrate or rebuild?

The existing AD lab ran under VirtualBox on the Kali laptop. Two options were available: export and convert the existing VDI disk images, or rebuild the domain from scratch on Proxmox.

**Rebuild was chosen.** The reasoning:

- **Firmware mismatch.** The VirtualBox VMs were BIOS-based. Windows 11 requires UEFI, Secure Boot, and TPM 2.0 — the client VM could not have been migrated cleanly regardless of effort.
- **Storage controller conversion.** Migrating would have required pre-injecting VirtIO drivers before export, then booting on emulated SATA and performing a driver swap to reach VirtIO SCSI. Skipping that sequence produces `INACCESSIBLE_BOOT_DEVICE`.
- **Evaluation licensing.** A rebuild restarts the Server 2025 180-day evaluation clock rather than inheriting a partially consumed one.
- **Repetition value.** Building a forest from scratch is itself the skill being practiced.

The migration path carried several hours of work and multiple failure points to arrive at a compromised configuration. The rebuild produced a correct one in comparable time.

---

## Build process

### 1. Hypervisor access and hardening

Key-based SSH from the admin workstation to the hypervisor, with an ED25519 keypair and a client-side config alias:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/hooklab
ssh-copy-id -i ~/.ssh/hooklab.pub root@192.168.40.50
```

```
Host hooklab
    HostName 192.168.40.50
    User root
    IdentityFile ~/.ssh/hooklab
    IdentitiesOnly yes
```

Once key authentication was confirmed, `sshd_config` was hardened with `PermitRootLogin prohibit-password` and `PasswordAuthentication no`. The Proxmox web GUI shell was kept open throughout as an out-of-band recovery path — a habit worth keeping whenever modifying the service that provides your own access.

### 2. Network configuration

The `vmbr1` stanza above was added to `/etc/network/interfaces` and applied with `ifreload -a`. Network edits were performed from the GUI shell rather than the SSH session, since a misconfigured bridge can sever SSH but not the console.

Verification showed `state UNKNOWN` on the interface — expected and correct for a bridge with no physical ports, since the kernel has no carrier to report.

### 3. Installation media

Three ISOs were staged to `/var/lib/vz/template/iso`:

- **Windows Server 2025** (evaluation, 180 days)
- **Windows 11 25H2** multi-edition (installed unactivated — fully functional for lab use)
- **virtio-win 0.1.285** — paravirtualized drivers

The VirtIO driver ISO is mandatory, not optional. Windows has no built-in driver for the VirtIO SCSI controller, so without it the installer presents an empty disk list with no explanation.

### 4. VM provisioning

Both VMs share a common baseline:

| Setting | Value | Reasoning |
|---|---|---|
| Machine type | `q35` | Modern PCIe chipset |
| BIOS | OVMF (UEFI) + EFI disk, pre-enrolled keys | Required for Secure Boot |
| SCSI controller | VirtIO SCSI single | Paravirtualized performance |
| Disk | SCSI, discard + SSD emulation + iothread | TRIM passthrough to thin storage |
| CPU type | `host` | Exposes native instruction set |
| Ballooning | Disabled | Predictable memory for a lab |
| QEMU guest agent | Enabled | Filesystem-consistent snapshots |
| Network | `vmbr1`, VirtIO model | Isolated bridge |

HookClient additionally received a **TPM v2.0 state device**, without which Windows 11 setup refuses to proceed.

### 5. Domain controller promotion

```powershell
# Static addressing — DNS points at itself
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.10.10.10 `
                 -PrefixLength 24 -DefaultGateway 10.10.10.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 127.0.0.1

Rename-Computer -NewName "HookDC" -Restart

# Role installation
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

# Forest creation
Install-ADDSForest -DomainName "hooklab.local" `
                   -DomainNetbiosName "HOOKLAB" `
                   -InstallDns:$true -Force:$true
```

Renaming *before* promotion avoids an extra reboot and a mismatched DC name in the directory.

**Verification:**

```
DNSRoot      : hooklab.local
NetBIOSName  : HOOKLAB
DomainMode   : Windows2025Domain

DC:          \\HookDC.hooklab.local
Address:     \\10.10.10.10
Forest Name: hooklab.local
Flags:       PDC GC DS LDAP KDC TIMESERV GTIMESERV WRITABLE
             DNS_DC DNS_DOMAIN DNS_FOREST CLOSE_SITE FULL_SECRET
```

The flags confirm the DC holds the PDC emulator FSMO role, serves as global catalog, runs the Kerberos KDC, and is authoritative for the domain's DNS zone.

Two DNS delegation warnings appeared during promotion. These are expected on a first DC — there is no parent zone above `hooklab.local` in which to create a delegation record, and the warning text itself states no action is required.

### 6. Client provisioning and domain join

```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.10.10.20 `
                 -PrefixLength 24 -DefaultGateway 10.10.10.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.10.10.10
```

**The single most important setting in the entire build** is the client's DNS server. It must point at the domain controller and nothing else. AD clients locate domain services through SRV records published in DNS; a client pointed at any other resolver will fail to join with errors that look like a dozen unrelated problems.

Connectivity was validated before attempting the join:

```
Test-NetConnection 10.10.10.10 -Port 389

RemoteAddress    : 10.10.10.10
RemotePort       : 389
SourceAddress    : 10.10.10.20
TcpTestSucceeded : True
```

Testing LDAP on 389 directly is more useful than ICMP here, since the Windows firewall may drop pings while domain services remain fully reachable.

**Result:**

```
Full computer name: HookClient.hooklab.local
Domain:             hooklab.local
```

### 7. Snapshots

Both VMs were snapshotted immediately after reaching a known-good state:

| VM | Snapshot | State captured |
|---|---|---|
| HookDC | `clean-dc-promoted` | Fresh forest, no users or GPOs |
| HookClient | `clean-domain-joined` | Joined, no additional software |

Snapshot output confirmed `freeze guest filesystem` / `thaw guest filesystem` operations, meaning the QEMU guest agent quiesced the filesystem for a consistent capture rather than a crash-consistent copy. The HookClient snapshot correctly included `tpmstate0` alongside the OS and EFI volumes — omitting TPM state would break the Windows 11 installation on rollback.

RAM was deliberately excluded. Disk-only snapshots restore to a clean boot rather than a mid-session memory state, which is the desired behavior for a rollback point.

---

## Problems encountered and resolved

Documenting failures is more useful than presenting a frictionless build. Each of these is a real trap.

### Commands executed against the wrong host

An `scp` intended to copy an ISO from the laptop to the hypervisor was run from inside an active SSH session, causing `~/Downloads` to resolve on the remote host instead of the local one.

**Resolution:** Exit the session or use a second terminal. **Lesson:** Check the shell prompt before running any command involving local paths.

### Driver ISO attached to the controller it provides drivers for

The VirtIO driver ISO was initially attached to the VirtIO SCSI bus. This creates a circular dependency — Windows cannot read the disc containing the SCSI driver without already having the SCSI driver.

**Resolution:** Attach the driver ISO to **IDE**, which Windows supports natively out of the box.

### UEFI boot fall-through to PXE

First boot attempt terminated in `>>Start PXE over IPv4` after failing both CD devices.

```
BdsDxe: failed to start Boot0002 "UEFI QEMU DVD-ROM" ... Time out
BdsDxe: failed to load Boot0004 "UEFI QEMU QEMU HARDDISK" ... Not Found
```

**Cause:** The virtio-win ISO is not a bootable image and has no EFI bootloader. With it and the network adapter both enabled as boot candidates, the firmware exhausted the list and fell through to network boot.

**Resolution:** In the boot order, disable `ide0` (driver ISO) and `net0` as boot devices, leaving only the installer ISO and the target disk. Unchecking a device removes it as a *boot candidate* without unmounting it — the driver disc remains fully accessible to Windows setup.

### Empty disk selection screen

Expected behavior with no VirtIO driver loaded. Setup offers **Load driver** → browse to `D:\vioscsi\2k25\amd64` → install the Red Hat VirtIO SCSI pass-through controller. The target disk appears immediately after.

### Windows 11 edition selection

The multi-edition ISO defaults to **Home**, which cannot join a domain under any circumstances. **Pro** must be explicitly selected from the image list — a dead end that would only surface much later at the join step.

### Microsoft account requirement during OOBE

Windows 11 setup pushes hard toward a Microsoft account, which is inappropriate for an isolated lab machine with no internet access.

**Resolution:** `Shift+F10` at the network screen opens a command prompt; `start ms-cxh:localonly` launches local account creation. (The older `oobe\bypassnro` method was removed in recent builds.)

### Domain join failing via PowerShell

Both `Add-Computer` and `Rename-Computer` returned `The user name or password is incorrect` despite confirmed LDAP connectivity to the DC.

**Resolution:** The same operations completed successfully through the System Properties GUI (`sysdm.cpl` → Computer Name → Change), which performed the rename and domain join in a single operation. The cmdlets proved more sensitive to local credential context than the GUI path.

**Lesson:** When a cmdlet fails but the underlying connectivity tests clean, the equivalent GUI tool is worth trying before deep troubleshooting.

---

## Skills demonstrated

- **Type 1 hypervisor administration** — Proxmox VE provisioning, storage configuration, snapshot management
- **Network segmentation** — isolated bridge design, host-based NAT gateway, deliberate air-gap controls
- **Windows Server administration** — AD DS role deployment, forest and domain creation, integrated DNS
- **Active Directory fundamentals** — forest/domain/OU hierarchy, FSMO roles, global catalog, SRV-record-based service location
- **PowerShell** — network configuration, role installation, forest promotion, system validation
- **Linux administration** — SSH key infrastructure, sshd hardening, Debian network configuration, secure file transfer
- **Systematic troubleshooting** — boot firmware diagnosis, driver dependency resolution, layered connectivity testing

---

## Planned next steps

1. Populate the directory with OUs, users, groups, and a realistic Group Policy structure
2. Deploy Sysmon with a tuned configuration on both hosts; enable command-line process auditing and PowerShell script block logging
3. Forward Windows event logs to a SIEM for centralized detection work
4. Run CALDERA adversary emulation against the domain and evaluate detection coverage against MITRE ATT&CK
5. Introduce deliberate misconfigurations (Kerberoastable service accounts, weak ACLs) and practice both exploitation and detection

---

## Environment reference

```
Proxmox VE 9.2.10 — node hooklab — 192.168.40.50 (vmbr0)
└── vmbr1 — 10.10.10.0/24 — isolated, host gateway 10.10.10.1
    ├── VM 100  HookDC       10.10.10.10   Windows Server 2025 — DC / DNS / KDC
    └── VM 101  HookClient   10.10.10.20   Windows 11 Pro — domain member

Forest / Domain : hooklab.local  (NetBIOS: HOOKLAB)
Functional level: Windows2025Domain
```
