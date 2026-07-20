# Glossary

- **Inner packet** — complete IPv4 or IPv6 packet read from a packet device.
- **Packet device** — source/sink abstraction for inner packets; Linux TUN is one implementation.
- **Cell** — canonical plaintext protocol unit inside an encrypted batch.
- **Data fragment** — cell carrying a byte range of one inner packet.
- **Batch** — one or more cells sealed together as one AEAD plaintext.
- **Protected record** — authenticated encrypted batch plus required public framing.
- **Carrier** — transport adapter carrying opaque protected bytes or events.
- **Carrier event** — scheduler output submitted to a carrier.
- **Scheduler** — bounded component mapping protected bytes to sized and timed carrier events.
- **Cover profile** — versioned state machine describing permitted event behavior.
- **Decoy application** — harmless normal application served by the operator-controlled HTTP/3 endpoint.
- **Epoch** — bounded generation of traffic keys.
- **Packet number** — per-direction value used for replay ordering and/or nonce derivation.
- **Replay window** — bounded state rejecting duplicate and too-old protected records.
- **Reassembly entry** — bounded state collecting fragments for one inner packet.
- **Mandatory feature** — feature whose absence or unknown meaning requires rejection.
- **Optional feature** — safely skippable negotiated extension with explicit length rules.
- **Classification resistance** — measured difficulty of separating test traffic from a chosen baseline; not a guarantee of invisibility.
