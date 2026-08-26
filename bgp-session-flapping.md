# BGP Session Flapping

## Symptom
- BGP neighbor repeatedly transitions between `Established` and `Idle`/`Active`.
- Syslog: `%BGP-5-ADJCHANGE: neighbor <ip> Down` followed shortly by `Up`, repeating.
- Route churn — routes withdrawn and re-advertised, causing downstream instability or packet loss.
- Possible CPU spikes on the router during reconvergence.

## Diagnostic Commands
```
show ip bgp summary
show ip bgp neighbors <ip>
show ip bgp neighbors <ip> | include Last read|Last write|reset reason
show logging | include BGP
show interfaces <intf> | include errors|drops|CRC
show ip route <neighbor-ip>
show tcp brief | include <neighbor-ip>
show processes cpu sorted | exclude 0.00
traceroute <neighbor-ip>
show ip bgp neighbors <ip> received-routes
```

## Likely Causes
- **Underlying link instability** — flapping physical interface, errors, or intermittent connectivity to the peer.
- **BGP hold-timer expiration** due to high CPU, congestion, or missed keepalives.
- **MTU mismatch** or path MTU issues causing update packets to be dropped.
- **Max-prefix limit exceeded**, causing the session to be torn down intentionally.
- **Route flapping upstream** propagating instability (not the local link itself).
- **Duplicate router-ID or ASN misconfiguration**.
- **Asymmetric routing / unstable IGP path** to the peer address (for multi-hop eBGP/iBGP).
- **DDoS or traffic flood** on the peering link starving BGP control-plane traffic.

Note: in a previous lab I ran into BGP flapping, and logs filled with bgp errors (which required using config# logging console 1 to allow me to enter commands and troubleshoot without the cli filling with errors) due to a failure to hardcode a router-id, which upon reload caused the router to select its own RID, which did not match the configurations previously administered

## Resolution Steps
1. Check `show ip bgp summary` for uptime and reset counters — confirm the flap pattern and frequency.
2. Inspect the physical/logical path to the peer: interface errors, recent link flaps, CRC errors.
3. Review `show ip bgp neighbors <ip>` for the "reset reason" — this usually points directly at the cause (e.g., "Peer closed the session," "Hold timer expired," "Max prefix limit exceeded").
4. If hold-timer expiration: check router CPU/memory utilization and interface congestion during flap windows.
5. If max-prefix exceeded: verify whether the prefix increase is expected (legitimate growth) or a leak/hijack from the peer, then adjust the limit or filter as appropriate.
6. If MTU-related, verify consistent MTU end-to-end and check for fragmentation issues.
7. For multi-hop sessions, confirm the IGP/static route to the peer's loopback is stable and not flapping.
8. Consider tuning BGP timers (with caution and peer coordination) or enabling BFD for faster, more deterministic failure detection instead of relying on long hold timers.
9. Document flap timestamps and correlate with any recent maintenance, capacity changes, or upstream provider incidents.

## When to Escalate
- Flapping continues after ruling out local interface, CPU, and timer issues — escalate to routing engineering for capture/analysis of BGP update traffic.
- Session involves an external peer/ISP and the issue is suspected to be on their side — escalate to the peering/circuit provider with timestamps and reset reasons in hand.
- Max-prefix drops are being caused by a suspected route leak or hijack — escalate immediately to security/network engineering.
- Flapping is causing customer-facing outages or SLA breaches — escalate per incident severity policy regardless of root cause status.
