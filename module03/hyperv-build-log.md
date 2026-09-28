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

## Part Two: measured VHDX behavior

### Method

I created two empty, Hyper-V-compatible VHDX files with Windows DiskPart. Each disk had a configured capacity of 2,048 MiB and was quick-formatted as NTFS. The measured files were:

- Expanding: `C:\Users\leala\Documents\Codex\2026-09-20\do-x20\artifacts\hyperv-test\cit386-dynamic.vhdx`
- Fixed: `C:\Users\leala\Documents\Codex\2026-09-20\do-x20\artifacts\hyperv-test\cit386-fixed.vhdx`

For each disk, I recorded the VHDX file's `Length` from the Windows host, attached the disk, wrote a 536,870,912-byte (512 MiB) file containing generated nonzero data, flushed the write, detached the disk, and recorded the host file size again. I then reattached the disk, deleted the payload, detached it, and took the last host-side measurement. Measuring after detachment ensured that buffered writes had reached the VHDX file.

### Measurements from the host filesystem

| Disk type | Configured capacity | Initial host-file size | Data written | Host size after write | Host size after deleting data |
|---|---:|---:|---:|---:|---:|
| Dynamically expanding VHDX | 2,048 MiB | 104,857,600 bytes (100 MiB) | 536,870,912 bytes (512 MiB) | 641,728,512 bytes (612 MiB) | 641,728,512 bytes (612 MiB) |
| Fixed VHDX | 2,048 MiB | 2,151,677,952 bytes (2,052 MiB) | 536,870,912 bytes (512 MiB) | 2,151,677,952 bytes (2,052 MiB) | 2,151,677,952 bytes (2,052 MiB) |

Before deleting the payload, I expected the expanding disk to become smaller because 512 MiB had been freed inside its filesystem. It did **not** shrink: its host file stayed at 612 MiB. Deleting a guest file marks filesystem blocks as available for reuse inside the virtual disk, but it does not automatically compact the VHDX container on the host. Reclaiming that host space requires a separate supported optimization or compaction procedure.

The fixed disk behaved differently at creation time rather than during the write. Its host file immediately occupied approximately the entire configured capacity, plus VHDX metadata, and remained the same size after both the write and deletion.

## What the figures mean for a busy host

Twelve freshly formatted copies of this expanding test disk would occupy about 1,200 MiB, while twelve copies after the same write would occupy about 7,344 MiB. Twelve fixed disks would reserve about 24,624 MiB immediately. Expanding disks improve initial storage density, but a busy host must still be sized for their possible growth, and deleting guest data does not automatically return that capacity to the host. I would choose fixed disks when predictable allocation and consistent write behavior matter more than density—for example, a latency-sensitive database host where the administrator wants capacity reserved before production load arrives.

## Commands used for the measurement

The disks were created as an expanding and a fixed VHDX with DiskPart. The 512 MiB payload was written from the host through the attached test volume with a buffered .NET file stream and flushed before each measurement. The host sizes were read with:

```powershell
Get-Item -LiteralPath ".\cit386-dynamic.vhdx", ".\cit386-fixed.vhdx" |
    Select-Object Name, Length, @{Name='MiB'; Expression={[math]::Round($_.Length / 1MB, 2)}}
```

All reported sizes are text copied from the host filesystem measurements, not values displayed inside a guest operating system.
