# Milestones

## M0 — Specification scaffold

- Repository structure.
- Threat model.
- Terminology.
- Initial architecture.
- ADR template.
- No protocol implementation.

## M1 — Cell codec

- Versioned cell representation.
- Canonical encoding.
- Streaming decoder.
- Configurable size limits.
- Round-trip and malformed-input tests.
- Fuzz target.
- No encryption.
- No networking.

## M2 — Multiplexer

- Multiple logical channels.
- Reliable and unreliable channel metadata.
- Fair bounded queues.
- Fragmentation and reassembly.
- Deterministic simulation tests.

## M3 — Behaviour scheduler

- Abstract carrier-event interface.
- State-machine profile.
- Size and timing distributions.
- Bounded delay and padding budgets.
- Deterministic clock.
- Statistical reports from the simulator.

## M4 — Session cryptography

- Separate cryptographic ADR.
- Standard authenticated-encryption library.
- Direction-separated keys.
- Epoch and nonce management.
- Replay protection.
- Test vectors and negative tests.

## M5 — Local carrier

- Client and server communicating over localhost.
- Transport-independent core.
- Fragmentation and loss simulation.
- No TUN and no routing changes.

## M6 — HTTP/3 carrier experiment

- Adapter around a standards-compliant implementation.
- No custom QUIC cryptography.
- Ordinary decoy application remains functional.
- Capture and compare observable traffic properties.

## M7 — TUN integration

- Linux-only experimental client.
- Explicit user action required.
- Routes restored after shutdown.
- No firewall modification without separate approval.
- Integration tests in an isolated network namespace.

## M8 — Evaluation

- Labeled packet captures.
- Baseline traffic.
- Tunnel traffic.
- Classifier evaluation.
- False-positive and true-positive measurements.
- Performance and bandwidth-overhead reports.
