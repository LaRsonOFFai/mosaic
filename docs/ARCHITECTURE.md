# MOSAIC Architecture

Status: Draft 0.1

## 1. Layer model

```text
┌─────────────────────────────────────────────────────────┐
│ OS applications and kernel networking                   │
└──────────────────────────┬──────────────────────────────┘
                           │ raw IPv4/IPv6 packets
┌──────────────────────────▼──────────────────────────────┐
│ PacketDevice                                               │
│ MemoryPacketDevice / LinuxTunDevice                        │
└──────────────────────────┬──────────────────────────────┘
                           │ validated inner packets
┌──────────────────────────▼──────────────────────────────┐
│ Packet pipeline                                            │
│ queueing / flow metadata / fragmentation / reassembly      │
└──────────────────────────┬──────────────────────────────┘
                           │ plaintext cells
┌──────────────────────────▼──────────────────────────────┐
│ Session                                                    │
│ handshake / key epochs / AEAD / replay window              │
└──────────────────────────┬──────────────────────────────┘
                           │ encrypted records
┌──────────────────────────▼──────────────────────────────┐
│ Scheduler                                                  │
│ fragmentation / coalescing / delay / padding / profiles    │
└──────────────────────────┬──────────────────────────────┘
                           │ carrier events
┌──────────────────────────▼──────────────────────────────┐
│ Carrier                                                    │
│ memory / localhost stream / HTTP/3                         │
└──────────────────────────┬──────────────────────────────┘
                           │ network
┌──────────────────────────▼──────────────────────────────┐
│ Peer                                                       │
└─────────────────────────────────────────────────────────┘
```

The scheduler is bypassable. Fast mode may map encrypted records to carrier events with minimal transformation. Strict mode applies a cover profile.

## 2. Proposed Cargo workspace

```text
crates/
├── mosaic-core/              shared limits, IDs, errors, validated packet type
├── mosaic-codec/             canonical cell and record encoding
├── mosaic-pipeline/          queues, fragmentation, reassembly
├── mosaic-session/           handshake, keys, AEAD, replay, epochs
├── mosaic-packet-device/     PacketDevice trait and memory implementation
├── mosaic-tun-linux/         Linux-only TUN adapter
├── mosaic-carrier/           carrier traits and memory/loopback implementations
├── mosaic-carrier-h3/        HTTP/3 adapter and decoy integration boundary
├── mosaic-scheduler/         cover profiles and event scheduling
├── mosaic-client/            client orchestration
├── mosaic-server/            server orchestration
├── mosaic-cli/               command-line interface
└── mosaic-sim/               deterministic simulations and experiments

tools/
└── evaluation/               offline feature extraction and reports
```

Crates may be introduced only when their milestone begins. Empty speculative crates are not required in M0.

## 3. Dependency direction

Allowed high-level dependency direction:

```text
mosaic-core
  ↑
  ├── mosaic-codec
  ├── mosaic-packet-device
  ├── mosaic-carrier
  ├── mosaic-pipeline ────────┐
  ├── mosaic-session          │
  └── mosaic-scheduler        │
                              │
packet device + pipeline + session + scheduler + carrier
                              ↓
                    client / server orchestration
                              ↓
                            CLI
```

Forbidden dependencies:

- `mosaic-core` depending on Tokio, TUN, HTTP/3, CLI, or platform APIs;
- codec depending on session cryptography;
- session depending on HTTP/3;
- TUN code inside client orchestration;
- route manipulation inside protocol crates;
- scheduler parsing raw user payloads.

## 4. Core conceptual interfaces

Exact Rust signatures require ADRs and may change during M0 planning. The semantic boundaries are mandatory.

### Packet device

```rust
trait PacketDevice {
    type Error;

    fn mtu(&self) -> usize;
    fn try_receive(&mut self, out: &mut [u8]) -> Result<Option<usize>, Self::Error>;
    fn try_send(&mut self, packet: &[u8]) -> Result<bool, Self::Error>;
}
```

An async adapter may wrap this interface. The protocol core must remain testable without an async runtime.

### Carrier

```rust
struct CarrierCapabilities {
    reliable_ordered: bool,
    datagrams: bool,
    max_event_size: usize,
}

trait Carrier {
    type Error;

    fn capabilities(&self) -> CarrierCapabilities;
    fn try_send(&mut self, event: &[u8]) -> Result<bool, Self::Error>;
    fn try_receive(&mut self, out: &mut [u8]) -> Result<Option<usize>, Self::Error>;
}
```

The actual implementation may use async traits at the orchestration edge. The protocol must not rely on write/read boundary preservation.

### Clock

```rust
trait Clock {
    type Instant: Copy + Ord;
    fn now(&self) -> Self::Instant;
}
```

A deterministic simulated clock is mandatory for timeout and scheduler tests.

### Random source

Security-sensitive randomness and deterministic test randomness must be different typed interfaces or otherwise impossible to confuse in release builds.

## 5. Data path

Client outbound path:

```text
PacketDevice.receive
  -> validate packet
  -> classify only minimal local scheduling metadata
  -> assign packet_id
  -> fragment
  -> encode plaintext cells
  -> session seal into encrypted records
  -> scheduler map records to carrier events
  -> carrier send
```

Server inbound path:

```text
carrier receive
  -> scheduler recover encrypted record stream
  -> session authenticate and decrypt
  -> decode cells
  -> reassembly
  -> validate completed packet
  -> PacketDevice.send or controlled sink
```

The reverse direction uses independent keys, packet numbers, queues, and metrics.

## 6. Control plane

Control messages are carried inside authenticated encryption after the handshake. Initial controls include:

- negotiated limits;
- ping/pong;
- graceful close;
- key update;
- optional feature negotiation;
- non-secret error category.

Sensitive diagnostics are local-only and are not sent to an unauthenticated peer.

## 7. Privilege boundary

Only the Linux TUN and optional route manager require elevated network privileges. They must live outside protocol crates.

Preferred deployment split:

```text
unprivileged mosaic process
        │ narrow local IPC or inherited file descriptor
privileged setup helper
        │ creates/configures TUN and exits
kernel TUN
```

The first prototype may run one process with explicit privileges, but the architecture must not make that permanent.

## 8. Configuration boundary

Configuration is parsed into an untrusted raw form, validated into typed configuration, and only then passed to components.

```text
file/CLI/env -> RawConfig -> validation -> ClientConfig / ServerConfig
```

Secrets use dedicated secret types whose `Debug` and `Display` redact contents.

## 9. Scheduler boundary

The scheduler receives:

- opaque encrypted bytes;
- enqueue timestamp;
- delivery class such as control/realtime/bulk;
- maximum permitted delay;
- carrier capabilities.

It must not receive raw inner packet payloads. It may receive bounded metadata derived locally, but every metadata field must be justified because metadata can become a side channel.

## 10. Decoy boundary

The HTTP/3 service routes normal requests to a harmless decoy application. A tunnel session is created only after a valid authenticated initiation carried inside an ordinary application request.

Invalid input follows normal application behavior. It must not return proxy-specific status codes, special timing shortcuts, or distinctive protocol errors.

## 11. State machines

Explicit state machines are required for:

- handshake and established session;
- key epochs;
- connection lifecycle;
- reassembly entry lifecycle;
- scheduler cover profile;
- route application and rollback.

Illegal transitions return typed errors and are covered by tests.
