# ANP × libp2p × IPFS Interoperability PoC

**Interoperability between Agent Network Protocol (ANP), libp2p, and IPFS/IPNS for decentralized, verifiable, peer-to-peer agents.**

> **Status:** Proof of Concept / Research  
> **Scope:** Agent identity, DID interoperability, IPFS/IPNS DID publication, DID ↔ libp2p Peer ID binding, and ANP messaging over libp2p.

---

## Overview

This repository explores interoperability between **ANP**, **libp2p**, and **IPFS/IPNS** to enable agents to communicate using a common messaging model while remaining independent of a particular identity method or network transport.

The PoC is based on a simple architectural principle:

> **A DID represents the stable identity of an agent, a libp2p Peer ID represents a network node, and an IPFS CID represents a particular version of content. These identifiers should be cryptographically and verifiably linked, but should not be collapsed into the same identity.**

The proposed architecture combines:

- **W3C DID-based identity** for stable, verifiable agent identities;
- **IPFS/IPNS** for decentralized DID Document publication and resolution;
- **libp2p** for secure peer-to-peer connectivity;
- **ANP** for agent messaging and authentication;
- **Cryptographic bindings** between agent identity, authorized network peers, and content.

The objective is not to replace existing ANP identity or messaging semantics, nor to make libp2p Peer IDs equivalent to DIDs. Instead, the PoC investigates how these systems can be composed while preserving their distinct roles.

---

## Why This PoC?

Agent networks increasingly need to operate across different environments:

- web applications;
- decentralized networks;
- edge devices;
- cloud infrastructure;
- local networks;
- mobile applications;
- autonomous agents;
- multi-device deployments.

A web-oriented agent identity should not necessarily require a centralized web endpoint, while a peer-to-peer node should not necessarily become the permanent identity of the agent operating it.

This PoC therefore explores a layered model:

```text
┌─────────────────────────────────────────────┐
│              Agent Identity                 │
│                                             │
│                 W3C DID                     │
└──────────────────────┬──────────────────────┘
                       │
                       │ authorizes
                       ▼
┌─────────────────────────────────────────────┐
│         DID ↔ Peer Authorization             │
│                                             │
│     DID Verification Method / Peer Key      │
└──────────────────────┬──────────────────────┘
                       │
                       │ binds
                       ▼
┌─────────────────────────────────────────────┐
│             Network Identity               │
│                                             │
│               libp2p Peer ID                 │
└──────────────────────┬──────────────────────┘
                       │
                       │ reachable through
                       ▼
┌─────────────────────────────────────────────┐
│               Connectivity                  │
│                                             │
│      Multiaddr / Secure libp2p Session      │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────┐
│               Agent Messaging               │
│                                             │
│                  ANP                         │
└─────────────────────────────────────────────┘
```

The identity, network, and content layers remain distinct.

---

# Architecture

The PoC consists of three primary layers.

## 1. DID Identity Layer

The DID represents the **long-term application identity of the agent**.

A DID Document contains the cryptographic material and service information necessary to authenticate the agent and discover its available network endpoints.

ANP 1.2 already separates authentication from a particular DID method. The current ANP model supports methods such as `did:wba` and `did:web`, while allowing additional DID methods to be integrated.

This PoC investigates an IPFS/IPNS-based DID environment as another identity option.

---

## 2. DID ↔ libp2p Binding Layer

A DID identifies the agent, while a libp2p Peer ID identifies a concrete network node.

The two identities should be **cryptographically and verifiably linked**, but should not be treated as the same identity.

The DID Document can contain a verification method representing the public key associated with an authorized libp2p peer.

```text
Agent DID
    │
    │ authorizes
    ▼
Peer Verification Method
    │
    │ binds to
    ▼
libp2p Peer ID
    │
    │ reachable through
    ▼
Multiaddr
```

This makes it possible for an agent to change its network infrastructure without changing its long-term identity.

For example:

```text
Agent DID
    │
    ├── Peer A / Peer ID A
    │
    ├── Peer B / Peer ID B
    │
    └── Peer C / Peer ID C
```

This also allows a future deployment to support multiple cloud, edge, mobile, or device nodes under a single agent identity.

