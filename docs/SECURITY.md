# MOSAIC Security Requirements

## 1. Input handling

All bytes from packet devices, carriers, configuration files, cover profiles, IPC, and peers are untrusted.

Required practices:

- check lengths before slicing or allocation;
- use checked addition and multiplication;
- reject impossible conversions;
- cap nesting, counts, and buffered partial input;
- avoid recursive parsers;
- never loop without consuming input or advancing state;
- keep error paths free of panics;
- do not expose partially authenticated data.

## 2. Resource limits

Every component must have explicit limits:

- codec buffer;
- handshake buffer;
- incomplete records;
- incomplete packet reassemblies;
- bytes per reassembly;
- global reassembly bytes;
- queued packets and bytes;
- scheduler delayed events;
- concurrent sessions;
- authentication failures per interval;
- decoy request body size;
- log event rate.

Limits must be validated at startup and included in metrics.

## 3. Cryptography

- No custom cryptographic algorithm.
- Use maintained RustCrypto, Noise-framework, or equivalent reviewed implementations selected by ADR.
- Pin only compatible versions through `Cargo.lock`; do not invent local forks without review.
- Use OS cryptographic randomness for production keys and nonces where required.
- Use independent traffic keys for each direction.
- Bind version, role, features, and relevant limits to the authenticated transcript.
- Include public record headers in AEAD associated data.
- Delete obsolete key material after the permitted transition window.
- Redact secrets from `Debug`, `Display`, logs, metrics, panic reports, and config dumps.

## 4. Authentication behavior

- Before a peer is authenticated, do not reveal detailed parser or authentication errors remotely.
- Do not use Basic Authentication as the tunnel's cryptographic authorization mechanism.
- Do not compare secret tokens with ordinary early-exit string comparison.
- Replay protection is mandatory for reusable initiation messages.
- Rate-limit failed initiation attempts without creating a unique public banner.

## 5. Dependency policy

For each new security-sensitive dependency, the implementing task must record:

- purpose;
- maintenance status checked by the developer;
- feature flags enabled;
- default features disabled or retained and why;
- unsafe-code exposure if known;
- license compatibility;
- alternatives considered.

Run available dependency auditing tools in hardening milestone. A clean audit does not replace code review.

## 6. Logging

May log:

- session-local random identifier;
- state transition category;
- byte/packet counts;
- queue and latency summaries;
- coarse error category;
- peer address only when explicitly enabled.

Must not log:

- inner packet bytes;
- destination addresses by default;
- private keys, PSKs, tokens, cookies, or complete headers;
- AEAD nonces together with keys;
- raw handshake payloads;
- decoy authentication bodies.

## 7. TUN and routes

- TUN tests are opt-in.
- Default unit tests never need root.
- Route changes occur only after explicit validation.
- Protect the server route before changing broader routes.
- Record rollback steps before applying each change.
- Roll back in reverse order.
- Handle signals and normal shutdown.
- Document recovery for process crash.
- Never silently alter DNS.

## 8. HTTP/3 service

- Require certificate validation on the client.
- Use operator-controlled hostnames and certificates.
- Keep decoy and tunnel request-size limits finite.
- Do not accept unbounded streaming bodies before authentication.
- Do not return stack traces.
- Keep tunnel dispatch behind successful cryptographic verification.
- Preserve normal decoy behavior for ordinary requests.
- No domain fronting or unrelated third-party infrastructure.

## 9. Scheduler

- Padding and delays have hard budgets.
- Strict mode must still enforce liveness and shutdown deadlines.
- Profile parsing is bounded.
- Profile state machines reject unreachable or non-progress cycles unless explicitly allowed with finite timers.
- Random choices use a suitable source; deterministic seeded randomness is test-only or simulation-only.
- The scheduler never invents acknowledgements for data not accepted by the carrier.

## 10. Release boundary

Before calling any build production-ready, require:

- independent cryptographic review;
- independent parser and TUN review;
- reproducible builds or documented build provenance;
- dependency audit;
- fuzzing over meaningful duration;
- interoperability tests;
- incident and key-rotation procedures;
- clear statement of unsupported security claims.
