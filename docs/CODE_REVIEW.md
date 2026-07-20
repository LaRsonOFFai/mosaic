# MOSAIC Code Review Guide

Review findings are prioritized as:

- Critical — key compromise, arbitrary code execution, privilege boundary failure, route lockout without recovery, or unauthenticated remote control.
- High — nonce reuse, authentication bypass, unbounded remote allocation, replay delivery, parser panic reachable from remote input, or secret leakage.
- Medium — inconsistent specification, incomplete bounds, unsafe error behavior, race, deadlock, cleanup failure, or test gap affecting security claims.
- Low — maintainability, diagnostics, minor inefficiency, or documentation issue without immediate security impact.

## Required review areas

### Parsing

- length checked before allocation and slicing;
- checked arithmetic;
- canonical encoding;
- no partial delivery on failure;
- bounded buffered partial input;
- correct trailing-byte behavior;
- no panic on malformed input.

### Fragmentation and reassembly

- overlap and duplicate policy;
- global and per-entry limits;
- timeout correctness;
- exactly-once delivery;
- packet-ID reuse;
- no quadratic attacker-controlled work.

### Cryptography

- standard construction and correct library usage;
- unique nonces;
- direction separation;
- transcript binding;
- replay window;
- epoch transition;
- constant-time secret comparison where required;
- secret redaction and deletion.

### Concurrency

- bounded channels;
- cancellation safety;
- no deadlock on shutdown;
- backpressure propagation;
- no detached task retaining secrets or routes;
- deterministic state transitions.

### TUN and routes

- explicit privilege requirement;
- server route protection;
- rollback recorded before mutation;
- reverse-order cleanup;
- signal and error handling;
- tests isolated from host network;
- no hidden DNS/firewall modifications.

### HTTP/3 and decoy

- certificate validation;
- bounded request bodies and streams;
- no tunnel-specific unauthenticated response;
- no third-party infrastructure assumptions;
- decoy still functions normally;
- authentication before tunnel dispatch.

### Scheduler

- protected-byte conservation;
- bounded delay and padding;
- finite-state progress;
- no payload inspection;
- no unbounded cover traffic;
- metrics match actual behavior.

### Tests

- assertions verify behavior rather than only absence of errors;
- negative cases are present;
- test-only insecure configuration cannot leak into release;
- flaky sleep-based tests avoided;
- required commands actually ran.
