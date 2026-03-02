# Securing the Agentic AI Frontier: Why IBM Verify is the Critical Identity Layer

# Executive Summary

The identity industry has long focussed on managing humans, their account lifecycle and runtime access.  A lot of these processes are based on traditional static controls.  With the scale out of cloud infrastructure, and automation, these systems are not well positioned to handle machine accounts, and attackers are exploiting them in successful attacks.    Non-Human Identities (NHIs) now outnumber humans by 100s:1 and increasing, and so too is the attack surface exploitable by these attackers.  Agentic AI programs incorporate both humans and machines in end to end workflows, and with that brings both a cyber risk and an accountablity gap.  If not closed, orgnizations face a crisis of accountability - as they will never be able to prove, with non-repudiation and integrity, a person was responsible for consequence of action.  

The emergence of agentic artificial intelligence ( Agentic AI) has fundamental implications for the existing paradigms of identity and access management (IAM). Unlike traditional software applications that operate within static parameters, Agentic AI systems exhibit autonomy, including the ability to independently invoke tools, negotiate cross-domain resource access, and execute transactional workflows on behalf of users.  Customers expect solutions to operate within their existing technological landscape - which necessitates an interoperable approach that works at speed and with safety.    

A full end to end identity security strategy demands a number of elements in order to deliver the runtime security accountability:

- **Unified Identity Fabric:** Bridging human and NHI management into a single platform for centralized audit trails.
- **Discovery & Inventory:** Automatically identifying all service accounts, bots, and AI agents across cloud and on-premises systems.
- **Automated Lifecycle Management:** Standardized creation using templates with clear human ownership tagging.
- **AI-Powered Posture Management:** Real-time risk scoring to detect behavioral drift in agents.
- **Just-in-Time, Just-Enough Access:** Transforming long-lived NHIs into demand-driven entities with precisely scoped permissions.

This shift necessitates a transition from human-centric authentication models to a security architecture capable of supporting phishing resistent authentication, complex delegation, fine-grained authorization, full tracability in human and account owners and cryptographically bound identity.   

If this is rationalised down to the runtime security elements, the core requirements are implemented by supporting advanced WebAuthn, OpenID Connect (OIDC) and OAuth 2.0 extensions.   Differentiation between major identity providers—e.g. IBM Verify and Microsoft Entra ID—is increasingly exposed as these scenarios are deployed within client environments.  

Since these technical standards exist today, we take this opportunity to explore the critical decision points on implementation.  This paper explores that runtime access management element.  It does this within the context of a typical enterprise, one which has subscribed to a Microsoft Entra ID entitlement.  While Microsoft Entra ID remains a ubiquitous solution for general workforce identity, its architecture is frequently constrained by proprietary implementations due to resistance in adopting emerging IETF standards. Conversely, IBM has demonstrated, and continues to advocate for strong open standards.  IBM Verify demonstrates this by providing a robust standards-compliant framework for securing the high-churn, distributed environments typical of agentic AI.   

---

## The Identity Imperative in Agentic AI Ecosystems

The security requirements for agentic AI extend significantly beyond those of conventional applications. Traditional OAuth 2.0 and OIDC flows assume a synchronous user session where a human provides real-time consent. Agentic AI, however, frequently operates in headless, background, or decoupled environments where such interactions are impractical. Furthermore, agents often initiate recursive delegation chains requiring a specific subset of the original user's permissions. In such environments, the principle of least privilege is a functional requirement for mitigating the risks of privilege escalation and autonomous decision-making errors.   

Standard bearer tokens, where simple possession is sufficient for authorization, present an unacceptable risk. If a bearer token is leaked from an agent’s cache or intercepted during transit, it can be replayed by an adversary without cryptographic proof. To address this, agentic systems require sender-constraining mechanisms like DPoP (Demonstration of Proof-of-Possession) to ensure tokens are bound to the client. Additionally, coarse-grained OAuth scopes are insufficient for agents performing specific, parameterized tasks. The emergence of Rich Authorization Requests (RAR) provides the framework necessary for expressing these fine-grained intentions.   

---

## Technical Superiority of IBM Verify in Advanced OIDC Standards

IBM Verify’s value lies in its "API-first" and "standards-first" directive, leading to the implementation of multiple OIDC extensions that Microsoft Entra ID has either not adopted or has implemented in a limited capacity.

### Dynamic and Granular Authorization: Rich Authorization Requests (RAR)

IBM Verify supports OAuth 2.0 Rich Authorization Requests (RFC 9396), allowing clients to request detailed attributes during authorization.

> In an agentic AI scenario, an agent might need permission to "Execute a transfer of $100 from Account A to Account B" rather than a blanket "Execute transfers" permission.