---

## 3. ANP over libp2p

ANP is treated as the **agent messaging layer**, while libp2p provides the underlying peer-to-peer transport.

The existing ANP semantics remain unchanged.

Conceptually:

```text
                 ANP
                  │
        ┌─────────┼─────────┐
        │         │         │
      HTTPS      WSS      libp2p
```

The proposed **ANP over libp2p Transport Binding** would define the transport-specific behavior necessary to carry ANP messages over a libp2p stream.

This includes:

- libp2p protocol ID;
- stream establishment;
- message framing;
- request/response correlation;
- notifications;
- message-size limits;
- timeouts;
- retries;
- backpressure;
- connection authentication;
- DID ↔ Peer ID verification.

The underlying ANP messaging model remains the same.

---

# Identity Model

One of the central design decisions of this PoC is to keep three identifiers conceptually separate.

| Identifier | Represents | Example |
|---|---|---|
| **DID** | Stable agent/application identity | `did:example:agent-a` |
| **Peer ID** | Concrete libp2p network node | `12D3KooW...` |
| **CID** | Particular version of content | `bafy...` |

A DID Document may be published through IPFS/IPNS, with IPNS providing the mutable reference and IPFS providing content-addressed versions.

```text
Stable DID
    │
    ▼
  IPNS
    │
    ▼
 IPFS CID
    │
    ▼
DID Document
```

The CID therefore represents a particular version of the DID Document rather than becoming the permanent agent identity.

This allows the DID Document, keys, and network endpoints to evolve while the Agent DID remains stable.

---

# IPFS/IPNS-Based DID

The first identity-related question for the PoC is whether an existing IPFS/IPNS-oriented DID method, such as `did:ipid`, can satisfy the requirements of an agent identity layer.

If an existing DID method is insufficient for the required agent functionality, a new IPFS/IPNS-based DID method or profile could be investigated collaboratively.

The relevant requirements include:

- DID identifier syntax;
- DID Document creation;
- DID Document publication;
- DID Resolution;
- DID Document updates;
- key rotation;
- deactivation;
- caching;
- rollback protection;
- cryptographic binding between the DID and IPNS;
- DID Document versioning on IPFS.

The PoC should first evaluate existing approaches before defining new protocol machinery.

---

# Key Model

The recommended long-term model separates the cryptographic keys used for different purposes.

```text
DID / IPNS Control Key
        │
        ├── Controls DID state and updates
        │
        ▼
ANP Authentication Key
        │
        ├── Authenticates ANP requests
        └── Produces / verifies Origin Proof
        │
        ▼
libp2p Peer Key
        │
        └── Derives Peer ID and authenticates
            the libp2p connection
```

The DID Document can represent the Peer Public Key as a `verificationMethod`, without making it part of the DID's `authentication` relationship by default.

A conceptual DID Document looks like:

```json
{
  "id": "did:example:agent-a",

  "verificationMethod": [
    {
      "id": "did:example:agent-a#auth-key-1",
      "type": "Multikey",
      "controller": "did:example:agent-a",
      "publicKeyMultibase": "zAUTH..."
    },
    {
      "id": "did:example:agent-a#peer-key-1",
      "type": "Multikey",
      "controller": "did:example:agent-a",
      "publicKeyMultibase": "zPEER..."
    }
  ],

  "authentication": [
    "did:example:agent-a#auth-key-1"
  ]
}
```

In this model:

- `#auth-key-1` proves the ANP business identity;
- `#peer-key-1` proves the network peer identity;
- the keys can rotate independently.

For the smallest PoC, the same key pair may be used for both DID authentication and libp2p peer identity to demonstrate the cryptographic relationship quickly. However, the preferred long-term architecture keeps these roles separate.

---

# `ANPLibp2pService`

The PoC proposes a lightweight service entry called:

```text
ANPLibp2pService
```

Its purpose is to provide network discovery information while explicitly referencing the Peer Verification Method.

Example:

