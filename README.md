# NOC Runbooks

Quick-reference troubleshooting guides for common network operations issues. Each runbook follows the same structure: **Symptom → Diagnostic Commands → Likely Causes → Resolution Steps → When to Escalate**.

## Contents

| Runbook | Description |
|---|---|
| [OSPF Neighbor Down](./ospf-neighbor-down.md) | Adjacency stuck or missing, routes disappearing |
| [BGP Session Flapping](./bgp-session-flapping.md) | Repeated Established/Idle transitions, route churn |
| [Interface Errors / Duplex Mismatch](./interface-errors-duplex-mismatch.md) | CRC errors, collisions, degraded throughput |
| [VLAN Trunk Misconfiguration](./vlan-trunk-misconfiguration.md) | Cross-switch VLAN reachability issues |
| [DHCP Pool Exhausted](./dhcp-pool-exhausted.md) | Clients unable to obtain IP addresses |
| [Port Security Violation](./port-security-violation.md) | Ports err-disabling due to MAC violations |

## Usage

These are first-response guides intended for NOC tier-1/tier-2 engineers. Commands shown use generic Cisco IOS-style syntax — adjust for your specific platform (NX-OS, Junos, Arista EOS, etc.) as needed. Currently, my training covers CISCO syntax, as I gain experience with other vendor CLI those will be added/updated. 

Each runbook's **When to Escalate** section defines the point at which an issue should be handed off to network engineering, security, or another specialized team rather than continuing tier-1/2 troubleshooting.

