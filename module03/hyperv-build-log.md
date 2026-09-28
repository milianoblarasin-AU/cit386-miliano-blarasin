# Hyper-V Build Log and VHDX Disk Measurement

## Verification boundary

This computer runs Windows 11 Home, which does not provide the full Hyper-V role, Hyper-V Manager, the Virtual Machine Management service (`vmms`), or the Hyper-V PowerShell module. Windows reports that a hypervisor is present because virtualization-based platform features are active, but that is not the same as having the Hyper-V VM role. I therefore did not claim that a Hyper-V guest booted on this host. The VHDX disk experiment in Part Two was performed with Windows' built-in virtual-disk stack and uses measurements taken from the host filesystem.

## Part One: host workstation

| Item | Recorded value |
|---|---|
| Windows edition | Windows 11 Home, 64-bit |
| Windows version and build | 10.0.26200, build 26200 |
| Processor | 11th Gen Intel Core i7-11700KF at 3.60 GHz; 8 cores and 16 logical processors |
| Installed memory | 34,183,786,496 bytes (31.84 GiB, marketed as 32 GB) |
| Hyper-V role | Not available on this Windows edition; it was not enabled |

## Intended laboratory VM configuration

These are the settings to use when recreating the machine on the Windows Pro, Enterprise, Education, or Server laboratory host that provides Hyper-V:

| Setting | Choice | Reason |
|---|---|---|
| Name | `CIT386-Miliano-HV` | Identifies the course, owner, and hypervisor without relying on the guest hostname. |
| Generation | Generation 2 | Provides UEFI firmware, Secure Boot capability, and modern synthetic devices. |
| Guest operating system | Ubuntu Server 24.04 LTS, 64-bit | A supported 64-bit UEFI guest appropriate for Generation 2. |
| Startup memory | 4,096 MB | Enough for installation and normal command-line server work without reserving excessive host memory. |
| Dynamic Memory | On; 1,024 MB minimum and 8,192 MB maximum | Allows the host to recover unused memory while leaving room for the guest to grow under load. |
| Virtual processors | 2 | Adequate for a small course server while limiting contention on a shared host. |
| Network switch | `Default Switch` | Supplies simple NAT-based connectivity without requiring a dedicated external switch. |
| Virtual disk capacity | 60 GiB | Provides room for the operating system, updates, and course data. |
| Virtual disk type | Dynamically expanding VHDX | Conserves host storage initially and grows as guest blocks are written. |
| Intended disk path | `C:\Hyper-V\Virtual Hard Disks\CIT386-Miliano-HV.vhdx` | Keeps the disk in the laboratory host's dedicated Hyper-V storage location. |

### Why Generation 2

Generation 2 provides UEFI firmware, Secure Boot support, and a SCSI-based boot path rather than emulated legacy BIOS hardware. It requires a 64-bit guest that can boot through UEFI. It would be the wrong choice for an older 32-bit or BIOS-only operating system, or for media that depends on legacy devices available only to Generation 1.
