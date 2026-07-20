# MOSAIC Milestones

Only one milestone is implemented per Codex task and Git commit.

## M0 — Repository bootstrap

Deliverables:

- Cargo workspace;
- initial crates required for M1;
- toolchain and lint policy;
- CI for formatting, linting, and tests;
- ADR index and initial architecture ADR;
- no functional protocol implementation.

Exit criteria:

- workspace builds;
- tests pass;
- docs remain internally consistent.

## M1 — Canonical cell codec

Deliverables:

- accepted codec ADR;
- versioned cell types;
- canonical encoder;
- bounded streaming decoder;
- malformed-input tests;
- property tests;
- fuzz target.

No encryption, sockets, TUN, or scheduler.

## M2 — Packet pipeline

Deliverables:

- validated IPv4/IPv6 packet wrapper;
- packet IDs;
- fragmentation;
- bounded out-of-order reassembly;
- queue limits and drop reasons;
- deterministic clock;
- memory packet device;
- simulation tests for loss, duplicate, reorder, and timeout.

No cryptography, live sockets, or TUN.

## M3 — Session cryptography

Deliverables:

- cryptographic ADR;
- bounded handshake framing;
- server authentication and client authorization;
- directional traffic keys;
- protected records;
- replay window;
- key epochs;
- secret redaction and zeroization where practical;
- negative and vector-style tests.

No external networking or TUN.

## M4 — Local client and server

Deliverables:

- carrier abstraction finalized;
- memory and loopback-only reliable carrier;
- client/server orchestration;
- synthetic packet generator and sink;
- complete encrypted IPv4/IPv6 round trip;
- shutdown and backpressure tests;
- CLI restricted to localhost by default.

No TUN, default-route changes, or HTTP/3.

## M5 — Linux TUN integration

Deliverables:

- separate Linux TUN crate;
- PacketDevice implementation;
- explicit MTU handling;
- opt-in privileged integration test in a network namespace;
- cleanup and recovery documentation;
- optional route-plan dry run.

The host default route is not changed by automated tests.

## M6 — HTTP/3 carrier and decoy service

Deliverables:

- HTTP/3 dependency ADR;
- valid TLS verification;
- operator-controlled decoy application;
- bounded authenticated initiation;
- ordinary decoy behavior for unauthenticated requests;
- local certificate integration tests;
- no domain fronting or third-party service impersonation.

## M7 — Scheduler and cover profiles

Deliverables:

- scheduler ADR;
- fast, balanced, and strict modes;
- versioned bounded profile format;
- split/coalesce/padding/delay/interleave operations;
- deterministic simulation;
- overhead, queue, and latency metrics;
- proof by tests that protected bytes are conserved.

## M8 — Evaluation harness

Deliverables:

- reproducible local capture workflow;
- feature extraction from metadata;
- baseline and mode datasets;
- classifier and performance reports;
- TPR/FPR and ROC-AUC;
- documented limitations;
- no universal detection-resistance claim.

## M9 — Hardening and release candidate

Deliverables:

- consolidated security review;
- dependency audit;
- extended fuzzing instructions and results available in the environment;
- configuration validation;
- route guard and rollback hardening if route management exists;
- operational recovery guide;
- interoperability fixtures;
- release checklist;
- explicit list of unresolved risks.

A release candidate is not a production-security guarantee.
