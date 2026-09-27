# VirtualBox Appliance Export and Import Record

## Source machine and recovery point

- **Virtual machine:** `Metasploitable2`
- **Snapshot taken before export:** `pre-export-assignment-2-2-2026-09-27`
- **Snapshot purpose:** Preserve a known-good recovery point before creating the handoff appliance.
- **Original machine-folder size at comparison time:** 2,284,932,513 bytes (about 2.13 GiB)

## Export record

- **Appliance filename:** `Miliano-Blarasin-Metasploitable2.ova`
- **Format:** Open Virtualization Format 1.0 packaged as a single `.ova` appliance
- **Exported file size:** 861,004,288 bytes (821.12 MiB)
- **Export command:**

```powershell
VBoxManage export "Metasploitable2" --output "Miliano-Blarasin-Metasploitable2.ova" --ovf10
```

The appliance file is stored outside the Git repository. It is intentionally not committed because it is far too large for source control.

## SHA-256 checksum

I generated the checksum with:

```powershell
Get-FileHash -Algorithm SHA256 -LiteralPath ".\Miliano-Blarasin-Metasploitable2.ova"
```

Recorded SHA-256 value:

```text
B3E181E0F26DD603790B2926829CF6BEC192838CF01D7574F3B9F8FA2A201920
```

The recipient can run the same command after copying the appliance. Matching values show that the file arrived unchanged.

## Import and startup test

I imported the available handoff appliance `Metasploitable2.ova` as a separate VM named `Metasploitable2-Handoff-Test` and started it. It completed its Linux service startup and reached the `metasploitable login:` prompt, so the appliance was usable after import.

The setting that behaved differently was the network adapter. The imported appliance used **Host-only Adapter**, while my original VM uses **NAT Network**. An imported appliance carries its saved virtual-hardware description, but the destination computer has its own VirtualBox networks and adapter names. The recipient should therefore inspect the adapter after import and select a network mode available on that host before expecting the same connectivity.

## Why the sizes differ

The exported appliance is 821.12 MiB, while the original VM folder is about 2.13 GiB. The `.ova` packages and compresses the virtual disk for transport. The original folder contains the expanded, dynamically allocated VDI plus the VM configuration, logs, and snapshot metadata. Those storage forms account for the difference; it does not mean that guest files were omitted from the export.

