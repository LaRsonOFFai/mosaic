# Task M5 — Linux TUN integration

## Operating mode

Plan first. Treat TUN and routes as a privilege boundary. Do not execute privileged commands automatically during implementation.

## Goal

Connect the existing packet-device abstraction to a real Linux TUN interface while preserving a fully unprivileged default test suite.

## Required design ADR

Create `docs/adr/0006-linux-tun-and-routing.md` covering:

- selected TUN library or direct `/dev/net/tun` approach;
- unsafe/system-call exposure;
- interface ownership and cleanup;
- MTU derivation;
- blocking/nonblocking integration;
- privilege requirements;
- route plan and rollback model;
- server-endpoint route protection;
- Linux network-namespace testing;
- crash recovery.

## Required implementation

1. Add `mosaic-tun-linux` as a Linux-only crate.
2. Implement `PacketDevice` for a TUN interface.
3. Validate that reads contain a complete IPv4 or IPv6 packet within limits.
4. Support configurable interface name with safe validation or an automatically generated temporary name.
5. Support configurable MTU with validated range.
6. Add CLI options to use TUN, but require an explicit flag.
7. Keep route management in a separate module from TUN I/O and protocol logic.
8. Implement a route-plan dry-run that prints intended changes and rollback without applying them.
9. If route application is implemented now, require two explicit flags, record rollback before each mutation, and protect the server endpoint first.
10. Add recovery documentation for abnormal shutdown.

## Required tests

Default tests:

- compile on Linux without privileges;
- mocked or in-memory device tests;
- MTU and interface-name validation;
- cleanup state machine;
- route-plan ordering and rollback generation;
- no command execution in dry-run.

Opt-in privileged integration test:

- runs only when a specific environment variable/feature is enabled;
- uses an isolated network namespace;
- creates a temporary TUN;
- sends known IPv4 and IPv6 packets;
- verifies end-to-end transfer through the local M4 server;
- never alters the host default route;
- cleans up on success and injected failure.

## Safety constraints

- Never run `sudo` or privileged commands without the user's explicit action.
- Automated tests must not alter host routes, DNS, firewall, or sysctls.
- Do not add persistence or privileged daemon installation.
- Do not silently route all traffic.
- No packet logging.
- No HTTP/3 yet.

## Acceptance criteria

On Linux, an explicitly invoked isolated test can move packets between a TUN interface and the M4 loopback tunnel. The ordinary workspace test suite remains unprivileged. Cleanup and recovery are documented. Only M5 is implemented.
