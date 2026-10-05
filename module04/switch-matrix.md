# Hyper-V Reachability Matrix and VirtualBox Mapping

## Verification boundary

The available computer runs Windows 11 Home. It has the Windows hypervisor platform used by VirtualBox's NEM backend, but it does not provide Hyper-V Manager, the Hyper-V virtual-switch cmdlets, or the full Hyper-V role. I did not invent laboratory observations or timing figures that were not measured. The tables below record the expected network behavior and identify the two items that still require the assigned laboratory host: the tested reachability cells and the paired VirtualBox boot timings.

## Reachability targets

The five targets used in the matrix are:

1. **Management OS** — the Windows host that owns the virtual switch.
2. **Same-switch guest** — another VM attached to the same named virtual switch.
3. **Physical LAN peer** — another computer on the host's physical network.
4. **Internet** — an address beyond the local network, assuming normal upstream routing.
5. **Inbound from the physical LAN** — a connection initiated by a different physical computer toward the guest.

## Switch reachability matrix

`Expected` describes the result implied by the switch design. Each cell must be replaced or confirmed with the result observed on the laboratory machine before submission.

| Hyper-V switch configuration | Management OS | Same-switch guest | Physical LAN peer | Internet | Inbound from physical LAN |
|---|---|---|---|---|---|
| External; management OS shares the adapter | Expected: reachable | Expected: reachable | Expected: reachable | Expected: reachable | Expected: reachable, subject to guest firewall |
| External; management OS does **not** share the adapter | Expected: not reachable through that adapter | Expected: reachable | Expected: reachable | Expected: reachable | Expected: reachable, subject to guest firewall |
| Internal | Expected: reachable | Expected: reachable | Expected: not reachable | Expected: not reachable without separately configured routing/NAT | Expected: not reachable |
| Private | Expected: not reachable | Expected: reachable | Expected: not reachable | Expected: not reachable | Expected: not reachable |

### External-switch row change

Turning off **Allow management operating system to share this network adapter** removes the host-side virtual network adapter from that external switch. The guest remains bridged to the physical network, so its same-switch, LAN, Internet, and inbound results should not change. The management OS result is the one that changes: it no longer communicates through that physical adapter. A host with another active network adapter may still be reachable by a different path, so the test must target the address associated with the adapter used by the external switch.

### Result that is easiest to predict incorrectly

The external-switch sharing option affects the management operating system, not the guest's bridge to the physical network. I would have expected disabling sharing to disconnect the guest because the physical adapter appears to belong to Windows. Hyper-V instead binds the physical adapter to the virtual switch and gives the management OS an optional virtual adapter on that switch. Removing that optional adapter isolates the host on that path while leaving the guest's external connectivity intact.

## Hyper-V to VirtualBox mode mapping

| Hyper-V switch type | Closest VirtualBox network mode | Why it is the closest match |
|---|---|---|
| External | Bridged Adapter | Both connect the guest directly to the physical LAN, where it can obtain a LAN address and accept inbound connections subject to firewall policy. |
| Internal | Host-only Adapter | Both create a network shared by the host and attached guests without connecting it directly to the physical LAN. |
| Private | Internal Network | Both limit communication to guests attached to the same named virtual network and omit a host-side interface. |

The external pairing does not line up perfectly when **Allow management operating system to share this network adapter** is disabled. VirtualBox Bridged Adapter mode bridges a guest to a selected host interface, but it does not offer the same per-switch control that removes the management operating system's virtual adapter while preserving the guest bridge. Hyper-V's managed **Default Switch** also has no exact one-to-one entry in the three-row table; VirtualBox NAT is similar for outbound access, but the Default Switch is created and reconfigured automatically by Windows.

## VirtualBox timing comparison

The paired boot test requires the same guest to be timed once with the Windows hypervisor platform disabled and once with it enabled. The available system was not rebooted with security and platform features disabled, so the off-platform figure is not available and no number is claimed.

| VirtualBox guest state | Time from Start until login prompt | Evidence |
|---|---:|---|
| Windows hypervisor platform off; direct VT-x/AMD-V backend | **Laboratory measurement required** | Record the stopwatch result and the relevant VirtualBox log line naming the direct hardware-virtualization backend. |
| Windows hypervisor platform on; Hyper-V/NEM backend | **Laboratory measurement required** | Record the stopwatch result and the VirtualBox log line naming NEM or the Windows Hypervisor Platform. |

With the platform off, VirtualBox can own the processor's hardware-virtualization interface directly. With it on, the Microsoft hypervisor owns that interface and VirtualBox submits work through the Windows Hypervisor Platform/NEM layer. The guest configuration does not change, but an additional scheduling and translation layer moves underneath VirtualBox; that can increase boot time and is why the two states must be measured rather than assumed.

## Laboratory completion checklist

- [ ] Replace or confirm every expected matrix result using the assigned Hyper-V laboratory host.
- [ ] Record the external-switch row with management-OS sharing enabled.
- [ ] Record the external-switch row again with management-OS sharing disabled.
- [ ] Record both VirtualBox timing figures using the same guest and the same start/stop points.
- [ ] Preserve the relevant log line or timestamp used to support each timing.

