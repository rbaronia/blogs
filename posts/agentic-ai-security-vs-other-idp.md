# Securing the Agentic AI Frontier: Why IBM Verify is the Critical Identity Layer

## Executive Summary
The identity industry has long focused on managing humans, their account lifecycle, and runtime access. However, with the scale-out of cloud infrastructure and automation, these systems are not well-positioned to handle machine accounts. **Non-Human Identities (NHIs)** now outnumber humans by 100s:1, and so too is the attack surface.

Agentic AI programs incorporate both humans and machines in end-to-end workflows, bringing both a cyber risk and an accountability gap. Organizations face a crisis of accountability—as they will never be able to prove, with non-repudiation and integrity, a person was responsible for the consequence of an action. Unlike traditional software, Agentic AI systems exhibit autonomy, including the ability to independently invoke tools and execute transactional workflows. This necessitates a transition to a security architecture capable of supporting phishing-resistant authentication, fine-grained authorization, and cryptographically bound identity.

---

## Technical Superiority: IBM Verify vs. Microsoft Entra ID
Differentiation between major identity providers is increasingly exposed as agentic scenarios are deployed. While Microsoft Entra ID remains a ubiquitous solution for workforce identity, its architecture is frequently constrained by proprietary implementations. Conversely, IBM Verify provides a robust standards-compliant framework.

### Standard Comparison Table

| Capability | IBM Verify (Standards-Based) | Microsoft Entra ID |
| :--- | :--- | :--- |
| **RAR (RFC 9396)** | Supported via `authorization_details` | Not Supported |
| **DPoP (RFC 9449)** | Native cross-platform application layer | Broker-dependent (Windows) / Experimental |
| **CIBA (Core)** | Supported (Poll and Ping modes) | Not a standard OIDC implementation |
| **PAR (RFC 9126)** | Central to FAPI profiles; fully supported | Selective support |
| **Token Exchange** | RFC 8693 (Standard) | OBO (Proprietary) |

---

## Core Technical Differentiators

### 1. Dynamic and Granular Authorization: RAR (RFC 9396)
IBM Verify supports **OAuth 2.0 Rich Authorization Requests**, allowing agents to request specific permissions (e.g., "Transfer $100 from Account A to B") rather than a blanket "Execute transfers" scope.

* **IBM Verify:** Uses JSON schema files to validate incoming request details.
* **Entra ID:** Restricted to string-based scopes pre-registered in the app manifest, often forcing developers to over-permission AI agents.

### 2. Cryptographic Sender Constraining: DPoP (RFC 9449)
Standard bearer tokens present an unacceptable risk if leaked. DPoP ensures tokens are bound to the client via an asymmetric key pair. IBM Verify embeds a public key thumbprint in the `cnf` (confirmation) claim:

$$jkt = \text{Base64URL}(\text{SHA-256}(\text{JWK\_Public\_Key}))$$

### 3. Identity Propagation: RFC 8693 vs. Entra OBO
The most critical differentiator in multi-hop agentic security is how identity moves across services.

* **Limitations of Entra OBO:** Proprietary and non-interoperable. It often creates "token bloat" and makes the intermediate agent "invisible," losing the link to the originating user if a service principal is used.
* **IBM Verify Token Exchange:** Implements the full **RFC 8693** spec. It allows for a `subject_token` (user) and an `actor_token` (agent). The resulting token includes an `act` claim, ensuring total auditability of who acted for whom.

---

## Securing Interaction Models

### User-to-Agent (U2A)
In the U2A model (e.g., a "Copilot"), the goal is delegated authority.
* **Granular Consent:** Using RAR to show the user exactly what the agent will do.
* **Decoupled Step-up:** Using **CIBA** (Client-Initiated Backchannel Authentication) to trigger a push challenge to the user’s mobile device for just-in-time approval, even if the agent is operating in the background.

### Agent-to-Agent (A2A)
In the A2A model (e.g., an orchestration agent calling a sub-agent), the goal is cryptographic accountability.
* **Chain of Custody:** By using RFC 8693, interactions maintain a nested actor chain. A downstream service can trace authority back from "Sub-Agent B" to "Master Agent A" and finally to the human user.

---

## Continuous Risk: The Antenna Framework
Agentic security requires a dynamic response to changing risk postures. **IBM Verify Antenna** is a lightweight, self-hosted container that enables the exchange of security events using the Shared Signals Framework (SSF). It allows for real-time revocation of a rogue agent’s access when anomalous behavior is detected.

## IBM-HashiCorp Synergy
Beyond protocol support, IBM provides a comprehensive suite:
* **Identity Protection (VIP):** Uses AI/ML to discover "shadow" assets and unmanaged agents.
* **Dynamic Credentials with Vault:** Issues short-lived, just-in-time credentials, removing the risk of static secrets.
* **Vault Radar:** Scans code for exposed cloud keys or API tokens before deployment.

---

## Summary
Entra ID is an excellent workforce platform, but Agentic AI introduces control points that require open-standard solutions. **IBM Verify** provides a superior platform for runtime security by adhering strictly to RAR, DPoP, CIBA, and RFC 8693. This approach addresses the "accountability gap" and "token bloat" inherent in proprietary models, enabling a traceable thread through all AI-driven workloads.