```json
{
  "service": [
    {
      "id": "did:example:agent-a#anp-libp2p",
      "type": "ANPLibp2pService",
      "serviceEndpoint": {
        "peerId": "12D3KooW...",
        "peerKey": "did:example:agent-a#peer-key-1",
        "multiaddrs": [
          "/dns4/agent.example/tcp/443/wss/p2p/12D3KooW..."
        ],
        "protocols": [
          "/anp/1.0"
        ]
      }
    }
  ]
}
```

The important relationship is:

```text
Agent DID
    │
    │ authorizes
    ▼
Peer Verification Method
    │
    │ derives / binds
    ▼
Peer ID
    │
    │ reachable through
    ▼
Multiaddr
    │
    ▼
libp2p Connection
    │
    ▼
ANP Messaging
```

A caller can therefore:

1. resolve the target DID;
2. validate the DID Document;
3. discover `ANPLibp2pService`;
4. obtain the Peer Key, Peer ID, and multiaddr;
5. verify the relationship between the Peer Public Key and Peer ID;
6. establish a secure libp2p connection;
7. verify that the connected peer is authorized by the DID;
8. exchange ANP messages.

This trust chain is a core part of the PoC.

---

# Two Authentication Layers

The architecture deliberately contains **two authentication layers**.

## Layer 1 — libp2p Connection Authentication

This answers:

> **Which peer am I currently connected to?**

The answer is established using libp2p's Peer ID and secure connection mechanisms.

```text
Peer Key
   │
   ▼
Peer ID
   │
   ▼
Secure libp2p Connection
```

---

## Layer 2 — ANP Business Identity Authentication

This answers:

> **Which Agent DID actually originated this ANP message?**

This continues to use ANP's existing DID authentication and Origin Proof mechanisms.

```text
Agent DID
   │
   ▼
Authentication Key
   │
   ▼
ANP Origin Proof
```

The two layers therefore answer different questions:

```text
Peer Key / Peer ID
        │
        ▼
Network Connection Identity


Agent DID / Authentication Key
        │
        ▼
ANP Business Identity
```

They complement each other rather than replacing each other.

This separation is particularly important when messages may later traverse relays, gateways, proxies, or other transport paths.

---

# PoC Scope

The initial PoC is divided into two complementary phases.

## PoC 1 — IPFS/IPNS DID × ANP

The first phase validates whether an IPFS/IPNS-based DID can participate in the ANP 1.2 identity and messaging model.

Conceptually:

```text
┌──────────────────┐
│ Agent A          │
│ did:web/wba      │
└────────┬─────────┘
         │
         │ ANP
         │
         ▼
┌──────────────────┐
│ Agent B          │
│ IPFS/IPNS DID    │
└──────────────────┘
```

Existing HTTP/HTTPS transport can continue to be used during this phase.

### PoC 1 Components

The implementation would include:

- IPFS/IPNS DID Resolver;
- ANP DID Method Adapter;
- DID Document validation;
- authentication-key verification;
- bidirectional ANP Direct Messaging.

### PoC 1 Goal

Demonstrate:

> **Agents using different DID methods can share the same ANP authentication and messaging model.**

This phase isolates identity interoperability before introducing a new network transport.

---

# PoC 2 — ANP over libp2p

The second phase introduces libp2p as the peer-to-peer transport.

```text
Agent A
   │
   │ Target DID
   ▼
Resolve DID Document
   │
   │ Discover ANPLibp2pService
   ▼
Peer Verification Method
   │
   │ Verify Peer ID
   ▼
libp2p Secure Connection
   │
   ▼
ANP direct.send
   │
   │ Verify Origin Proof
   ▼
Agent B
```

The end-to-end demonstration should show an agent using a web-oriented DID such as `did:web` or `did:wba` communicating with an agent whose identity is published through an IPFS/IPNS-based DID.

The initiating agent should:

1. resolve the target DID;
2. validate its DID Document;
3. discover `ANPLibp2pService`;
4. retrieve the authorized Peer Key;
5. retrieve the Peer ID and network addresses;
6. verify the Peer ID against the declared Peer Public Key;
7. establish a secure libp2p connection;
8. establish the ANP stream;
9. send a standard ANP `direct.send` message;
10. verify the recipient's ANP identity and Origin Proof.

The intended result is an end-to-end interoperability demonstration across **DID methods, identity systems, and transport layers**.

