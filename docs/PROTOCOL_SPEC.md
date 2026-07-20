# MOSAIC Protocol Specification

Status: Pre-implementation draft 0.1

This document defines the target semantics. Wire-format details become normative only after the corresponding ADR is accepted and implementation tests are added.

The key words MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY indicate requirement strength.

## 1. Overview

MOSAIC transfers complete IPv4 and IPv6 packets through an authenticated encrypted session. It is carrier-independent: an in-memory test link, a byte stream, or HTTP/3 may carry the same protected records.

MOSAIC has four nested units:

```text
inner IP packet
  -> one or more DATA_FRAGMENT cells
  -> one or more plaintext batches
  -> protected records
  -> scheduler carrier events
```

Carrier-event boundaries MUST NOT define cell, batch, fragment, or inner-packet boundaries.

## 2. Byte order and canonical encoding

All fixed-width integers use network byte order (big-endian).

The initial codec ADR MUST choose one canonical length/identifier representation after comparing fixed-width integers, bounded unsigned varints, and TLV. Whichever representation is selected:

- there MUST be one encoding for each value;
- overlong and non-canonical representations MUST be rejected;
- all size conversions MUST use checked arithmetic;
- the decoder MUST validate length limits before allocation;
- trailing bytes MUST be either explicitly permitted or rejected.

## 3. Versions and features

The protocol has:

- major version: incompatible wire semantics;
- minor version: backward-compatible additions;
- mandatory feature bits;
- optional feature bits.

A peer MUST reject an unknown mandatory feature. A peer MAY ignore an unknown optional feature only when its length and skipping rule are unambiguous.

Version negotiation itself MUST be authenticated by the session handshake or protected record layer. Downgrade-sensitive values MUST be included in the handshake transcript.

## 4. Limits

Both peers negotiate limits, but each peer always enforces the smaller of its local limit and the peer-advertised limit.

Required limit categories:

- maximum inner packet length;
- maximum cell body length;
- maximum plaintext batch length;
- maximum protected record length;
- maximum fragments per packet;
- maximum incomplete reassemblies;
- maximum total reassembly bytes;
- maximum queued bytes per direction;
- maximum control-message length;
- maximum handshake-message length.

A peer MUST NOT accept a negotiated value above its hard implementation ceiling.

## 5. Plaintext batch

After session establishment, the AEAD plaintext represents one batch:

```text
Batch {
    major_version
    minor_version
    batch_flags
    cell_count
    cells[cell_count]
}
```

Requirements:

- `cell_count` is bounded;
- the sum of encoded cells equals the batch length exactly;
- unknown mandatory cell types reject the batch;
- padding cells are included in the authenticated plaintext;
- a malformed cell rejects the complete protected record;
- no partially decoded application data is delivered from a rejected record.

## 6. Cell types

Initial cell types:

```text
DATA_FRAGMENT
CONTROL
PADDING
```

Reserved values are not accepted unless negotiated as optional extensions.

### 6.1 DATA_FRAGMENT

Conceptual fields:

```text
DataFragment {
    packet_id: u64
    total_packet_len: bounded integer
    fragment_offset: bounded integer
    fragment_len: bounded integer
    fragment_bytes[fragment_len]
}
```

Validation:

- total length is non-zero and within the inner-packet limit;
- fragment length is non-zero unless a future negotiated extension permits otherwise;
- offset is smaller than total length;
- offset plus fragment length is checked for overflow and does not exceed total length;
- encoded body length matches the declared fragment length exactly;
- a packet identifier is scoped to one session direction and key epoch policy defined by ADR.

A sender SHOULD choose fragment sizes using the current record budget rather than the inner path MTU alone.

### 6.2 CONTROL

Conceptual form:

```text
Control {
    control_code
    control_flags
    value_len
    value[value_len]
}
```

Initial controls:

- `PARAMETERS` — negotiated non-secret limits and capabilities;
- `PING` — liveness token;
- `PONG` — corresponding token;
- `KEY_UPDATE` — key-epoch transition signal when required by the selected construction;
- `CLOSE` — graceful close category and optional non-sensitive reason;
- `ERROR` — authenticated post-handshake protocol error category.

Control messages MUST be idempotent or have explicit duplicate handling.

### 6.3 PADDING

Padding is authenticated and ignored after validation.

Conceptual form:

```text
Padding {
    padding_len
    bytes[padding_len]
}
```

The receiver MUST bound padding length. Padding bytes MUST NOT carry hidden protocol semantics.

## 7. Packet identifiers

Packet identifiers MUST avoid ambiguity among simultaneously incomplete packets. The sender may use a monotonically increasing counter with a cryptographically random initial value or another ADR-approved construction.

Identifier reuse is prohibited while an earlier packet with that identifier could remain in the receiver's reassembly window.

## 8. Fragmentation

The sender:

1. validates the complete inner packet;
2. assigns a packet identifier;
3. chooses fragments that fit current batch limits;
4. covers each byte exactly once;
5. may interleave fragments from different packets;
6. must not create zero-progress fragmentation loops.

