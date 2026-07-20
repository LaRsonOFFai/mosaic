# MOSAIC

MOSAIC is an experimental encrypted IP tunneling protocol with pluggable carriers, bounded packet fragmentation/reassembly, and an optional traffic-shaping scheduler.

The project is intended for reproducible research on localhost, isolated lab networks, and operator-controlled infrastructure. It does not claim guaranteed invisibility, anonymity, or universal filtering bypass.

## Current state

The repository initially contains specifications and Codex task files only. Implementation proceeds milestone by milestone from `tasks/00_BOOTSTRAP.md` through `tasks/09_HARDENING.md`.

## Design principles

- protocol core independent from TUN and transports;
- standard cryptography only;
- explicit resource limits;
- malformed input must not panic or allocate without bounds;
- deterministic tests before privileged integration;
- no third-party impersonation or domain fronting;
- every detection-resistance claim must be measured.

## Required documents

Start with:

- `AGENTS.md`
- `docs/PRODUCT_REQUIREMENTS.md`
- `docs/ARCHITECTURE.md`
- `docs/PROTOCOL_SPEC.md`
- `docs/THREAT_MODEL.md`
- `docs/MILESTONES.md`

## Development flow

Implement one task at a time. Plan first, implement second, review uncommitted changes, run all checks, then create one Git commit for the milestone.
