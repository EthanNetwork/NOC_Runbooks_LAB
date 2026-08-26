# Interface Errors / Duplex Mismatch

## Symptom
- Slow throughput or intermittent connectivity on a specific link.
- Rising counters for CRC errors, late collisions, runts, or FCS errors on an interface.
- Interface shows "up/up" but performance is degraded.
- Applications report timeouts or retransmissions that correlate with a specific port/link.

## Diagnostic Commands
```
show interfaces <intf>
show interfaces <intf> | include duplex|errors|CRC|collisions|input errors|output errors
show interfaces status
show interfaces counters errors
show controllers <intf>          ! platform-dependent, physical layer detail
show run interface <intf>
show interfaces <intf> transceiver detail   ! for SFP/optics
clear counters <intf>            ! then re-check after a few minutes to see fresh error rate
```

## Likely Causes
- **Duplex/speed mismatch** — one end auto-negotiating, the other hard-set (or mismatched hard-set values), producing late collisions and CRC errors.
- **Faulty or dirty cable/fiber**, or a bad connector.
- **Failing or out-of-spec SFP/transceiver**, or mismatched optic types (e.g., single-mode vs multi-mode).
- **Cable length exceeding spec** for the given speed/medium.
- **EMI/interference** on copper runs near noise sources.
- **NIC or switch port hardware fault**.
- **Auto-negotiation disabled on only one side** of the link.

## Resolution Steps
1. Check `show interfaces <intf>` for duplex/speed on both ends of the link — they must match exactly (both auto, or both hard-set to the same value).
2. If mismatched, set both ends to the same mode — auto/auto is generally preferred for modern equipment unless a specific reason requires manual configuration.
3. Clear counters and monitor for a few minutes: `clear counters <intf>` then re-check error counts to confirm whether the issue is ongoing or historical.
4. Inspect and reseat the physical cable/SFP; try a known-good cable or optic to isolate hardware vs. configuration.
5. Verify cable type/length is within spec for the interface speed (e.g., Cat6 vs Cat5e limits, fiber mode compatibility).
6. Check the transceiver's diagnostic info (`transceiver detail`) for TX/RX power levels out of spec.
7. If errors persist on a specific physical port after ruling out cable/duplex, try moving the connection to a different port to isolate a hardware fault.
8. Document baseline error rate after remediation for future comparison.

## When to Escalate
- Errors persist after confirming matched duplex/speed, clean cabling, and known-good optics — escalate to hardware/facilities team to inspect the physical plant or replace the line card/port.
- Suspected hardware fault on core/aggregation switching or router equipment — escalate to network engineering for a maintenance window and possible RMA.
- Errors are affecting a production-critical link with active user impact — escalate per severity policy while continuing diagnosis in parallel.
- Recurrent faults on the same port after transceiver/cable replacement — escalate for chassis/module-level investigation.