The protocol does not require fragments to arrive in order.

## 9. Reassembly

A reassembly entry contains:

- packet identifier;
- expected total length;
- received byte ranges;
- bounded storage;
- creation and last-update time;
- delivery state.

Receiver behavior:

- exact duplicate ranges with identical bytes MAY be accepted without additional delivery;
- conflicting duplicate bytes MUST reject and discard the entry;
- overlap policy MUST be deterministic and tested;
- total-length changes MUST reject and discard the entry;
- expired entries are discarded;
- global and per-entry limits are checked before allocation;
- a completed packet is validated and emitted exactly once;
- later duplicates cannot cause second delivery.

## 10. Session establishment

The exact handshake pattern and library are selected in M3 by ADR. It MUST satisfy:

- server authentication to the client;
- client authorization using standard authenticated cryptographic mechanisms;
- forward secrecy when supported by the selected standard pattern;
- transcript binding of versions, mandatory features, roles, and negotiated limits;
- independent send and receive traffic secrets;
- resistance to replay of completed initiation messages within the stated model;
- bounded handshake parsing and state;
- no custom cipher or custom key exchange.

The implementation MUST separate unauthenticated handshake input from established record processing.

## 11. Protected records

A protected record contains enough public framing to locate and decrypt one ciphertext. Exact fields are selected by ADR.

Conceptual form:

```text
ProtectedRecord {
    public_header
    ciphertext
    authentication_tag
}
```

Requirements:

- header fields needed before decryption are authenticated as associated data;
- keys are direction-specific;
- each `(key, nonce)` pair is unique;
- packet numbers or equivalent nonce inputs use checked rollover handling;
- authentication failure delivers no plaintext;
- repeated authentication failures are rate-limited at the orchestration layer;
- the decoder does not reveal secret-dependent detail to the peer.

## 12. Replay protection

For carriers that may duplicate or reorder protected records, the receiver maintains a bounded anti-replay window per key epoch.

A record is delivered at most once. Too-old records, duplicate records, and records outside allowed epochs are rejected without modifying application state.

For a strictly ordered reliable stream, an implicit sequence may be used only if the ADR explains reconnection and truncation behavior.

## 13. Key epochs

The session supports an initial traffic-key epoch and bounded key updates. The ADR must define:

- update trigger;
- sender transition;
- receiver acceptance window;
- simultaneous update behavior;
- old-key deletion;
- packet-number reset or continuation;
- recovery from lost update signals.

Epoch ambiguity or nonce reuse MUST terminate the session.

## 14. Carrier contract

A carrier reports capabilities but never changes protocol semantics.

The protocol assumes only that:

- reliable carriers eventually report closure or delivery failure;
- a stream may split or combine writes arbitrarily;
- datagrams may be lost, duplicated, or reordered;
- maximum event size is finite;
- backpressure is observable.

The first end-to-end implementation uses reliable ordered delivery. Datagram support is optional until separately specified.

## 15. Scheduler contract

The scheduler maps protected-record bytes to carrier events and reconstructs the same protected-record byte stream at the receiver.

It may:

- split record bytes;
- coalesce adjacent record bytes;
- delay within a configured budget;
- insert carrier-level padding that is unambiguously removable;
- interleave queues while preserving each required ordered stream.

It MUST NOT:

- alter protected bytes;
- exceed hard memory limits;
- delay control traffic beyond its deadline;
- create an unbounded cover-traffic loop;
- inspect decrypted IP payloads.

## 16. HTTP/3 initiation and decoy behavior

The HTTP/3 adapter carries opaque handshake and protected bytes inside an operator-controlled application protocol.

Requirements:

- standard TLS certificate validation remains enabled;
- no use of third-party front domains or unrelated services;
- ordinary clients can use the decoy application without MOSAIC credentials;
- invalid tunnel initiation follows normal decoy application behavior;
- unauthenticated input does not receive a distinctive proxy or tunnel error;
- authentication and tunnel dispatch occur only after bounded parsing;
- the public application behavior is documented and testable.

The implementation MUST NOT claim that a Rust HTTP/3 stack matches a browser fingerprint without measurement.

## 17. Close behavior

An authenticated `CLOSE` begins graceful shutdown. A peer:

- stops accepting new inner packets;
- may drain queued protected records within a deadline;
- acknowledges close if the selected state machine requires it;
- deletes secrets;
- closes the carrier;
- triggers local route rollback outside protocol logic.

Abrupt carrier closure discards incomplete reassemblies and session secrets.

## 18. Error policy

Before authentication, public behavior must be indistinguishable from ordinary decoy handling within practical implementation limits.

After authentication, peers may exchange coarse error categories. Detailed stack traces, parser offsets useful only to developers, key state, and secret-dependent failures remain local.

## 19. Extensibility

Extensions require:

- assigned type or feature identifiers;
- mandatory/optional classification;
- bounded parse and skip rules;
- downgrade considerations;
- test vectors;
- specification update and ADR.
