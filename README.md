# WIMSE: Transitive Attestation for Workload Proof of Residency

[![IETF Draft](https://img.shields.io/badge/IETF-Draft-blue.svg)](https://datatracker.ietf.org/doc/draft-mw-wimse-transitive-attestation/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## Overview

This repository contains the IETF Internet-Draft for **Transitive Attestation**, a mechanism within the **Workload Identity in Multi-Service Environments (WIMSE)** framework designed to solve the problem of **Identity Portability** and **Credential Theft**.

By introducing **Proof of Residency (PoR)**, this proposal anchors software workloads (including AI agents) to a verified, hardware-rooted local environment, making credentials functionally "sticky" to their authorized host.

## The Problem: Identity Portability

Current workload identity mechanisms (like DPoP) focus on binding tokens to keys. However, if a private key or session token is stolen (via RCE or prompt injection), it can be used by an attacker from an unverified environment. This "portability" is a critical security gap in distributed cloud and sovereign AI workloads.

## The Solution: Transitive Attestation for Proof of Residency

"Transitive Attestation" establishes a chain of trust from a hardware root through a local agent (or directly from hardware) to the workload.

### The Identity Bridge
The core of this proposal is the **Identity Bridge**—a mechanism that maps low-level hardware claims into high-level **Application Identities (SVIDs)**. This ensures that Verifiers receive a standardized Proof of Residency (PoR) without needing to understand diverse, vendor-specific silicon measurement schemas.

### Architectural Patterns
The Transitive Attestation model supports two primary flows:
1.  **Agent-Mediated Flow**: Traditional in standard environments where a local **Workload Identity Agent (WIA)** (e.g., SPIRE Agent) performs local attestation and translation.
2.  **Direct Quoting Flow**: Typical in Confidential Computing (CC) TEEs where the workload performs "Direct Quoting" of the hardware state, and the **transitive mapping** to SVIDs is handled by a remote **Verifier/CA** (e.g., SPIRE Server).

### Technical Evidence
A Proof of Residency assertion (PoR) typically consolidates two layers of evidence:
- **Platform Attributes**: Processor identity, microcode version (TCB), and security state.
- **Workload Measurements**: Cryptographic hashes of the code/memory image and custom metadata (e.g., `REPORT_DATA`).

### Local Binding Mechanism
In the **Agent-Mediated flow**, the workload connects to the local WIA through a **Unix Domain Socket (UDS)**. This kernel-enforced communication channel provides the initial cryptographic guarantee of local residency, ensuring the requester is co-located with the hardware-rooted agent.

## Chain of Accountability

The proposal forms a three-layer trust stack that aligns IETF standards with physical reality:

| Layer | Component | Core Responsibility |
| :--- | :--- | :--- |
| **The Mechanism** | **RATS** | Hardware primitives (TPM, PTP, Geo-sensors). |
| **The Policy** | **WIMSE** | (This Draft) Solves identity portability via Transitive Attestation. |
| **The Shield** | **SPICE** | Privacy-preserving Selective Disclosure (SD-CWT). |

## Strategic Alignment

This project is designed to integrate with:
- **CNCF SPIFFE/SPIRE**: Formalizing the binding for SPIRE's node-to-workload attestation.
- **Confidential Computing Consortium (CCC)**: Grounding residency in TEE execution models.
- **IETF Verifiable Geofencing**: Acting as the technical implementation profile for residency proofs.

## Building the Draft

The draft is written in Markdown and uses `mmark` and `xml2rfc` for conversion.

### Prerequisites
- [mmark](https://github.com/mmark-md/mmark)
- [xml2rfc](https://pypi.org/project/xml2rfc/)

### Build Commands
```bash
# Generate TXT, HTML, and XML outputs
make

# Clean build artifacts
make clean
```

## Contributing

This is an active IETF submission. Feedback is welcome via GitHub issues or the [WIMSE mailing list](https://www.ietf.org/mailman/listinfo/wimse).
