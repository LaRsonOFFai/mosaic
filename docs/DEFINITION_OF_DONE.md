# Definition of Done

A milestone is complete only when all applicable items are satisfied.

## Scope

- Only the requested milestone was implemented.
- No placeholder is presented as complete functionality.
- No future milestone was silently pulled in.

## Design

- Public interfaces match `docs/ARCHITECTURE.md` or the document was updated with an ADR.
- Wire or cryptographic changes have an accepted ADR.
- Resource limits and failure behavior are explicit.

## Implementation

- Code compiles on the supported stable toolchain.
- Input paths have no panic-based error handling.
- Queues, allocations, retries, and timeouts are bounded.
- Secrets are redacted.
- Platform-specific and privileged code stays isolated.

## Tests

- Positive, negative, boundary, and malformed-input tests exist.
- Property/fuzz targets exist where required.
- Tests assert meaningful outcomes.
- Privileged tests are opt-in.

## Documentation

- Specification matches implementation.
- README contains run/test instructions for the implemented scope.
- Known limitations and recovery steps are documented.

## Verification

At minimum:

```sh
cargo fmt --all --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo test --workspace --all-features
```

Any skipped command is reported with the exact reason. A failed required command means the milestone is not done.

## Final report

The Codex response lists:

- files changed;
- ADRs added or updated;
- tests added;
- commands and results;
- security checks;
- known limitations;
- confirmation of milestone boundary.
