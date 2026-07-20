```markdown
# Threat model

## Protected assets

- Contents of tunneled application traffic.
- Destination addresses and channel metadata.
- Session authentication material.
- Association between internal packets and external carrier events.

## Adversary capabilities

The evaluated adversary may:

- observe packet sizes, directions, timing, and connection duration;
- record and replay traffic;
- drop, delay, reorder, duplicate, or corrupt packets;
- initiate connections to the public server;
- send malformed protocol messages;
- train classifiers on labeled traffic captures.

## Initially out of scope

The first prototype does not claim protection against:

- blocking the server IP address;
- blocking the server domain;
- a global observer watching both ends;
- compromise of the client or server;
- traffic analysis based on user behavior over long periods;
- allowlist-only networks;
- coercion or legal restrictions.

## Security objectives

- Invalid input never causes a panic or unbounded allocation.
- A passive observer cannot read internal channel metadata.
- Internal packet boundaries do not directly determine outer boundaries.
- Unauthenticated probes receive no tunnel-specific information.
- Replay attempts are rejected after session authentication is added.
- Every detection-resistance claim must be supported by an experiment.

## Non-objectives

- The protocol is not described as invisible or undetectable.
- The protocol does not provide anonymity by itself.
- The protocol does not replace endpoint security.