---

# End-to-End Flow

A complete PoC interaction can be represented as:

```text
┌──────────────┐
│   Agent A    │
│              │
│ did:web/wba  │
└──────┬───────┘
       │
       │ 1. Resolve Agent B DID
       ▼
┌──────────────────────┐
│   Agent B DID        │
│                      │
│ IPFS/IPNS DID        │
│ DID Document         │
└──────────┬───────────┘
           │
           │ 2. Discover
           │    ANPLibp2pService
           ▼
┌──────────────────────┐
│ Peer Verification    │
│ Method               │
└──────────┬───────────┘
           │
           │ 3. Verify Peer ID
           ▼
┌──────────────────────┐
│ libp2p Peer           │
│                      │
│ Peer ID + Multiaddr  │
└──────────┬───────────┘
           │
           │ 4. Secure connection
           ▼
┌──────────────────────┐
│ ANP over libp2p      │
│                      │
│ /anp/1.0             │
└──────────┬───────────┘
           │
           │ 5. direct.send
           ▼
┌──────────────────────┐
│ Agent B              │
│                      │
│ Verify Origin Proof  │
└──────────────────────┘
```

---

# Interoperability Test Matrix

The PoC should validate interoperability at multiple levels.

## Identity Interoperability

| Test | Expected Result |
|---|---|
| `did:web` ↔ IPFS/IPNS DID | Both can participate in ANP |
| `did:wba` ↔ IPFS/IPNS DID | Both can participate in ANP |
| DID Document resolution | Valid documents resolve successfully |
| Authentication verification | Correct authentication keys are accepted |
| Invalid authentication key | Message is rejected |

## Peer Binding

| Test | Expected Result |
|---|---|
| Valid Peer ID + Peer Key | Connection accepted |
| Peer ID does not match declared key | Connection rejected |
| Unauthorized Peer ID | Connection rejected |
| Authorized peer rotation | New peer accepted after DID update |
| Revoked peer | Connection rejected |

## Messaging

| Test | Expected Result |
|---|---|
| ANP `direct.send` over libp2p | Message delivered |
| Valid Origin Proof | Message accepted |
| Invalid Origin Proof | Message rejected |
| Replay attempt | Message rejected |
| Request/response correlation | Correct operation associated with response |
| Notification | Delivered without request/response dependency |

---

# Key PoC Tests

## 1. Multi-DID Interoperability

The first test validates that different DID methods can participate in the same ANP messaging system.

```text
did:web
    │
    │
    ▼
   ANP
    ▲
    │
    │
IPFS/IPNS DID
```

Both sides should be able to exchange ANP messages using the same protocol semantics.

---

## 2. Peer Rotation

The PoC should demonstrate that a Peer ID can change without changing the Agent DID.

```text
Agent DID
    │
    ├── Peer Key 1
    │       │
    │       └── Peer ID 1
    │
    │       [rotation]
    │
    └── Peer Key 2
            │
            └── Peer ID 2
```

The DID remains stable.

The DID Document is updated to revoke the old peer and authorize the new peer.

---

## 3. Multi-Peer Agent

A single Agent DID may eventually authorize multiple network nodes:

```text
                 Agent DID
                     │
          ┌──────────┼──────────┐
          │          │          │
          ▼          ▼          ▼
       Peer A      Peer B      Peer C
          │          │          │
        Cloud       Edge      Device
```

This enables a stable agent identity to operate across multiple network locations.

---

## 4. Security Failure Cases

The PoC should explicitly test negative cases.

The implementation should reject:

- unauthorized Peer IDs;
- Peer IDs that do not match the declared Peer Public Key;
- tampered DID Documents;
- invalid IPNS records;
- revoked peers;
- revoked authentication keys;
- invalid ANP Origin Proofs;
- replay attacks.

Security failures should be treated as first-class interoperability tests rather than as optional edge cases.

---

# DID Document Update and Key Rotation

A major benefit of separating Agent identity from network identity is independent key and peer rotation.

For example:

