# Task M6 — HTTP/3 carrier and operator-controlled decoy

## Operating mode

Plan and dependency ADR first. Use only operator-controlled infrastructure. Do not implement domain fronting or third-party service impersonation.

## Goal

Add a standards-compliant HTTP/3 carrier that transports MOSAIC handshake/protected bytes inside a harmless decoy application owned by the operator.

## Required ADR

Create `docs/adr/0007-http3-carrier.md` comparing maintained Rust HTTP/3/QUIC options and deciding:

- library stack and enabled features;
- async runtime integration;
- TLS certificate validation;
- ALPN and connection lifecycle;
- request/stream mapping;
- maximum body and stream sizes;
- backpressure;
- reconnect policy;
- decoy dispatch boundary;
- unauthenticated behavior;
- test-certificate strategy;
- known fingerprint limitations.

The ADR must explicitly state that standards compliance does not prove browser-fingerprint equivalence.

## Decoy application

Implement a small harmless operator-controlled application, for example a minimal note synchronization demo with:

- ordinary public health/about endpoint;
- bounded create/list/read operations using test-only local storage;
- normal HTTP status codes and bodies;
- no claim to imitate a named third-party product.

Tunnel initiation is an authenticated opaque application operation. Invalid or absent MOSAIC authorization must follow ordinary decoy behavior and must not emit proxy-specific status codes, tunnel banners, detailed parser errors, or TLS validation bypasses.

## Required implementation

1. Add `mosaic-carrier-h3`.
2. Implement the existing carrier contract over HTTP/3.
3. Keep protocol cells and keys unknown to the generic HTTP/3 transport layer except at the explicit authenticated dispatch boundary.
4. Bound request bodies, concurrent streams, pending handshakes, and queued bytes.
5. Preserve certificate and hostname verification on the client.
6. Support graceful reconnect with fresh session state.
7. Keep the loopback/local carrier available for deterministic tests.
8. Add client/server CLI configuration for a user-supplied operator-controlled hostname, certificate trust configuration suitable for tests, and secrets loaded from files or secure input rather than source code.
9. Redact headers and authorization material from logs.
10. Update deployment documentation for a lab-owned server only.

## Required tests

- decoy works without MOSAIC credentials;
- valid authenticated initiation establishes a session;
- invalid initiation returns ordinary bounded decoy behavior;
- malformed/oversized bodies are handled without tunnel-specific detail;
- invalid certificate and hostname are rejected;
- local test certificate succeeds only when explicitly trusted for the test;
- arbitrary stream chunking;
- backpressure and concurrent stream limits;
- reconnect creates new keys and replay state;
- existing M4 carrier contract tests pass against H3 where applicable;
- no external third-party host is contacted by automated tests.

## Constraints

- No domain fronting.
- No third-party CDN/service impersonation.
- No disabled TLS validation.
- No automated public deployment.
- No network scanning.
- No scheduler shaping beyond minimal carrier framing; that is M7.

## Acceptance criteria

A client and operator-controlled local/test server transfer encrypted synthetic packets and optional TUN packets over HTTP/3 while the normal decoy application remains usable. Unauthenticated requests reveal no explicit tunnel interface. Required commands pass and only M6 is implemented.
