# VirtualBox Build Log: Metasploitable2

## Host inventory

| Item | Recorded value | Notes |
|---|---|---|
| Host manufacturer | ASUS | Reported by Windows as the system manufacturer. |
| Host model | System Product Name | This is the model string supplied by the system firmware. |
| Processor | 11th Gen Intel Core i7-11700KF at 3.60 GHz | The host has substantially more CPU capacity than this small training guest requires. |
| Total memory | 34,183,786,496 bytes (31.84 GiB; marketed as 32 GB) | This leaves ample memory for the host while the guest is running. |
| Firmware virtualization | Already enabled; no firmware change was required for this build | Windows reports that a hypervisor is active and virtualization-based security is running. VirtualBox therefore uses the Windows Hyper-V/NEM backend rather than taking direct ownership of VT-x. |

## Guest settings and reasons

| Setting | Chosen value | Reason |
|---|---|---|
| Name | `Metasploitable2` | The name matches the intentionally vulnerable training system and makes its purpose immediately recognizable in VirtualBox. |
| Guest operating-system type | `Linux 2.6 / 3.x / 4.x / 5.x (64-bit)` | Metasploitable2 is an older Ubuntu-based Linux appliance, so this generic Linux profile is compatible with its kernel and virtual hardware. |
| Memory | 1,024 MB | The appliance is a small command-line security lab server. One GiB is enough for its services while using only about 3.1% of the host's memory. |
| Processor count | 1 vCPU | The deliberately vulnerable lab services do not need parallel compute, and one vCPU minimizes unnecessary host load. |
| Virtual disk size | 8,192 MB (8 GiB) | The installed appliance and its lab data fit comfortably while the capacity remains small enough for quick copies and backups. |
| Disk format | VDI | VDI is VirtualBox's native disk format and provides the simplest management in this single-hypervisor lab. |
| Disk allocation | Dynamically allocated | Dynamic allocation avoids consuming the entire 8 GiB on the host before the guest actually writes that much data. |

The VM also uses a single SATA-attached disk and a NAT Network adapter. NAT Network gives the training guest outbound and lab-network connectivity without bridging the intentionally vulnerable machine directly onto the physical LAN.

## Virtual disk on the host

- **Full path:** `C:\Users\leala\VirtualBox VMs\Ethical Hacking\Metasploitable2\Metasploitable2-disk001.vdi`
- **Virtual capacity shown by VirtualBox:** 8,192 MB (8 GiB)
- **Current host-file size:** 2,281,701,376 bytes (about 2.13 GiB)
- **Comparison:** The file currently occupies about 26.6% of its maximum virtual capacity. This difference is expected because the VDI is dynamically allocated.

## What did not work the first time

Direct VT-x acceleration was not available to VirtualBox. The VM log reported `VT-x is not available` and said that VirtualBox was falling back to NEM. I checked Windows with `systeminfo` and confirmed that a Microsoft hypervisor and virtualization-based security were already active. Instead of disabling those security features, I kept the working configuration and allowed VirtualBox to use the Windows Hyper-V/NEM backend. The VM booted through that backend; the tradeoff is that the VirtualBox log warns it cannot run at its full potential in this mode.

## Verification method

I recorded the host values from Windows system information and the guest values from VirtualBox 7.2.0 using `VBoxManage showvminfo` and `VBoxManage showmediuminfo`. I also checked the VM log to verify the virtualization backend and the fallback described above.
