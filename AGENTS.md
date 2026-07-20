# AGENTS.md

## Project mission

MOSAIC is a research-oriented encrypted IP tunneling protocol with pluggable carriers and an experimental traffic scheduler.

The project studies resistance to traffic classification. Never describe the protocol as invisible, undetectable, guaranteed to bypass filtering, or anonymous.

The project may only be tested on localhost, isolated lab networks, or infrastructure controlled by the operator.

## Required reading

Before changing code, read:

1. `README.md`;
2. `docs/PRODUCT_REQUIREMENTS.md`;
3. `docs/ARCHITECTURE.md`;
4. `docs/PROTOCOL_SPEC.md`;
5. `docs/THREAT_MODEL.md`;
6. `docs/SECURITY.md`;
7. `docs/TEST_PLAN.md`;
8. `docs/MILESTONES.md`;
9. `docs/DEFINITION_OF_DONE.md`;
10. all accepted ADRs relevant to the current change;
11. the current file in `tasks/`.

When behavior changes, update the specification and tests in the same change.

## Milestone discipline

- Implement exactly one milestone per task.
- Do not start later milestones early.
- Do not broaden scope merely because a future feature would be convenient.
- Before implementation, provide a plan listing files, public interfaces, tests, risks, and unresolved decisions.
- Architectural or wire-format decisions require an ADR.
- Stop and report when the current milestone is complete.

## Hard scope restrictions

Unless the current milestone explicitly permits it:

- do not create TUN or TAP interfaces;
- do not modify routes, DNS configuration, firewall rules, or kernel settings;
- do not run commands with `sudo`, administrator rights, or privilege escalation;
- do not contact arbitrary external hosts;
- do not scan networks or endpoints;
- do not add persistence, autostart, concealment, or self-installation;
- do not implement domain fronting;
- do not impersonate third-party services;
- do not use third-party domains, CDNs, credentials, or accounts;
- do not bypass authentication or authorization;
- do not add packet capture outside an explicit local evaluation task;
- do not invent cryptographic primitives;
- do not hard-code real credentials, private keys, tokens, domains, or public IP addresses.

Package-registry access may be used only when the execution environment permits it and only for dependencies justified by the current milestone.

## Architecture rules

- The protocol core must not depend on Linux TUN, routing APIs, HTTP/3, or a specific async runtime.
- TUN is an adapter implementing the packet-device abstraction.
- A carrier is an adapter implementing the carrier abstraction.
- Codec, reassembly, session cryptography, scheduling, packet devices, and carriers are separate layers.
- Inner packet boundaries must not be required to match outer carrier-event boundaries.
- Carrier adapters must not inspect decrypted IP payloads unless explicitly required and documented.
- The scheduler must consume opaque encrypted records or metadata that does not expose user payload contents.
- Platform-specific code belongs in platform-specific crates.
- Route management must be separate from protocol logic and must support rollback.

## Rust rules

- Use stable Rust.
- Prefer edition 2024 when supported by the installed stable toolchain; otherwise use the newest supported stable edition and document the decision.
- Avoid `unsafe`. Any `unsafe` block requires a dedicated safety comment, tests, and ADR justification.
- No `unwrap`, `expect`, indexing panic, or `panic!` on paths reachable from untrusted input.
- Public APIs require rustdoc.
- Errors must be typed and actionable without exposing secrets.
- Network and file input are untrusted.
- All allocations, queues, loops, retries, timeouts, and reassembly state require explicit limits.
- Use checked arithmetic for input-derived sizes and offsets.
- Do not silently truncate values.
- Prefer deterministic tests by injecting clocks and random-number generators.
- Keep protocol-core tests independent of root privileges and live networks.

## Protocol rules

- Wire encoding must be canonical and versioned.
- Unknown mandatory features cause a clean rejection.
- Unknown optional features are ignored only when the specification explicitly allows it.
- Decoder behavior for truncation, overflow, duplicate fields, invalid flags, and trailing bytes must be tested.
- Internal cells and control operations are encrypted after session establishment.
- Length information exposed outside encryption must be minimized, authenticated where applicable, and documented.
- Nonce reuse is prohibited.
- Replay handling and key epochs must be explicit.
- Fragment reassembly must reject overlap, inconsistent total length, duplicate conflicts, and resource exhaustion.
- Queue overflow behavior must be deterministic and documented.

## Cryptography rules

- Use maintained, reviewed implementations of standard constructions.
- Cryptographic dependencies and protocol choices require an ADR.
- Secret material must be zeroized where practical and never logged.
- Authentication failures must not include secret-dependent diagnostics.
- Production code must not contain fixed secrets or deterministic nonces.
- Test-only fixed keys must be clearly marked and impossible to enable accidentally in release builds.
- Key direction, epoch, packet number, and nonce derivation must be tested.

## TUN and routing rules

These rules apply only from milestone M5 onward:

- Keep the TUN adapter in a separate crate.
- Do not change the host default route during automated tests.
- Prefer isolated Linux network namespaces for integration tests.
- Route changes require an explicit CLI flag and a second confirmation flag.
- The server endpoint must be excluded from the tunnel route before enabling a default route.
- Every applied change must have a recorded rollback action.
- Normal shutdown must attempt rollback.
- Documentation must include recovery commands for abnormal shutdown.
- Tests that require privileges must be opt-in and skipped by default.

## HTTP/3 and decoy rules

These rules apply only from milestone M6 onward:

- Use a standards-compliant maintained implementation.
- Do not claim browser fingerprint equivalence unless measured.
- Use only operator-controlled domains and servers.
- No domain fronting, SNI manipulation against third parties, or third-party service impersonation.
- The decoy service must be a real, harmless application with documented public behavior.
- Invalid or unauthenticated tunnel input must follow ordinary decoy behavior without tunnel-specific error messages.
- Do not weaken TLS certificate validation.

## Required verification

Before declaring a task complete, run all commands that exist for the current milestone:

```sh
cargo fmt --all --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo test --workspace --all-features
```

For codec, reassembly, and cryptographic changes, also run the available property tests and fuzz smoke tests. Fuzzing tools that require a separate toolchain may be reported as not installed, but the fuzz targets must still be buildable in the documented environment.

For documentation-only changes, check links, headings, task numbering, and consistency manually.

## Final report format

Every implementation response must include:

1. summary of the implemented milestone;
2. changed and created files;
3. important design decisions and ADRs;
4. commands executed and their exact result;
5. security checks performed;
6. known limitations;
7. confirmation that no later milestone was implemented.

Do not claim success when a required command was not run or failed. State the exact limitation honestly.
