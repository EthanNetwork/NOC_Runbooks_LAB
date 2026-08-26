# DHCP Pool Exhausted

## Symptom
- New clients fail to obtain an IP address; end users report "limited connectivity" or APIPA addresses (169.254.x.x).
- Syslog: `%DHCPD-4-NOBINDING` or similar "no address available" messages.
- Existing clients keep their leases but new devices joining the network can't connect.
- Helpdesk tickets clustering around a specific VLAN, subnet, or building/floor.

## Diagnostic Commands
```
show ip dhcp pool <pool-name>
show ip dhcp binding
show ip dhcp conflict
show ip dhcp server statistics
show run | section ip dhcp pool
show ip dhcp pool | include leased|Current|High
show vlan brief
show ip helper-address       ! on the client-facing interface/SVI
show logging | include DHCP
```

## Likely Causes
- **Pool undersized** for the number of active/expected devices on that subnet (common after VLAN growth or a BYOD/IoT rollout).
- **Long lease times** preventing timely reclamation of addresses from inactive/departed devices.
- **Stale or orphaned leases** not being released properly (e.g., abrupt disconnects, misbehaving clients).
- **DHCP conflicts** — statically assigned addresses overlapping with the DHCP pool range, shrinking effective availability.
- **Rogue DHCP server or misbehaving client** rapidly consuming leases (lease-starvation, sometimes malicious - a DHCP starvation attack).
- **IP helper-address misconfiguration** routing more clients than expected to a single pool, or several VLANs incorrectly sharing one pool.

## Resolution Steps
1. Confirm pool exhaustion: `show ip dhcp pool <pool-name>` — check "Current index," "High water mark," and "Total addresses" vs. leased count.
2. Identify what's consuming addresses: `show ip dhcp binding` to see active leases and correlate against expected device count.
3. Check for DHCP conflicts: `show ip dhcp conflict` — resolve any statically assigned IPs that overlap the DHCP range.
4. If legitimately short on space, expand the pool's address range (verify subnet has room) or reduce lease duration temporarily to recycle addresses faster.
5. Clear obviously stale bindings if devices are confirmed offline/decommissioned: `clear ip dhcp binding <address>` (or `*` for all, with caution).
6. If a rogue DHCP server or starvation attack is suspected, identify the source via port/MAC and isolate it; enable DHCP snooping on access switches if not already in place.
7. For sustained growth, consider splitting the VLAN/subnet or requesting additional address space to permanently resolve capacity constraints.
8. Re-verify new clients can obtain leases after remediation.

## When to Escalate
- Pool exhaustion is due to genuine subnet growth requiring re-addressing or a new subnet/VLAN — escalate to network engineering for a capacity/design change.
- Suspected DHCP starvation attack or rogue DHCP server — escalate to security team immediately in addition to network engineering.
- Exhaustion recurs shortly after expanding the pool, suggesting a leak (leases not being reclaimed) — escalate for deeper investigation into lease/client behavior.
- Business-critical location (e.g., a data center, hospital wing, trading floor) is affected — escalate immediately per severity policy regardless of root cause status.