```text
                         Agent DID
                            │
              ┌─────────────┴─────────────┐
              │                           │
        Authentication                 Peer
             Key                         Key
              │                           │
              ▼                           ▼
        ANP Identity                 Peer ID
                                          │
                                          │ rotation
                                          ▼
                                     New Peer ID
```

A peer may be replaced without changing the Agent DID.

Likewise, authentication keys can be rotated independently of network keys.

The PoC should therefore treat DID Document updates as an important part of the interoperability model rather than merely as a publication mechanism.

---

# IPFS/IPNS Content Model

The proposed content flow is:

```text
DID
 │
 ▼
IPNS Name
 │
 ▼
IPFS CID
 │
 ▼
DID Document Version
```

Each DID Document version is content-addressed by IPFS.

IPNS provides the mutable pointer to the current version.

This gives the architecture two useful properties:

### Stable identity

The DID remains stable across document changes.

### Immutable versions

Each IPFS CID identifies a specific version of the DID Document.

This separation can support:

- version history;
- key rotation;
- endpoint changes;
- peer authorization changes;
- deactivation;
- caching;
- rollback protection.

The exact update and resolution semantics remain part of the PoC research and should be validated against the chosen IPFS/IPNS-based DID method.

---

# ANP Transport Binding

The proposed libp2p transport binding should remain as small as possible.

The primary purpose is to map ANP's existing messaging semantics onto a libp2p stream.

The binding needs to specify:

```text
ANP Message
    │
    ▼
Message Framing
    │
    ▼
libp2p Stream
    │
    ▼
Secure libp2p Connection
```

The transport binding should define:

### Protocol Identification

A protocol identifier such as:

```text
/anp/1.0
```

is proposed in the service example.

### Stream Establishment

How an ANP stream is opened between two authorized peers.

### Message Framing

How individual ANP messages are delimited and reconstructed.

### Request / Response

How ANP `operation_id` and response correlation work over a persistent stream.

### Notifications

How notification-style ANP messages are transmitted without requiring a response.

### Limits

Maximum message sizes and other resource constraints.

### Reliability

Timeouts, retries, and failure behavior.

### Backpressure

How the transport handles a sender that is producing messages faster than the receiver can process them.

### Authentication

How the connected libp2p Peer ID is checked against the DID-authorized Peer Verification Method.

---

# Trust Model

The proposed trust model can be summarized as:

```text
                 ┌──────────────────┐
                 │    Agent DID     │
                 └────────┬─────────┘
                          │
                          │ authorizes
                          ▼
                 ┌──────────────────┐
                 │ Peer Verification│
                 │     Method       │
                 └────────┬─────────┘
                          │
                          │ binds
                          ▼
                 ┌──────────────────┐
                 │    Peer ID       │
                 └────────┬─────────┘
                          │
                          │ reachable
                          ▼
                 ┌──────────────────┐
                 │    Multiaddr     │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │ libp2p Connection │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │   ANP Message    │
                 └────────┬─────────┘
                          │
                          │ Origin Proof
                          ▼
                 ┌──────────────────┐
                 │  Agent Identity  │
                 └──────────────────┘
```

There are therefore two independent verification paths:

1. **Network path:** Peer Key → Peer ID → libp2p connection.
2. **Application path:** Agent DID → Authentication Key → ANP Origin Proof.

The PoC should verify both paths independently.

---

# Proposed Repository Structure

A possible repository structure for the PoC is:

```text
.
├── README.md
├── docs/
│   ├── architecture.md
│   ├── identity-model.md
│   ├── did-ipfs-ipns.md
│   ├── anp-libp2p-binding.md
│   ├── security-model.md
│   └── interoperability.md
│
├── examples/
│   ├── did-web/
│   ├── did-ipfs/
│   └── anp-libp2p/
│
├── tests/
│   ├── identity/
│   ├── peer-binding/
│   ├── transport/
│   ├── messaging/
│   └── security/
│
└── LICENSE
```

This structure is illustrative and can be adapted as implementation work begins.

---

# Implementation Phases

## Phase 1 — Identity Interoperability

- Evaluate existing IPFS/IPNS DID methods.
- Implement or integrate an IPFS/IPNS DID Resolver.
- Implement the ANP DID Method Adapter.
- Resolve DID Documents.
- Validate authentication keys.
- Exchange ANP messages between different DID methods.

