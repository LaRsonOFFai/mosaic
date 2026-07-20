# Task M9 — Hardening and research release candidate

## Operating mode

Begin with a repository-wide plan and gap analysis. Do not add unrelated features.

## Goal

Consolidate security, operability, documentation, and reproducibility for a research release candidate.

## Required work

### 1. Specification consistency

- Compare code, ADRs, protocol spec, architecture, CLI help, and tests.
- Resolve every mismatch or document it as an explicit known limitation.
- Freeze a draft protocol version and generate stable wire fixtures.

### 2. Security review

Perform focused reviews of:

- codec and public framing;
- fragmentation/reassembly;
- session and nonces;
- replay/epoch handling;
- concurrency/backpressure;
- TUN and route rollback;
- HTTP/3 authentication boundary;
- decoy behavior;
- scheduler bounds;
- secret logging/configuration.

Findings must be prioritized and fixed with regression tests where feasible.

### 3. Dependency review

- inventory dependencies and enabled features;
- remove unused dependencies;
- run available audit/license tools;
- document unresolved advisories or unavailable tools;
- ensure lockfile policy is documented.

### 4. Fuzzing and robustness

- ensure all required fuzz targets build;
- run meaningful smoke fuzzing available in the environment;
- add regression seeds for found failures;
- document longer recommended campaigns without claiming they were run when they were not.

### 5. Configuration and secrets

- strict config validation;
- secret redaction;
- safe file-permission guidance;
- no production defaults containing credentials;
- clear separation of test and release configuration.

### 6. Route and recovery hardening

If automatic route application exists:

- require explicit confirmation flags;
- verify server-route protection;
- transaction-like apply/rollback order;
- signal/error cleanup;
- dry-run output;
- manual emergency recovery guide;
- injected-failure tests.

If automatic route application does not exist, document exact manual lab setup and keep the limitation explicit.

### 7. Interoperability and fixtures

- stable encoded cell fixtures;
- protected-record fixtures that do not expose real secrets;
- client/server compatibility test;
- version rejection and feature-negotiation tests.

### 8. Documentation

Add or finalize:

- build and test guide;
- local unprivileged demo;
- isolated Linux TUN demo;
- operator-controlled HTTP/3 lab demo;
- configuration reference;
- recovery guide;
- security assumptions;
- evaluation reproduction;
- known limitations;
- release checklist.

### 9. Final validation

Run all available:

```sh
cargo fmt --all --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo test --workspace --all-features
```

Also run milestone-specific property, integration, audit, and fuzz smoke checks. Record exact commands and results.

## Constraints

- No new transport or platform.
- No domain fronting.
- No third-party service imitation.
- No persistence or installer with elevated privileges.
- No marketing claim of invisibility or universal bypass.
- Do not hide failed checks.

## Acceptance criteria

The repository has a coherent draft specification, passing required checks, documented security boundaries, reproducible demos/evaluation, rollback/recovery instructions, and an explicit unresolved-risk list. The result is called a research release candidate, not production-proven security.
