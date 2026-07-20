# Task M3 — Session cryptography and protected records

## Operating mode

Plan and ADR first. Do not invent cryptographic constructions.

## Goal

Add authenticated session establishment and protected records around M1/M2 cells using maintained standard cryptographic libraries.

## Required design ADR

Create `docs/adr/0004-session-cryptography.md` comparing at least two suitable standard approaches, such as a maintained Noise-framework implementation and a standard HPKE/TLS-exporter-based design where applicable.

The ADR must decide:

- exact standard handshake pattern or construction;
- server authentication method;
- client authorization method;
- dependency and enabled features;
- transcript-bound fields;
- traffic-key derivation;
- direction separation;
- protected-record public header;
- nonce construction;
- replay window;
- key epoch/update behavior;
- maximum handshake and record sizes;
- secret storage and zeroization;
- failure behavior.

Do not accept a design that relies on a custom cipher, custom Diffie-Hellman, unauthenticated encryption, reusable raw bearer-token comparison, or deterministic production nonces.

## Required implementation

Add `mosaic-session` crate containing:

- client and server handshake state machines;
- bounded handshake framing independent of carrier reads;
- validated configuration types for public keys and client authorization material;
- role-bound negotiated parameters;
- send and receive traffic state;
- protected-record seal/open APIs;
- authenticated public header handling;
- packet-number or sequence management;
- bounded anti-replay window;
- key epoch state;
- graceful close state;
- secret-redacting debug behavior;
- metrics without secret contents.

Integrate with the M2 in-memory simulation only.

## Required tests

- successful deterministic test handshake;
- wrong server key;
- wrong client authorization;
- modified initiation and response;
- replayed initiation;
- role confusion attempt;
- version/feature downgrade attempt;
- independent directional keys;
- seal/open fixtures;
- modified public header, ciphertext, and tag;
- duplicate, too-old, and reordered record handling;
- nonce uniqueness over a large deterministic sequence;
- packet-number rollover boundary;
- key update and deletion of old state;
- abrupt close and graceful close;
- secret types do not reveal values through `Debug` or `Display`;
- arbitrary protected-record bytes do not panic or allocate without bounds.

## Constraints

- No external network.
- No live TCP/UDP listener.
- No TUN.
- No route changes.
- No HTTP/3.
- No decoy application.
- No scheduler.
- No fixed production key material in source or fixtures reachable from release code.

## Acceptance criteria

A synthetic packet must complete this deterministic path:

```text
packet -> fragments -> cells -> protected records -> simulated link
-> authenticated decryption -> cells -> reassembly -> identical packet
```

Negative tests must prove modified or replayed protected data is not delivered. All required commands pass and only M3 is implemented.
