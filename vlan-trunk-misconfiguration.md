# VLAN Trunk Misconfiguration

## Symptom
- Devices on a VLAN cannot reach each other across switches, or intermittently lose connectivity.
- Certain VLANs work while others don't across the same trunk.
- Spanning-tree topology changes or a port unexpectedly placed in `err-disabled`/blocking state.
- Broadcast storms or duplicate traffic reported on a segment.
- New VLAN added but not reachable on remote switches.

## Diagnostic Commands
```
show interfaces trunk
show interfaces <intf> switchport
show vlan brief
show spanning-tree vlan <id>
show spanning-tree interface <intf> detail
show interfaces <intf> status
show run interface <intf>
show mac address-table vlan <id>
show cdp neighbors detail        ! confirm the other end of the link
show etherchannel summary        ! if the trunk is part of a port-channel
```

## Likely Causes
- **Native VLAN mismatch** between trunk ends — causes CDP native VLAN mismatch warnings and VLAN 1 traffic leakage or tagging errors.
- **Allowed VLAN list mismatch** — a VLAN is pruned/not permitted on one side of the trunk (`switchport trunk allowed vlan`).
- **Trunk encapsulation mismatch** (802.1Q vs ISL on older gear, or mismatched negotiation mode).
- **One side configured as access port**, the other as trunk — DTP negotiation failure or silent drop of tagged frames.
- **VTP domain/mode mismatch** causing VLAN database inconsistency across switches (if VTP is in use).
- **Spanning-tree misconfiguration** blocking the VLAN on that link (e.g., PVST inconsistency).
- **VLAN not created** on one or more switches in the path.

## Resolution Steps
1. Confirm both ends of the link are configured as trunks: `show interfaces trunk` and `show interfaces <intf> switchport` on both switches.
2. Compare native VLAN on both ends — they must match, or expect tagging/security issues and syslog warnings.
3. Verify the allowed VLAN list includes the VLAN(s) in question on every switch along the path: `switchport trunk allowed vlan <list>`.
4. Confirm the VLAN actually exists in the VLAN database on each switch (`show vlan brief`) — a missing VLAN definition silently drops traffic even if allowed in the trunk list.
5. Check trunk encapsulation settings match (dot1q on both sides) if the platform supports both ISL and dot1q.
6. If using VTP, confirm domain name, mode (server/client/transparent), and revision number consistency; mismatches can wipe or alter VLAN databases.
7. Check `show spanning-tree vlan <id>` for the port's STP state on that VLAN — if blocking unexpectedly, investigate root bridge election and STP topology.
8. After correcting mismatches, verify with `show mac address-table vlan <id>` that MAC addresses are being learned correctly across the trunk.

## When to Escalate
- VLAN issue spans multiple switches/sites and requires a coordinated change window to fix trunk configs — escalate to network engineering for planned remediation.
- Suspected VTP-related VLAN database corruption affecting multiple switches — escalate immediately, this can cause wide-reaching outages.
- Spanning-tree instability (topology change loops, root bridge flapping) tied to the trunk — escalate to senior network engineering before making further changes.
- Fix requires a maintenance window on a production core/distribution switch — escalate to schedule appropriately rather than making live changes.
