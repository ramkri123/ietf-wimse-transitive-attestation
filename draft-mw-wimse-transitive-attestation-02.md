%%%
title = "Transitive Attestation for Sovereign Workloads: A WIMSE Profile"
abbrev = "WIMSE-TRANS-POR"
category = "info"
docName = "draft-mw-wimse-transitive-attestation-02"
ipr = "trust200902"
area = "Security"
workgroup = "WIMSE"
keyword = ["attestation", "wimse", "transitive", "workload identity", "residency"]

[seriesInfo]
name = "Internet-Draft"
value = "draft-mw-wimse-transitive-attestation-02"
status = "informational"

[[author]]
initials = "R."
surname = "Krishnan"
fullname = "Ram Krishnan"
organization = "JPMorgan Chase & Co"
  [author.address]
  email = "ramkri123@gmail.com"

[[author]]
initials = "A."
surname = "Prasad"
fullname = "A Prasad"
organization = "Oracle"
  [author.address]
  email = "a.prasad@oracle.com"

[[author]]
initials = "D."
surname = "Lopez"
fullname = "Diego R. Lopez"
organization = "Telefonica"
  [author.address]
  email = "diego.r.lopez@telefonica.com"

[[author]]
initials = "S."
surname = "Addepalli"
fullname = "Srinivasa Addepalli"
organization = "Aryaka"
  [author.address]
  email = "srinivasa.addepalli@aryaka.com"


%%%

.# Abstract

This document defines a **WIMSE Profile** for Transitive Attestation within the Workload Identity in Multi-Service Environments (WIMSE) framework. It addresses the critical problem of **Identity Portability**, where software credentials (e.g., bearer tokens or keys) can be misappropriated and used from unauthorized environments—a risk amplified by the emergence of autonomous **AI Agents** that may move across jurisdictions or be hijacked via **prompt injection attacks**. By providing a standardized **Identity Conveyance** mechanism, this profile cryptographically binds software workloads to their local execution environment ("Proof of Residency") through a transitive chain of trust. This chain consumes **Evidence** from the underlying platform—supporting both **high-assurance** RATS-based profiles (e.g., [[!I-D.lkspa-wimse-verifiable-geo-fence]]) for residency verification and **standard** Workload Identity Agents for basic co-location proofs—to ensure that an identity is only valid when used from a verified, integral, or geographically compliant host.

{mainmatter}

# Introduction

This document defines the **WIMSE Profile** for "Transitive Attestation", addressing a critical technical gap in the high-level **WIMSE Architecture** [[!I-D.ietf-wimse-arch]] regarding how platform-level trust is transitively extended to software workloads. 

Currently, workload identities are often "context-agnostic"—once a credential (e.g., a bearer token) is issued, it can often be used from any environment. This **Identity Portability** allows an attacker who steals a token in one jurisdiction to use it from another, representing a significant **Sovereignty Violation** for workloads legally required to operate within specific boundaries. This is particularly critical for **AI Agents**, whose autonomous nature, susceptibility to hijacking via **prompt injection attacks**, and potential for rapid migration across cloud environments require strict, verifiable adherence to data residency and host integrity policies.

By addressing the **North-South** security axis of a workload's relationship with its local hosting environment, this profile establishes a **"Silicon-to-SVID"** chain of accountability. It ensures that the Workload Identity Agent (the local agent) is empowered to issue identities that are cryptographically bound to a verified execution context. This mechanism is flexible across assurance levels: it supports **High Assurance** residency verification rooted in hardware evidence (e.g., TPM/TEE), as well as **Standard Assurance** local co-location proofs provided by conventional workload agents.

This draft acts as the **Conveyance Layer** that integrates with the findings of a **RATS Profile** (such as **Verifiable Geofencing** [[!I-D.lkspa-wimse-verifiable-geo-fence]]) to establish two distinct levels of assurance:

*   **Co-location Verification**: A logical binding that ensures the workload and its agent are currently co-located on the same host, typically enforced via operating system isolation and local communication channels (e.g., Unix Domain Sockets).
*   **Residency Verification (High Assurance)**: A high-assurance binding where the Workload Identity Agent itself is proven to be integral and rooted in hardware. This ensures the identity is functionally "sticky" to the verified residence via a **Fast Path** renewal which uses cached attestation results.

A workload obtains a fresh signature or proof from a local Workload Identity Agent, typically utilizing a **Workload Fusion Nonce (N_fusion)** to prevent replay. This ensures the identity—and the usage of its associated credentials—is sensitive to the physical or logical residence of the workload, complementing the "East-West" delegation models. Re-attestation follows a **Tiered Schedule** (see [[!I-D.lkspa-wimse-verifiable-geo-fence]]), separating frequent identity renewal from heavyweight platform evidence collection.

# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [[RFC2119]] [[RFC8174]] when, and only when, they appear in all capitals, as shown here.

This document leverages the terminology defined in the RATS Architecture [[RFC9334]] and the WIMSE Architecture [[!I-D.ietf-wimse-arch]].

Workload Identity Agent (Workload Identity Agent):
: A local entity that acts as an **Attester** or **Attestation Intermediate** in the RATS framework. It is responsible for providing Evidence or Attestation Results to a workload.

Proof of Residency (PoR) / Co-location:
: A cryptographic proof that binds a workload's current execution session to a specific, verified local environment or host.

N_fusion (Workload Fusion Nonce):
: A nonce provided by the Resource Server or Workload Identity Agent specifically for the workload-to-agent fusion flow. To prevent "mix-and-match" attacks, the **Host Management Plane** SHALL cryptographically bind `N_fusion` to the platform-level nonce (**N_platform**) within its signed Attestation Result.

Workload identities are often represented by bearer tokens or keys that, once compromised, can be used by an attacker from any environment. This "portability" allows an attacker who achieves RCE on a workload (e.g., in Region A) or hijacks an AI agent's logic via **prompt injection** to use the stolen keys or intercepted tokens from an attacking machine (e.g., in Region B).

In the context of **Sovereign Workloads**, this portability is more than a technical vulnerability; it is a **Sovereignty Violation**. If a workload is legally or logically required to operate within a specific jurisdiction, any identity that can be successfully verified from outside that jurisdiction represents a failure of the sovereign boundary. **Workload Identity Managers** and Relying Parties currently lack a standardized way to ensure that a presented identity is being used from the same verified host that was originally authorized.

# The Solution: Transitive Attestation for Proof of Residency

"Transitive Attestation" establishes a chain of trust from a hardware root through a local agent to the workload. The Workload Identity Agent provides the workload with a "live" proof that it is currently resident on the verified host. This local peer-to-peer connection is typically enforced through a **Unix Domain Socket (UDS)**, providing a kernel-level guarantee that the workload is co-located with the hardware-rooted agent.

## mTLS-based Transitive Attestation

In an mTLS environment, the Proof of Residency (PoR) is bound to the mutually authenticated session and the local execution context via a transitive chain of trust.

### mTLS PoR Protocol Flow

The mTLS-based flow integrates residency verification into the session establishment and validation phase:

1. **Certificate Extensions**: The client (workload) supplies an X.509 certificate during the mTLS handshake containing a custom extension. This extension includes the public key or SVID details of the local Workload Identity Agent (Attester).
2. **Post-Handshake Nonce**: After the mTLS handshake is successfully completed, the client requests a residency-specific nonce from the **Workload Identity Manager (Workload Identity Manager)** or Resource Server (Verifier/Relying Party) to ensure anti-replay.
3. **Local Attestation Binding**: The client constructs a PoR assertion payload containing:
    - A cryptographic hash of the TLS Exporter value [[RFC5705]].
    - The residency nonce provided by the server.
    - A timestamp representing the current time of assertion creation.
<br>

> [!NOTE]
> By binding to the TLS Exporter instead of the application traffic keys, the Proof of Residency remains valid across TLS 1.3 `KeyUpdate` operations. Standard key rotation refreshes traffic keys but does not change the exporter master secret, thus avoiding unnecessary re-attestation cycles while maintaining strong cryptographic binding to the session.

4. **Agent Signature**: The client sends this payload to the local Workload Identity Agent (typically via a Unix Domain Socket). The Workload Identity Agent verifies the local peer environment and signs the payload with its private key.
5. **PoR Submission**: The client sends this attested response to the resource server for verification.
6. **Workload Identity Manager or Server Verification**: The **Workload Identity Manager (Workload Identity Manager)** or Resource Server performs a joint verification of identity and residency:
    - **Identity**: Verifies the client certificate as part of standard mTLS.
    - **Residency**: Verifies the PoR assertion signature against the Workload Identity Agent public key found in the client's certificate extension.
    - **Binding and Freshness**: Ensures that the TLS Exporter hash, the nonce, and the timestamp match the current active session and are within an acceptable freshness window.

Upon successful verification, the resource server has proof that the client identity (presented via mTLS) is currently resident in the same authorized environment as the verified Workload Identity Agent.

"Demonstrating Proof of Residency" (DPoR) is an enhancement to the Demonstrating Proof-of-Possession (DPoP) mechanism defined in [[RFC9449]]. While DPoP ensures *possession* of a private key held by the client, DPoR ensures the physical or logical *residency* of the workload using that key by binding the request to a local attestation.

### DPoR Protocol Flow

The DPoR flow integrates residency verification into the per-request application-level authorization:

1. **Nonced Request**: The **Workload Identity Manager (Workload Identity Manager)** or Resource Server SHOULD provide a residency-specific nonce (e.g., via a `DPoR-Nonce` header) to the client to ensure anti-replay of the residency proof.
2. **Local Attestation Binding**: The client constructs a DPoR assertion payload containing:
    - The hash of the DPoP public key used for the request (e.g., the `jkt` thumbprint).
    - The residency nonce provided by the server.
    - A timestamp representing the current time of assertion creation.
3. **Agent Signature**: The client sends this payload to the local Workload Identity Agent (typically via a Unix Domain Socket). The Workload Identity Agent (acting as an Attester) verifies the local execution context and signs the payload with its private key.
4. **DPoR Assertion Submission**: The client includes the resulting signature in a `DPoR` header or as an extension to the `DPoP` JWT.
5. **Workload Identity Manager or Server Verification**: The **Workload Identity Manager (Workload Identity Manager)** or Resource Server performs a joint verification of possession and residency:
    - **Possession**: Verifies the DPoP proof as per [[RFC9449]].
    - **Residency**: Verifies the DPoR assertion signature against the Workload Identity Agent public key.
    - **Binding and Freshness**: Ensures that the `jkt` (DPoP key thumbprint), the nonce, and the timestamp in the residency proof match the current request and are within an acceptable freshness window.

This binding ensures that a DPoP key cannot be "exported" and used from a different machine, as the resource server would detect the lack of a valid, hardware-rooted residency proof for that specific key from the new environment.

# Relation to Other IETF Work

| Layer | Component | WG | Core Responsibility |
| :--- | :--- | :--- | :--- |
| **Layer 1** | **Transitive Attestation** | **WIMSE** | **Conveyance**: Binds identity to the local agent (Co-location/Residency). |
| **Layer 2** | **Verifiable Geofencing** | **RATS** | **Platform Evidence**: Verifies host integrity and Workload Identity Agent hardware residency (TPM). |
| **Layer 3** | **Verifiable Geofencing** | **RATS** | **Location Evidence**: Verifies physical geography (GNSS/ZKP). |
| **Delegation** | **Actor Chain** | **SPICE** | Provides **East-West** identity delegation proof [[!I-D.draft-mw-spice-actor-chain]]. |
| **Shield** | **SPICE** | **SPICE** | Employs Selective Disclosure (SD-CWT) to protect residency/geographic privacy. |

1.  **Transitive Attestation (WIMSE) - Layer 1 (Conveyance)**: This document act as the technical integrator profile for WIMSE. It standardizes how local context results are transitively extended to workloads. It reflects its status as the **Consumer** of attestation results to address the **North-South** identity portability problem.
2.  **Verifiable Geofencing (RATS) - Layer 2 & 3 (Evidence)**: Defined in [[!I-D.lkspa-wimse-verifiable-geo-fence]]. This document acts as the **RATS Profile** that provides the hardware-rooted foundation (TPM, Silicon Root of Trust, GNSS) and the out-of-band monitoring required to verify the Workload Identity Agent itself. It generates the high-assurance evidence that Layer 1 consumes.
3.  **Actor Chain - The Delegation**: Complements this draft by addressing the **East-West** axis of agent-to-agent communication [[!I-D.draft-mw-spice-actor-chain]]. While Transitive Attestation proves *where* an actor is running (North-South), the Actor Chain proves *who* called whom across the network (East-West).
4.  **SPICE (Secure Patterns for Internet Credential Exchange) - The Shield**: Utilizes Selective Disclosure (SD-CWT) to protect sensitive location data. It allows a workload to prove residency within a broad "Sovereign Zone" without revealing precise GPS coordinates, balancing security with privacy.

Additional relationships include:

*   **Verifiable Geofencing [[!I-D.lkspa-wimse-verifiable-geo-fence]]**: Provides the normative technical specification for Layer 2 and Layer 3 attestation. This draft (Transitive Attestation) acts as the application-layer profile that implements these residency proofs within mTLS and DPoP flows, fulfilling the "Silicon-to-SVID" chain.
*   **WIMSE Architecture [[!I-D.ietf-wimse-arch]]**: This draft provides the technical fulfillment for the secure agent requirements and thread model mitigations (e.g., token theft) defined in the architecture.

# Other Related Efforts

Outside of the IETF, this proposal aligns with several industry standards for secure workload execution:

- **CNCF SPIFFE/SPIRE**: This draft formalizes the application-layer binding for SPIRE's node-to-workload attestation chain. It ensures that the short-lived SVIDs issued by SPIRE are cryptographically bound to the hardware-rooted residency assertion provided by the SPIRE Agent (acting as the Workload Identity Agent).
- **Confidential Computing Consortium (CCC)**: Proof of Residency (PoR) provides the cryptographic evidence required for "**Sovereign Workloads**" and "Data-in-Use" protection models. In Confidential Computing (CC) environments, the hardware itself can generate direct, cryptographically signed quotes (e.g., using AMD SEV-SNP VCEK/VLEK keys). These quotes typically include two distinct layers of evidence:
    1.  **Platform Attributes**: Measurements of the processor's identity, microcode version (TCB), and hardware security state.
    2.  **Workload Measurements**: Measurements of the workload's code/memory image and custom metadata (e.g., via the `REPORT_DATA` field).

  While these direct quotes provide high-assurance hardware-direct residency, the **Transitive Attestation** model specified in this document acts as the essential **Identity Bridge**. This bridge can manifest in two architectural patterns:
    1.  **Agent-Mediated Flow**: Traditional in standard environments where a local Workload Identity Agent (e.g., a SPIRE Agent) performs the local attestation and translation before the workload receives its identity.
    2.  **Direct Quoting Flow**: Typical in CC TEEs where the workload performs "Direct Quoting" of the hardware state. In this pattern, the **transitive mapping** from hardware evidence to application-layer identity (SVID) typically occurs at a remote **Workload Identity Manager (Workload Identity Manager)** (e.g., a SPIRE Server) during the initial credential issuance phase.

  In both patterns, the result is the same: the high-level application identity becomes cryptographically bound to the hardware-rooted residency, ensuring that Verifiers and Relying Parties do not need to implement complex, vendor-specific measurement verification logic for every diverse hardware platform.

# Security Considerations

Proof of Residency or Co-location specifically mitigates the "Stolen Credential Portability" threat, which encompasses both stolen private keys and stolen bearer/DPoP tokens. 

An attacker who steals a private key or intercepts an active token from a workload cannot use those credentials from an external environment. Any attempt to use the stolen credential requires a corresponding PoR assertion that is:

1.  **Locally Attested**: Linked to the local Workload Identity Agent's signing interface, typically protected by OS permissions or hardware roots.
2.  **Context-Specific**: Bound to a fresh, server-provided nonce and a current timestamp.
3.  **Protected**: Access to the Workload Identity Agent's signing capability is restricted by local Operating System permissions and logical isolation.

Consequently, credentials become functionally "sticky" to the verified residence; an attacker cannot generate a valid proof without achieving a compromise of the identity agent itself. While a software-based agent provides strong logical isolation, a hardware-rooted agent (see [[!I-D.lkspa-wimse-verifiable-geo-fence]]) provides the highest level of protection against agent cloning and export. 

The security of this model relies on the **Host Management Plane** correctly binding the identity renewal challenge (`N_fusion`) to a fresh, OOB platform quote (`N_platform`). If a verifier accepts a residency proof that is not bound to a fresh hardware quote, the "Silicon-to-Audit" chain is broken.

TBD: Discussion on Workload Identity Agent compromise, nonce entropy requirements, and clock skew for timestamp verification.

# IANA Considerations

This document has no IANA actions at this time.

{backmatter}

<reference anchor="RFC5705" target="https://www.rfc-editor.org/rfc/rfc5705">
  <front>
    <title>Keying Material Exporters for Transport Layer Security (TLS)</title>
    <author initials="E." surname="Rescorla" fullname="Eric Rescorla"/>
    <date month="March" year="2010"/>
  </front>
</reference>

<reference anchor="I-D.ietf-wimse-arch" target="https://datatracker.ietf.org/doc/html/draft-ietf-wimse-arch">
  <front>
    <title>Workload Identity in a Multi System Environment (WIMSE) Architecture</title>
    <author initials="Y." surname="Sheffer" fullname="Yaron Sheffer"/>
    <date month="October" day="21" year="2024"/>
  </front>
</reference>

<reference anchor="I-D.ietf-wimse-s2s-protocol" target="https://datatracker.ietf.org/doc/html/draft-ietf-wimse-s2s-protocol">
  <front>
    <title>WIMSE Service to Service Authentication</title>
    <author initials="P." surname="Howard" fullname="Pieter Howard"/>
    <date month="October" day="21" year="2024"/>
  </front>
</reference>

<reference anchor="I-D.lkspa-wimse-verifiable-geo-fence" target="https://datatracker.ietf.org/doc/html/draft-lkspa-wimse-verifiable-geo-fence">
  <front>
    <title>Verifiable Geofencing and Residency Proofs for Sovereign Workloads: A RATS Profile</title>
    <author initials="D." surname="Lopez" fullname="Diego Lopez"/>
    <date month="February" day="11" year="2025"/>
  </front>
</reference>

<reference anchor="I-D.novak-twi-attestation" target="https://datatracker.ietf.org/doc/html/draft-novak-twi-attestation">
  <front>
    <title>Remote Attestation for Trustworthy Workload Identity</title>
    <author initials="N." surname="Novak" fullname="Ned Novak"/>
    <date month="July" day="4" year="2024"/>
  </front>
</reference>
