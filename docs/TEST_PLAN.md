# MOSAIC Test Plan

## 1. Test layers

### 1.1 Unit tests

Each crate tests local invariants without network privileges.

### 1.2 Property tests

Required for codec, fragmentation, reassembly, replay windows, and scheduler conservation properties.

### 1.3 Fuzz tests

At minimum:

- cell decoder;
- protected-record public framing decoder;
- reassembly sequence input;
- cover-profile parser;
- handshake-message framing independent of cryptographic validity.

### 1.4 Integration tests

- complete synthetic packet round trip in memory;
- complete encrypted round trip over localhost;
- reconnect and shutdown;
- loss, duplication, reorder, truncation, and backpressure simulation;
- Linux TUN test inside an isolated namespace;
- HTTP/3 client/server test with a locally trusted test certificate.

### 1.5 Evaluation tests

Measure functionality and classification features without asserting impossible guarantees.

## 2. Codec test matrix

Required cases:

- smallest valid cell;
- maximum valid cell;
- every cell type;
- one byte at a time input;
- all split positions;
- multiple cells in one chunk;
- truncated header and body;
- declared length larger than limit;
- arithmetic overflow attempt;
- invalid type and flags;
- non-canonical integer representation if varints are selected;
- duplicate/unknown mandatory extension;
- trailing bytes;
- random arbitrary bytes;
- decoder reset after a complete error according to documented policy.

Properties:

```text
decode(encode(x)) == x
encoded_len(x) == actual encoded byte count
successful decode consumes exactly one canonical value
invalid input never panics
buffered bytes never exceed configured maximum
```

## 3. Fragmentation and reassembly matrix

- packet smaller than one fragment;
- exact boundary;
- one byte above boundary;
- maximum packet;
- all fragment orders for small packet;
- exact duplicates;
- conflicting duplicates;
- partial and full overlap;
- gap that later closes;
- inconsistent total length;
- offset overflow;
- fragment beyond end;
- zero-length fragment;
- timeout;
- limit on incomplete packets;
- global memory limit;
- same packet ID after completed delivery;
- two directions and epochs remain isolated;
- completed output equals input byte-for-byte.

## 4. Session tests

- successful client/server handshake;
- incorrect server key;
- incorrect client authorization;
- modified handshake message;
- replayed initiation;
- version downgrade attempt;
- wrong role binding;
- independent directional keys;
- seal/open round trip;
- modified ciphertext/header/tag;
- duplicate record;
- too-old record;
- reordered records inside replay window;
- packet-number overflow boundary;
- key update and old-key deletion;
- close and secret destruction behavior.

Use deterministic test vectors where the selected library permits them, but production randomness must remain non-deterministic.

## 5. Carrier tests

Carrier contract tests must run against memory and localhost implementations:

- arbitrary write/read splitting;
- coalesced writes;
- partial send/backpressure;
- orderly close;
- abrupt close;
- maximum event size;
- read/write cancellation;
- no assumption that one send equals one receive.

## 6. TUN tests

Default tests use `MemoryPacketDevice`.

Privileged Linux tests are opt-in and must:

- run in an isolated network namespace when possible;
- create a uniquely named temporary interface;
- set a controlled MTU and addresses;
- never alter the host default route;
- send known packets through the interface;
- verify cleanup after success and injected failure;
- print recovery commands on cleanup failure.

## 7. HTTP/3 tests

- valid certificate and hostname;
- invalid certificate rejected;
- decoy request works without tunnel credentials;
- malformed request follows ordinary bounded decoy handling;
- valid authenticated initiation creates a session;
- invalid initiation does not return a tunnel-specific code or body;
- request bodies and concurrent streams are bounded;
- reconnect and graceful close;
- no third-party host used in automated tests.

## 8. Scheduler tests

Conservation:

```text
recovered protected bytes == submitted protected bytes
```

Bounds:

- queued bytes never exceed limit;
- padding never exceeds byte budget;
- delay never exceeds selected class deadline;
- strict profile cannot create infinite zero-progress transitions;
- shutdown drains or aborts within deadline.

Determinism:

- same seed and inputs produce the same event sequence in simulation;
- different seeds preserve all correctness invariants.

Statistical checks should use tolerance ranges, not exact random sequences unless the seed is fixed.

## 9. Evaluation plan

Datasets:

- ordinary decoy application traffic;
- decoy application plus MOSAIC in fast mode;
- balanced mode;
- strict mode;
- localhost/plain baseline where useful;
- optional comparison with another tunnel only when legally and operationally appropriate.

Record:

- code revision;
- configuration and profile version;
- random seed;
- capture environment;
- number and duration of sessions;
- total useful bytes;
- carrier bytes;
- latency and loss settings.

Metrics:

- packet/event size distributions;
- direction changes;
- burst lengths;
- inter-arrival distributions;
- connection duration;
- useful throughput;
- padding overhead;
- added latency;
- classifier ROC-AUC;
- TPR at selected low FPR points;
- confidence intervals when practical.

The report must describe limitations and must not translate one classifier result into a universal invisibility claim.

## 10. Required routine commands

```sh
cargo fmt --all --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo test --workspace --all-features
```

Additional milestone-specific commands must be documented in the task and README.