### Success Criteria

```text
did:web / did:wba
       ↕
      ANP
       ↕
IPFS/IPNS DID
```

Successful bidirectional ANP messaging demonstrates the first interoperability milestone.

---

## Phase 2 — Peer Authorization

- Add a Peer Verification Method.
- Represent the libp2p Peer Public Key in the DID Document.
- Add `ANPLibp2pService`.
- Publish Peer ID and multiaddr information.
- Verify Peer ID against the declared public key.

### Success Criteria

```text
DID
 │
 ▼
Peer Verification Method
 │
 ▼
Peer ID
 │
 ▼
Multiaddr
```

The caller can determine whether a discovered libp2p peer is authorized by the target DID.

---

## Phase 3 — ANP over libp2p

- Define the ANP libp2p protocol binding.
- Establish a libp2p stream.
- Define message framing.
- Exchange ANP messages.
- Verify Origin Proof.
- Implement request/response correlation.
- Test notifications.
- Test failure and timeout behavior.

### Success Criteria

```text
DID Resolution
      ↓
Peer Discovery
      ↓
Peer Verification
      ↓
libp2p Connection
      ↓
ANP Stream
      ↓
ANP Message
      ↓
Origin Proof Verification
```

---

## Phase 4 — Rotation and Security

- Test Peer Key rotation.
- Test Peer ID rotation.
- Test authentication-key rotation.
- Test peer revocation.
- Test invalid DID Documents.
- Test invalid IPNS records.
- Test invalid Origin Proof.
- Test replay behavior.

### Success Criteria

Unauthorized or stale cryptographic identities are rejected while valid rotated identities continue to work.

---

# Division of Work

The PoC naturally separates into three areas.

## ANP Community

Potential areas of focus:

- DID Method Adapter;
- ANP DID Authentication;
- Origin Proof;
- ANP Messaging;
- `ANPLibp2pService`;
- ANP over libp2p binding;
- multi-DID interoperability tests.

## libp2p / IPFS Community

Potential areas of focus:

- Peer ID / Peer Key model;
- secure libp2p connections;
- Peer ID derivation and verification;
- IPFS publication;
- IPNS resolution and updates;
- peer-to-peer discovery;
- connectivity.

## Joint Work

The two communities can jointly define and validate:

- IPFS/IPNS DID method or adaptation profile;
- Peer Verification Method;
- `ANPLibp2pService`;
- DID ↔ Peer ID cryptographic binding;
- peer rotation;
- interoperability test suite.

This division follows the separation of concerns established in the PoC proposal.

---

# Security Considerations

Security is a fundamental part of this PoC.

The design should not assume that successful network connectivity implies successful agent authentication.

A peer may be reachable but unauthorized.

Likewise, an ANP message may contain a valid-looking sender DID but fail Origin Proof verification.

The PoC should therefore independently validate:

```text
DID Document
     │
     ├── Integrity
     │
     ├── Authentication Key
     │
     ├── Peer Verification Method
     │
     └── Service Information
             │
             ▼
          Peer ID
             │
             ▼
      Secure Connection
             │
             ▼
        ANP Message
             │
             ▼
        Origin Proof
```

Particular attention should be paid to:

- unauthorized peer injection;
- Peer ID / public-key mismatch;
- DID Document tampering;
- invalid or stale IPNS records;
- revoked peers;
- revoked authentication keys;
- replay attacks;
- stale DID Document versions;
- endpoint changes;
- key rotation.

The PoC is intended to expose these edge cases early, before the architecture is generalized.

---

# What This PoC Demonstrates

A successful implementation should demonstrate all of the following:

### 1. Multiple DID Methods

ANP can support agents using different DID methods.

### 2. IPFS/IPNS-Based Agent Identity

An agent identity can be represented using an IPFS/IPNS-based DID architecture.

### 3. DID ↔ Peer ID Binding

A DID can cryptographically authorize a specific libp2p Peer ID.

### 4. Transport Independence

The ANP identity and messaging model does not need to be tied to HTTPS or WSS.

### 5. ANP over libp2p

ANP messages can be exchanged over secure libp2p connections.

