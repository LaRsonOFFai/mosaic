# AGENTS.md

## Project mission

MOSAIC is an experimental, research-oriented encrypted tunneling
protocol and traffic-shaping simulator.

The current goal is not to build or deploy a production VPN.
The current goal is to develop a testable protocol core in an
isolated local environment.

Never describe the protocol as undetectable. Use the terms
"traffic indistinguishability experiment" and "resistance to
classification".

## Required reading

Before changing code, read:

- README.md
- docs/SPEC.md
- docs/THREAT_MODEL.md
- docs/MILESTONES.md
- relevant documents in docs/adr/

When behavior changes, update the corresponding documentation.

## Scope restrictions

Unless the current task explicitly says otherwise:

- Do not create a TUN/TAP interface.
- Do not modify operating-system routes.
- Do not modify firewall rules.
- Do not run commands with sudo or administrator privileges.
- Do not contact external servers.
- Do not implement deployment or persistence.
- Do not add domain fronting or third-party service impersonation.
- Do not implement custom cryptographic primitives.
- Do not hard-code credentials, keys, tokens, or IP addresses.

All testing must use unit tests, in-memory transports, or localhost.

## Engineering rules

- Use stable Rust.
- Avoid unsafe Rust. Any unsafe block requires a written justification.
- Treat all received bytes as untrusted.
- Every allocation based on network input must have a strict limit.
- Decoders must reject malformed and non-canonical input.
- Decoders must never panic on arbitrary input.
- Keep protocol logic independent from the transport implementation.
- Keep cryptography independent from framing and scheduling.
- Use dependency injection for clocks and random-number generation.
- Prefer deterministic tests.
- Do not silently ignore errors.
- Public types and public functions require documentation.
- Keep commits and diffs focused on the current milestone.

## Protocol design rules

- Do not expose destinations, channel types, sequence numbers, or
  control operation types outside authenticated encryption.
- Internal cell boundaries must not be required to match transport
  message boundaries.
- The streaming decoder must support fragmented and coalesced input.
- Protocol versions and limits must be explicitly represented.
- Unknown mandatory features must cause a clean rejection.
- Unknown optional features may be ignored only when the specification
  explicitly permits it.
- Padding must be validated and bounded.
- Resource exhaustion is part of the threat model.

## Cryptography rules

- Do not invent encryption, hashing, key exchange, or signature algorithms.
- Cryptographic choices require a separate ADR.
- Secret material must not be printed or logged.
- Authentication failures must not expose secret-dependent diagnostics.
- Tests may use deterministic test keys, but production code must not
  contain fixed secrets.

## Required verification

Before declaring a task complete, run:

```sh
cargo fmt --all --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo test --workspace --all-features
