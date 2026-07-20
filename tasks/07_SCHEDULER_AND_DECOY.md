# Task M7 — Bounded scheduler and cover profiles

## Operating mode

Plan and ADR first. Treat classification resistance as an experiment, not a guaranteed property.

## Goal

Add a scheduler between protected records and carrier events that can split, coalesce, interleave, delay, and pad within strict budgets defined by versioned cover profiles.

## Required ADR

Create `docs/adr/0008-scheduler-and-profile.md` defining:

- scheduler input/output contract;
- preservation of protected byte streams;
- delivery classes and deadlines;
- fast, balanced, and strict modes;
- profile data format;
- state-machine semantics;
- event-size and delay distributions;
- padding representation and removal;
- queue fairness;
- random-source policy;
- bounded progress and shutdown;
- metrics;
- receiver reconstruction behavior.

## Cover profile requirements

A profile must be versioned and bounded. It may define:

- states;
- allowed directions;
- event-size ranges or discrete distributions;
- inter-event delay ranges/distributions;
- burst-length distributions;
- state transitions;
- maximum residence time;
- maximum queue delay;
- padding byte budget;
- idle behavior;
- maximum concurrent logical event streams if supported.

Reject:

- too many states or transitions;
- invalid probabilities/weights;
- impossible size ranges;
- zero-progress cycles without finite timeout;
- budgets above implementation ceilings;
- unknown mandatory fields.

## Required implementation

1. Add `mosaic-scheduler`.
2. Implement deterministic simulated scheduling with injected clock and seeded RNG.
3. Implement fast mode with minimal transformation.
4. Implement balanced mode with bounded coalescing, splitting, and modest padding/delay.
5. Implement strict mode driven by a profile state machine.
6. Preserve protected bytes exactly at the receiver.
7. Add delivery classes for control, latency-sensitive, and bulk data without inspecting decrypted payloads in the scheduler.
8. Add queue, delay, padding, and event metrics.
9. Integrate with local and HTTP/3 carriers behind configuration.
10. Provide at least one synthetic example profile clearly labeled as a laboratory profile, not a claim to imitate a named service.

## Required tests

- byte conservation under random split/coalesce/interleave sequences;
- exact reconstruction with arbitrary carrier chunking;
- deterministic output for fixed seed;
- queue/byte limits;
- padding budget;
- deadline enforcement;
- control traffic priority within reserved bounds;
- profile parse fuzz target;
- invalid profile matrix;
- no infinite transition loop;
- shutdown deadline;
- fast mode functional equivalence;
- metrics reconcile with emitted events and bytes.

## Constraints

- Scheduler must not read inner packet payloads.
- No profile named after or copied from a third-party service.
- No automatic collection of other users' traffic.
- No claim that a profile defeats DPI.
- No domain fronting or external scanning.
- Do not weaken session cryptography or TLS.

## Acceptance criteria

All carriers can transfer identical protected data through disabled/fast, balanced, and strict scheduler modes. Limits and conservation properties are tested. Metrics quantify overhead and delay. Only M7 is implemented.
