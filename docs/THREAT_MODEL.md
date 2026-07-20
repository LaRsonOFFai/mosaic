# MOSAIC Threat Model

Status: Draft 0.1

## 1. Protected assets

- confidentiality and integrity of inner IP packets in transit;
- confidentiality of destinations and transport headers from a passive observer between client and server;
- authentication material and session keys;
- integrity of protocol negotiation;
- availability of bounded local resources;
- reduced direct correspondence between inner packet boundaries and carrier events.

## 2. Adversary capabilities

The evaluated network adversary may:

- observe source/destination addresses, packet sizes, directions, timing, connection duration, and TLS/QUIC-visible metadata;
- record traffic for later analysis;
- drop, delay, reorder, duplicate, truncate, or corrupt traffic;
- initiate connections to the public server;
- submit malformed application and handshake input;
- replay previously observed initiation or protected messages;
- train statistical classifiers on labeled captures;
- block an IP address, domain, transport, or broad traffic class.

The adversary does not initially possess valid client authorization material, server private keys, or control of either endpoint.

## 3. In-scope attacks

- passive metadata classification;
- active unauthenticated probing of the public service;
- malformed-input parser attacks;
- replay and duplicate delivery;
- resource exhaustion through bounded numbers of connections and messages;
- fragment overlap and reassembly confusion;
- downgrade attempts;
- nonce reuse caused by implementation errors;
- route recursion and accidental network lockout during local integration.

## 4. Out-of-scope attacks

The initial prototype does not claim protection against:

- a global observer correlating both ends;
- compromise of the client, server, build pipeline, or dependencies;
- traffic analysis based on long-term user behavior;
- blocking the known server IP or domain;
- allowlist-only networks;
- coercion or legal restrictions;
- endpoint malware;
- denial of service with unbounded attacker bandwidth;
- advanced TLS implementation fingerprinting beyond measured results;
- malicious operators of infrastructure outside the user's control.

## 5. Security objectives

### SO-001

A passive observer cannot read or modify inner packets without detection, assuming the selected standard cryptography remains secure and endpoints are not compromised.

### SO-002

Malformed bytes do not cause panics, unchecked allocation, integer overflow, or persistent state corruption.

### SO-003

A protected record, fragment, or complete packet is not delivered more than once because of replay or duplication.

### SO-004

Unauthenticated probing does not receive tunnel-specific status codes, banners, or detailed protocol errors from the HTTP/3 decoy endpoint.

### SO-005

Inner packet boundaries do not directly force carrier-event boundaries when the scheduler is enabled.

### SO-006

Any claim about classification resistance is accompanied by dataset description, baseline, false-positive rate, true-positive rate, and limitations.

### SO-007

TUN and route integration cannot modify the default host network during ordinary unit tests and has documented rollback.

## 6. Abuse-prevention and deployment boundaries

The project is restricted to:

- localhost;
- isolated lab networks;
- servers, domains, certificates, and accounts controlled by the operator.

The project excludes:

- domain fronting;
- misuse of third-party CDNs;
- impersonation of unrelated services;
- credential theft or bypass;
- unauthorized network access;
- hidden installation or persistence;
- automated scanning of public networks.

## 7. Assumptions requiring validation

The following are hypotheses, not guarantees:

- an ordinary decoy application may increase the collateral cost of simplistic blocking;
- scheduler profiles may reduce selected classifier accuracy;
- coalescing and fragmentation may weaken direct inner/outer burst correspondence;
- a standards-compliant HTTP/3 implementation may blend better than a bespoke transport.

Each hypothesis must be evaluated experimentally. Negative results are valid outcomes.
