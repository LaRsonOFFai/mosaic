# Task M1 — Canonical cell codec

## Operating mode

Plan first. Do not write code until the codec ADR proposal and implementation plan are presented.

## Goal

Implement a canonical, bounded, streaming codec for plaintext MOSAIC batches and the initial cell types.

## Required context

Read all project documents and the accepted M0 ADR. Reconcile any conflict before implementation.

## Design step

Create `docs/adr/0002-cell-codec.md`. Compare at least:

- fixed-width integers;
- bounded canonical unsigned varints;
- TLV framing.

Evaluate:

- strict canonical decoding;
- extension behavior;
- allocation safety;
- streaming implementation complexity;
- encoded overhead;
- fuzzability;
- ability to skip optional values safely.

The ADR must select exact field ordering, integer widths/limits, error behavior, and trailing-byte policy. Update `docs/PROTOCOL_SPEC.md` to match.

## Required implementation

In `mosaic-core` define or refine:

- protocol version types;
- validated limit types;
- cell type identifiers;
- typed shared errors where appropriate.

In `mosaic-codec` implement:

- `CodecLimits` with validated finite limits;
- `Batch`;
- `Cell` enum with `DataFragment`, `Control`, and `Padding`;
- canonical encoder;
- stateful streaming decoder;
- precise decode errors;
- encoded-length calculation without unchecked overflow;
- support for arbitrary input fragmentation and coalescing;
- decoder reset/terminal-error policy documented in rustdoc.

## Required security behavior

- Validate declared lengths before allocation.
- Never buffer more than the configured hard maximum.
- No `unwrap`, `expect`, direct indexing panic, or `panic!` on untrusted input.
- Reject non-canonical encodings.
- Reject unknown mandatory types.
- Optional extension skipping is allowed only if the ADR defines it safely.
- Reject mismatched lengths and invalid flags.
- Do not deliver cells from a malformed batch.

## Tests

Add:

- round-trip tests for every type;
- minimum and maximum boundary tests;
- one-byte-at-a-time streaming test;
- test for every split position of representative messages;
- multiple batches/cells in one chunk as supported by the selected framing;
- truncation at every byte boundary;
- over-limit length before allocation;
- integer-overflow attempts;
- invalid type/flag tests;
- non-canonical encoding tests;
- trailing-byte tests;
- arbitrary-input no-panic property test;
- encoded-length property;
- fuzz target for the decoder.

Do not use snapshots as the only wire-format test. Add explicit stable byte fixtures for selected examples.

## Constraints

- No encryption.
- No handshake.
- No sockets or async runtime unless already justified by M0.
- No TUN.
- No fragmentation algorithm or reassembly state; only encode/decode fragment fields.
- No scheduler.

## Acceptance criteria

- specification and ADR define exact M1 wire bytes;
- decoder handles one byte at a time;
- oversized declarations are rejected before proportional allocation;
- malformed input cannot cause a panic in tests/property tests;
- fuzz target is present and documented;
- required workspace commands pass;
- only M1 is implemented.