### 6. Independent Identity and Connectivity

An Agent DID can remain stable while its network Peer ID changes.

### 7. Multi-Peer Agents

A single agent identity can potentially authorize multiple network peers.

### 8. Cryptographic Verification

The application identity and network identity can each be independently verified.

These are the principal interoperability goals of the PoC.

---

# Future Work

Once the initial PoC is validated, the architecture can be extended toward more advanced agent-network scenarios.

Potential areas include:

- multi-peer agents;
- multi-device agents;
- ANP Direct E2EE over libp2p;
- ANP Group Messaging over libp2p;
- DHT-based agent discovery;
- Agent Description and metadata publication through IPFS;
- content-addressed agent metadata;
- Web ↔ libp2p gateways;
- a common Agent Identity Layer across different DID methods.

The longer-term architecture can be represented as:

```text
                         Agent DID
                            │
                 ┌──────────┴──────────┐
                 │                     │
          Web-based DID          P2P-based DID
                 │                     │
              HTTPS                IPFS / IPNS
                 │                     │
                 └──────────┬──────────┘
                            │
                    ANP Identity Layer
                            │
                 ┌──────────┴──────────┐
                 │                     │
              HTTPS / WSS            libp2p
                 │                     │
                 └──────────┬──────────┘
                            │
                       ANP Messaging
```

This would allow ANP to operate as a common protocol layer across heterogeneous identity systems and networking technologies rather than being coupled to one DID method or one transport.

---

# Design Principles

The PoC follows several principles.

## 1. Do not collapse identities

A DID, Peer ID, and CID have different semantic roles.

```text
DID     → Agent identity
Peer ID → Network node
CID     → Content version
```

They should be linked cryptographically where appropriate, but should not become interchangeable.

## 2. Preserve ANP semantics

Adding libp2p should not require redesigning ANP's existing identity and messaging semantics.

## 3. Keep transport separate from identity

An agent should be able to change its transport without necessarily changing its application identity.

## 4. Prefer existing standards and methods

The PoC should evaluate existing DID and IPFS/IPNS mechanisms before introducing new protocol specifications.

## 5. Make security failures explicit

Invalid identity bindings, revoked keys, invalid proofs, and replay attempts should be tested as part of interoperability.

## 6. Support identity continuity

An agent should retain a stable identity even when its network infrastructure changes.

---

# Non-Goals for the Initial PoC

The initial PoC does **not** attempt to solve every aspect of decentralized agent networking.

In particular, the first implementation does not need to fully solve:

- global DHT-based agent discovery;
- group messaging;
- multi-agent authorization frameworks;
- production-scale relay infrastructure;
- complete multi-device orchestration;
- a finalized new DID method specification;
- every possible libp2p transport;
- production deployment and operational tooling.

These can be addressed after the fundamental identity, peer-binding, and transport interoperability model has been validated.

---

# Open Questions

The PoC should help answer the following architectural questions:

1. Can an existing IPFS/IPNS DID method satisfy the requirements of ANP agent identity?
2. What is the most interoperable representation of a libp2p Peer Public Key inside a DID Document?
3. What should the normative DID ↔ Peer ID binding look like?
4. Should `ANPLibp2pService` become a formal service type?
5. What should the canonical ANP-over-libp2p protocol ID be?
6. What message framing should be used?
7. How should connection-level authentication and ANP-level authentication interact?
8. How should Peer ID rotation be represented?
9. How should multiple authorized peers be represented?
10. How should revocation and stale DID Document versions be handled?
11. What caching and rollback protections are required for IPFS/IPNS DID resolution?
12. Which portions of the architecture should become standardized after the PoC?

---

# Success Criteria

The PoC can be considered successful when the following end-to-end scenario works:

