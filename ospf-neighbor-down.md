# OSPF Neighbor Down

## Symptom
- OSPF neighbor state stuck in `INIT`, `2-WAY`, `EXSTART/EXCHANGE`, or missing entirely (no neighbor listed).
- Routes learned via OSPF disappear from the routing table.
- SNMP/syslog traps: `%OSPF-5-ADJCHG: Process X, Nbr <ip> ... from FULL to DOWN`.
- Downstream: traffic blackholing or suboptimal routing via a backup path.

## Diagnostic Commands
```
show ip ospf neighbor
show ip ospf interface <intf>
show ip ospf interface brief
show ip protocols
show run | section router ospf
show ip route ospf
ping <neighbor-ip>
show interfaces <intf> | include line protocol|MTU|duplex
debug ip ospf adj        ! use with caution in production
show ip ospf database
```

## Likely Causes
- **Layer 1/2 issue**: interface down, bad cable/SFP, or the link is up but one side has errors (see interface-errors runbook).
- **MTU mismatch** between neighbors — adjacency hangs in EXSTART/EXCHANGE.
- **Area ID or network type mismatch** (e.g., one side `point-to-point`, other `broadcast`).
- **Hello/Dead timer mismatch** — neighbors must match on both.
- **Authentication mismatch** (type or key) if OSPF auth is configured.
- **ACL or firewall** blocking OSPF multicast (224.0.0.5/6) or protocol 89.
- **Mismatched subnet/mask** on the shared segment.
- **Duplicate router-ID** in the OSPF domain.

## Resolution Steps
1. Confirm physical/L2 link is up and clean: `show interfaces <intf>` — check for errors, CRC, resets.
2. Verify IP reachability with a basic `ping` to the neighbor's interface address.
3. Compare OSPF parameters on both sides: area, network type, hello/dead intervals, authentication type/key, MTU.
4. If stuck at EXSTART/EXCHANGE, fix the MTU mismatch or set `ip ospf mtu-ignore` as a temporary workaround (not a permanent fix).
5. Check for duplicate router-IDs (`show ip ospf` on each device) and correct if found.
6. Verify no ACL/firewall is filtering OSPF multicast/protocol 89 between the two devices.
7. Clear the OSPF process on one side if config looks correct but adjacency won't form: `clear ip ospf process`.
8. Re-verify neighbor state returns to `FULL` and routes repopulate.

## When to Escalate
- Adjacency won't come up after confirming L1/L2, IP, and OSPF parameter parity — escalate to the routing/network engineering team for deeper packet capture analysis.
- Suspected hardware fault (bad SFP, line card) affecting multiple neighbors on the same device.
- Issue affects a core/backbone area (Area 0) with active production impact — escalate immediately per severity policy, don't wait to fully diagnose.
- Duplicate router-ID or design-level misconfiguration spanning multiple sites — escalate to network architecture for a config audit.
