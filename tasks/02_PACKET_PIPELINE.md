# Task M2 — Validated packets, fragmentation, reassembly, and queues

## Operating mode

Plan first. Identify all resource limits and state machines before coding.

## Goal

Build a deterministic, bounded packet pipeline around the M1 codec without cryptography or live networking.

## Required deliverables

1. Add `mosaic-pipeline` crate.
2. In `mosaic-core`, implement a validated opaque `IpPacket` type supporting IPv4 and IPv6 minimum validation.
3. Implement packet-ID generation behind an injectable interface suitable for deterministic tests.
4. Implement fragmentation from one validated packet into canonical `DataFragment` cells.
5. Implement out-of-order reassembly.
6. Implement explicit queue types with byte and item limits.
7. Implement drop reasons and metrics counters.
8. Add a deterministic clock abstraction used for reassembly expiry.
9. Implement `MemoryPacketDevice` in `mosaic-packet-device`.
10. Extend `mosaic-sim` with deterministic loss, duplicate, reorder, and delay injection.
11. Create `docs/adr/0003-fragmentation-reassembly.md` and update the specification.

## Mandatory reassembly behavior

- exact duplicates with identical bytes do not deliver twice;
- conflicting duplicate or overlap rejects and discards the affected packet entry;
- inconsistent total length rejects the entry;
- offsets use checked arithmetic;
- incomplete entries expire;
- per-packet, entry-count, and global-byte limits are checked before allocation;
- completed packet is emitted exactly once;
- later fragments for a completed/rejected ID follow a documented bounded policy;
- attacker-controlled work is not quadratic in the maximum packet length.

## Queue behavior

The ADR must define:

- maximum packets and bytes;
- what happens on overflow;
- whether control data has a reserved budget;
- fairness policy;
- whether newest or oldest data is dropped;
- metrics for each drop reason.

Do not implement an unbounded `VecDeque` exposed to input.

## Tests

Cover the full matrix from `docs/TEST_PLAN.md`, including:

- every fragment order for small examples;
- duplicates and conflicting overlaps;
- gaps, expiry, and limit eviction;
- maximum packet;
- exact boundary and one-byte-over boundary;
- random fragmentation/reordering property tests;
- equality of completed output and original packet;
- no second delivery;
- memory-use invariant derived from configured limits;
- simulated backpressure.

## Constraints

- No cryptography.
- No session handshake.
- No live sockets.
- No TUN.
- No route changes.
- No HTTP/3.
- No traffic-shaping scheduler.
- Do not parse TCP, UDP, DNS, or application payloads.

## Acceptance criteria

A deterministic simulation must pass arbitrary valid IPv4/IPv6 test packets through:

```text
MemoryPacketDevice -> fragmentation -> codec -> simulated link
-> codec -> reassembly -> MemoryPacketDevice
```

with configured reorder/duplicate/loss scenarios and bounded state. All required commands pass and only M2 is implemented.
