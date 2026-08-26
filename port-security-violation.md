# Port Security Violation

## Symptom
- Switch port goes `err-disabled` unexpectedly, cutting off a device or user.
- Syslog: `%PORT_SECURITY-2-PSECURE_VIOLATION: Security violation occurred, caused by MAC address <mac> on port <intf>`.
- User reports sudden loss of network connectivity after plugging in a new device, swapping a NIC, or connecting a switch/hub to a single port.
- Repeated violations logged on the same port over a short period.

## Diagnostic Commands
```
show port-security
show port-security interface <intf>
show port-security address
show interfaces <intf> status
show interfaces <intf> | include err-disabled
show mac address-table interface <intf>
show errdisable recovery
show errdisable detect
show logging | include PORT_SECURITY
```

## Likely Causes
- **New/unauthorized device plugged in**, exceeding the configured maximum MAC address count on the port.
- **NIC replacement or virtualization** (new MAC address) on a device previously learned/sticky on that port.
- **Unauthorized hub/switch connected**, introducing multiple MAC addresses behind a single access port.
- **MAC spoofing or a rogue device** attempting to bypass network access controls.
- **Sticky MAC misconfiguration** — port learned the wrong MAC initially (e.g., during a temporary device swap) and now legitimately rejects the correct one.
- **Violation mode set to `shutdown`** causing the port to err-disable on the very first violation, rather than `restrict`/`protect` which drop offending traffic without disabling the port.

## Resolution Steps
1. Identify the port and violation details: `show port-security interface <intf>` — note the configured max, current count, violation mode, and the offending MAC.
2. Confirm whether the new MAC is expected (legitimate device/NIC swap) or unexpected (possible unauthorized device).
3. If legitimate: clear the port security violation and reset the port:
   - `shutdown` then `no shutdown` on the interface, or
   - `clear port-security sticky interface <intf>` / remove the stale MAC, then re-add or let it relearn (If the MAC was stickied, make sure you put it back in sticky w/ config-if# switchport port-security mac-address sticky [MAC-ADDRESS] rather than leaving it dynamic, you dk why it was stickied but someone had a good reason) 
4. If the port was configured for sticky MAC and the device changed, update or clear the sticky entry so the new MAC is accepted going forward.
5. If unauthorized/unexpected device: do not simply reset the port — investigate the physical location and device before restoring access.
6. Review whether the max MAC count is appropriate for that port (e.g., a port feeding an IP phone + PC needs at least 2).
7. Confirm `errdisable recovery` settings — decide whether auto-recovery is appropriate for this port type or if manual intervention should remain required for security-sensitive ports.
8. Document the violation and resolution, especially if it involved an unauthorized device.

## When to Escalate
- Repeated or widespread port security violations across multiple ports/switches, suggesting a coordinated intrusion attempt — escalate to security team immediately.
- Confirmed or suspected unauthorized device/rogue hub found connected — escalate to security and follow incident response procedures before restoring the port.
- Violation is on a sensitive or regulated segment (e.g., PCI, finance, data center) — escalate per compliance/security policy regardless of apparent cause.
- Uncertain whether a MAC change is legitimate (e.g., cannot reach the end user to confirm) — escalate to the site/desktop support team to verify before re-enabling the port.
