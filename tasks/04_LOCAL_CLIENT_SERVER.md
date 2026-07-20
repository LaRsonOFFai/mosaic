# Task M4 — Local client/server and carrier contract

## Operating mode

Plan first. Finalize carrier semantics before choosing runtime details.

## Goal

Create a complete loopback-only encrypted client/server prototype using synthetic packet sources and sinks.

## Required deliverables

1. Finalize the `mosaic-carrier` contract and document it in `docs/adr/0005-carrier-contract.md`.
2. Implement:
   - deterministic in-memory carrier;
   - reliable ordered localhost carrier;
   - arbitrary read/write splitting;
   - bounded send and receive buffers;
   - backpressure;
   - orderly and abrupt close.
3. Add `mosaic-client` and `mosaic-server` orchestration crates.
4. Connect packet device, pipeline, session, and carrier through bounded channels or an equivalent bounded design.
5. Add client/server state machines and graceful shutdown deadlines.
6. Add CLI subcommands for:
   - local test server bound to loopback only;
   - local synthetic client;
   - deterministic generated IPv4/IPv6 packets;
   - metrics summary.
7. Update README with exact local test commands.

## Runtime requirements

If an async runtime is introduced:

- justify it in the ADR;
- keep runtime-specific types outside protocol-core public APIs when practical;
- use bounded channels;
- handle cancellation;
- ensure no detached task survives shutdown;
- avoid sleep-based correctness tests by using simulation where possible.

## Required tests

- one and many packet round trips;
- both directions simultaneously;
- random carrier chunk boundaries;
- backpressure with bounded memory;
- abrupt client and server close;
- graceful drain deadline;
- malformed peer bytes;
- authentication failure;
- reconnect starts fresh replay and key state;
- no packet delivered twice;
- metrics equal actual packet/byte counts;
- loopback binding is enforced by default.

## Constraints

- Bind only to `127.0.0.1` and `::1` by default.
- No TUN.
- No route, DNS, or firewall changes.
- No HTTP/3 or TLS carrier.
- No external hosts.
- No scheduler beyond direct bounded record transfer.
- Do not add a generic public proxy.

## Acceptance criteria

Two local processes or an integration test must exchange encrypted synthetic IPv4 and IPv6 packets end to end. Memory remains bounded under backpressure. Required commands pass and only M4 is implemented.
