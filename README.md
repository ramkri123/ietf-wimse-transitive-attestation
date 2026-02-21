# WIMSE: Transitive Attestation for Workload Proof of Residency

[![IETF Draft](https://img.shields.io/badge/IETF-Draft-blue.svg)](https://datatracker.ietf.org/doc/draft-mw-wimse-transitive-attestation/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## Overview

This repository contains the IETF Internet-Draft for **Transitive Attestation**, a mechanism within the **Workload Identity in Multi-Service Environments (WIMSE)** framework designed to solve the problem of **Identity Portability** and **Credential Theft**.

By introducing **Proof of Residency (PoR)**, this proposal anchors software workloads (including AI agents) to a verified, hardware-rooted local environment, making credentials functionally "sticky" to their authorized host.

## The Problem: Identity Portability

Current workload identity mechanisms (like DPoP) focus on binding tokens to keys. However, if a private key or session token is stolen (via RCE or prompt injection), it can be used by an attacker from an unverified environment. This "portability" is a critical security gap in distributed cloud and sovereign AI workloads.

## The Solution: Proof of Residency (PoR)

"Transitive Attestation" establishes a chain of trust from a hardware root (TPM/Secure Enclave) through a local **Workload Identity Agent (WIA)** to the workload itself.

### Key Protocols
- **mTLS-based PoR**: Binds residency verification into the session establishment.
- **DPoR (Demonstrating Proof of Residency)**: An enhancement to RFC 9449 (DPoP) that binds application-level requests to hardware attestation.

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