IBM Verify’s RAR implementation enables the authorization server to make informed decisions through configuration parameters:

- **authorization_details_types_supported:** Defines specific types the server can process, such as *account_information* or *payment_initiation*.
- **authorization_details_types_schema:** Leverages JSON schema files to validate incoming request details, ensuring necessary transactional context is provided before consent.
- **identifier_strategy:** Allows custom functions to compute unique identifiers for complex authorization details.

In contrast, Microsoft Entra ID does not support the OAuth 2.0 RAR specification. Entra ID remains restricted to string-based scopes that must be pre-registered in the app manifest, currently forcing developers to over-permission their AI agents because arbitrary JSON structures cannot be handled at runtime.   

## Cryptographic Sender Constraining: Demonstration of Proof-of-Possession (DPoP)

DPoP (RFC 9449) is essential for preventing token replay by cryptographically binding the access token to a client-generated key pair. IBM Verify provides native support for DPoP across all grant types, including *authorization_code* and *client_credentials*. The process involves:

1. **Key Generation:** The agent generates an asymmetric key pair.
2. **DPoP Proof:** The agent includes a `DPoP` header containing a signed JWT with a public key and claims such as `htu` (HTTP Target URI) and `htm` (HTTP Method).
3. **Token Binding:** IBM Verify validates the proof and embeds a public key thumbprint in the `cnf` (confirmation) claim of the token. The thumbprint is calculated as:
jkt=Base64URL(SHA-256(JWK_Public_Key))

Microsoft Entra ID’s PoP implementation is primarily optimized for public clients running on Windows using the Windows Account Manager (WAM) broker. PoP for confidential clients (server-side agents) is considered limited and platform-constrained and lacks support for multi-platform environments, creating a platform-dependent model that is less suitable for modern, containerized AI workloads.

## Decoupled Interaction: Client-Initiated Backchannel Authentication (CIBA)

A critical requirement for agentic AI is triggering authentication when the user is not present at the console. CIBA allows a client to trigger an authentication flow without front-channel redirects. IBM Verify supports CIBA with *Poll* and *Ping* modes, allowing an agent to securely initiate a push challenge (via the `/oauth2/ciba` endpoint) to the user’s mobile device to authorize a high-value action. Entra ID lacks a standardized OIDC CIBA implementation for third-party developers, relying instead on proprietary notification systems within its own ecosystem.


## Secure Metadata: Pushed Authorization Requests (PAR)

Pushed Authorization Requests (PAR - RFC 9126) address URL leakage by requiring the client to push the authorization payload to a secure back-channel endpoint. IBM Verify implements PAR as a core feature, receiving a request_uri in return. This ensures integrity, confidentiality, and scalability for complex requests. PAR support exists in limited SDK and framework contexts and is not uniformly enforced across the Entra ID platform.

### Standard Comparison Table

| Capability | IBM Verify (Standards-Based) | Microsoft Entra ID |
| :--- | :--- | :--- |
| **RAR (RFC 9396)** | Supported via `authorization_details` | Not Supported |
| **DPoP (RFC 9449)** | Native cross-platform application layer | Broker-dependent (Windows) / Experimental |
| **CIBA (Core)** | Supported (Poll and Ping modes) | Not a standard OIDC implementation |
| **PAR (RFC 9126)** | Central to FAPI profiles; fully supported | Selective support |
| **Token Exchange** | RFC 8693 (Standard) | OBO (Proprietary) |

## Identity Propagation: RFC 8693 vs. Entra OBO

The most critical differentiator in multi-hop agentic security is how identity is propagated across services. IBM Verify adheres to the IETF RFC 8693 OAuth 2.0 Token Exchange standard, while Entra ID relies on its proprietary On-Behalf-Of (OBO) flow.   

### Limitations of Microsoft Entra OBO Flow
Microsoft’s OBO flow suffers from several architectural constraints that hinder agentic security:
1. ***Proprietary and Non-Interoperable:*** OBO is a custom Microsoft implementation. This creates a "walled garden" where agents cannot easily propagate identity to third-party services (e.g., non-Azure APIs) without building custom "shim" layers.
2. ***Token Bloat and Context Leakage:*** OBO typically replicates the user's entire identity context into the downstream token. This "token bloat" often leads to over-permissioning because the downstream service receives the user's full context rather than a purpose-scoped subset.   
3. ***Invisibility of the Actor:*** In the OBO model, the intermediate agent is often "invisible" in the new token. The downstream resource sees the user's identity but lacks a structured, cryptographically-bound way to identify the specific agent that initiated the request.
4. ***Service Principal Barrier:*** Entra's OBO flow is primarily designed for user principals. If an agent authenticates using a service principal, it cannot use OBO to call another API while maintaining user context; it must fallback to a client-credentials grant, which completely loses the link to the originating user.