```text
┌─────────────────────────────────────────────────────────┐
│                      Agent A                            │
│                                                         │
│                    did:web / did:wba                    │
└─────────────────────────┬───────────────────────────────┘
                          │
                          │ 1. Resolve Agent B DID
                          ▼
┌─────────────────────────────────────────────────────────┐
│                      Agent B                            │
│                                                         │
│                   IPFS/IPNS-based DID                    │
│                                                         │
│                    DID Document                         │
│                         │                               │
│              ┌──────────┴──────────┐                    │
│              │                     │                    │
│       Authentication Key      Peer Key                  │
│                                      │                  │
│                                      ▼                  │
│                                   Peer ID                │
│                                      │                  │
│                                      ▼                  │
│                                   Multiaddr              │
└──────────────────────────────────────┬──────────────────┘
                                       │
                                       │ 2. Connect
                                       ▼
                              ┌────────────────┐
                              │    libp2p      │
                              │ Secure Session │
                              └───────┬────────┘
                                      │
                                      │ 3. ANP /anp/1.0
                                      ▼
                              ┌────────────────┐
                              │      ANP       │
                              │  direct.send   │
                              └───────┬────────┘
                                      │
                                      │ 4. Origin Proof
                                      ▼
                              ┌────────────────┐
                              │ Agent Identity │
                              │   Verified     │
                              └────────────────┘
```

The key demonstration is:

> **Agent A, using a web-based DID, resolves Agent B's IPFS/IPNS-based DID, discovers an authorized libp2p peer, establishes a secure libp2p connection, and sends a standard ANP message whose application-level identity is independently verified.**

This validates the central interoperability hypothesis of the PoC.

---

# Related Technologies

This PoC brings together three complementary protocol ecosystems:

| Technology | Role |
|---|---|
| **ANP** | Agent identity semantics, authentication, messaging |
| **W3C DID** | Stable, verifiable agent identity |
| **IPFS** | Content-addressed DID Document storage |
| **IPNS** | Mutable naming / reference to current DID Document |
| **libp2p** | Secure peer-to-peer networking |
| **Peer ID** | Network peer identity |
| **Multiaddr** | Network endpoint representation |

The architecture intentionally preserves these distinct responsibilities rather than introducing a single identifier or layer to perform all functions.

---

# Contributing

This repository is intended as an interoperability and research PoC.

Contributions are particularly useful in the following areas:

- DID method interoperability;
- IPFS/IPNS resolution;
- DID Document validation;
- Peer ID verification;
- DID ↔ Peer ID binding;
- ANP transport bindings;
- libp2p stream handling;
- interoperability testing;
- security testing;
- key rotation and revocation;
- documentation and protocol analysis.

When proposing changes, please clearly identify whether the change affects:

1. **Agent identity**
2. **DID resolution**
3. **Peer authorization**
4. **libp2p transport**
5. **ANP messaging**
6. **Security semantics**

Keeping these concerns explicit will help preserve the separation between identity, content, networking, and messaging.

---

# Current Status

This repository describes and implements the **ANP × libp2p × IPFS interoperability PoC direction**.

The architecture described here should be considered **experimental** until validated through interoperable implementations and security testing.

In particular, the following are PoC design elements rather than established production standards:

- `ANPLibp2pService`;
- the specific DID ↔ Peer ID binding model;
- the proposed `/anp/1.0` libp2p transport binding;
- IPFS/IPNS DID integration details;
- Peer Verification Method conventions;
- peer rotation semantics.

The PoC is intended to provide concrete implementation experience that can inform subsequent protocol specifications and interoperability standards.

---

# License

See the repository license for the applicable terms.

---

## Summary

The proposed architecture connects three otherwise distinct concepts:

```text
        IDENTITY
           │
           ▼
       W3C DID
           │
           │ authorizes
           ▼
       Peer Key
           │
           ▼
       libp2p Peer ID
           │
           │ connects
           ▼
       libp2p
           │
           │ transports
           ▼
         ANP
           │
           │ authenticates
           ▼
      Agent Identity
```

At the same time, IPFS/IPNS provides decentralized publication and versioning of the identity metadata:

```text
DID
 │
 ▼
IPNS
 │
 ▼
IPFS CID
 │
 ▼
DID Document
```

The central hypothesis is that **agent identity, network identity, content identity, and messaging transport can remain independently meaningful while being cryptographically linked where necessary**.

This PoC provides a path toward interoperable agents that can move between web and peer-to-peer environments without requiring the agent's long-term identity to be tied to a particular transport, endpoint, or network node.