### Advantages of IBM Verify Token Exchange (RFC 8693)
IBM Verify provides a true Security Token Service (STS) that implements the full RFC 8693 specification, offering significantly more flexibility and security:   
1. ***Subject and Actor Separation:*** The exchange allows for a subject_token (representing the user) and an actor_token (representing the agent). The resulting token includes an act claim, ensuring total auditability of who acted for whom.   
2. ***The may_act Claim:*** IBM Verify supports the may_act claim, which can be embedded in the original user token to define exactly which agents or clients are authorized to perform an exchange on behalf of that user.   
3. ***Cross-Domain Trust:*** Tokens can be exchanged across different trust domains, allowing an agent to swap a token from one provider for a token issued by IBM Verify to call protected enterprise resources.   
4. ***Runtime Policy Evaluation:*** IBM Verify can evaluate an access policy as part of the token exchange flow, allowing for finer-grained authorization checks than a static scope mapping can provide.   


## Securing Interaction Models: User 2 Agent vs. Agent 2 Agent

Agentic security requires different strategies based on whether a human is the ultimate driver or if agents are collaborating autonomously.

User-to-Agent (U2A) Model
- In the U2A model (e.g., a "Copilot"), the primary goal is delegated authority.
- Granular Consent: IBM Verify uses RAR to show the user exactly what the agent will do (e.g., "Allow Agent to move file X to location Y").
- Decoupled Step-up: If an agent attempts a sensitive action, CIBA is used to "wake up" the user on their mobile device for a just-in-time approval, even if the agent is operating as a background task.

Agent-to-Agent (A2A) Model
- In the A2A model (e.g., an orchestration agent calling a sub-agent), the primary goal is cryptographic accountability.
- Interoperable Messaging: As agents adopt the Agent2Agent (A2A) protocol (emerging A2A specifications under Linux Foundation stewardship), IBM Verify’s support for Token Exchange ensures that identity is preserved as tasks pass between disparate agent frameworks.   
- Chain of Custody: By using RFC 8693, A2A interactions maintain a nested actor chain. A downstream service receiving a request from "Sub-Agent B" can trace the authority back to "Master Agent A" and finally to the human user, preventing anonymous lateral movement in the network.

---

## Shared Signals and Continuous Risk: The Antenna Framework

Agentic security requires a dynamic response to changing risk postures. IBM Verify Antenna is a lightweight, self-hosted container that enables the exchange of security events using the Shared Signals Framework (SSF).
As a Transmitter: Antenna ingests data from sources (audit streams, databases) and converts them into standardized Security Event Tokens (SETs) to broadcast to receivers.
As a Receiver: It consumes SETs (e.g., "session-revoked", "risk-level-changed") and converts them into immediate actions, such as revoking agent sessions when anomalous behavior is detected.
This allows for real-time revocation of a rogue agent’s access. Microsoft Entra ID’s Continuous Access Evaluation (CAE) is robust but primarily optimized for its own first-party services and Windows endpoints, lacking the modular flexibility of Antenna for cross-vendor environments.

---

# Extended IAM Controls and IBM-HashiCorp Synergy

Beyond protocol support, IBM provides a comprehensive suite of IAM controls and infrastructure security:

Identity Protection (VIP): Uses AI/ML to analyze network logs and telemetry, discovering "shadow" assets and directories that often harbor unmanaged agents.
Dynamic Credentials with Vault: With HashiCorp now an IBM company, HashiCorp Vault issues short-lived, just-in-time credentials for AI applications, removing the risk of hard-coded static secrets.
Vault Radar: Scans source code and configuration files for exposed secrets (cloud keys, API tokens) before an agent is deployed.

---

# Summary 

Entra ID is an excellent workforce identity platform, particularly for clients with primarily Microsoft technologies. Agentic AI introduces a different problem class — and introduces new control points that require other technology solutions to participate.  This expanded technology surface exposes limitations in Entra, which present themselves to clients as integration challenges, introducing friction and cost.  

IBM Verify provides a superior platform for agentic AI runtime security by adhering strictly to advanced standards like RAR, DPoP, and CIBA, and the SSF via the Antenna framework. Its support for RFC 8693 Token Exchange explicitly addresses the "accountability gap" and "token bloat" inherent in Microsoft’s proprietary OBO model. The addition of HashiCorp Vault into this fabric allows for a complete "secrets and identity" plane that Microsoft Entra ID—constrained by legacy proprietary flows—cannot match in complex, multi-vendor environments.

Customers should also take an open approach to solving this dilemma, by creating a loosely coupled, open standards based approach to agentic runtime security.  This will enable them to know that their workloads, bank accounts, customer systems have a traceability thread running through it that can address the yawning accountability gap these systems are creating.
