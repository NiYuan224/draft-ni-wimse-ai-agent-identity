---
title: "Identity Endorsements for AI Agents"
abbrev: "wimse-ai-agent"
category: info

docname: draft-ni-wimse-ai-agent-identity-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "Workload Identity in Multi System Environments"
keyword:
 - WIMSE
 - AI Agent
 - Identity
venue:
  group: "Workload Identity in Multi System Environments"
  type: "Working Group"
  mail: "wimse@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/wimse/"
  github:
  latest:

author:
 -
    fullname: Yuan Ni
    organization: Huawei
    email: niyuan1@huawei.com
 -
    fullname: Chunchi Peter Liu
    organization: Huawei
    email: liuchunchi@huawei.com
 -
    fullname: Michael Richardson
    organization: Sandelman Software Works
    email: mcr+ietf@sandelman.ca


normative:
  RFC9334:
  RFC8995:
  RFC7030:
  RFC2119:
  RFC8174:
informative:
  I-D.ietf-wimse-aims:
  I-D.ietf-rats-endorsements:
  I-D.ietf-wimse-arch:
  I-D.ietf-wimse-workload-creds:



--- abstract

AI Agents are workloads that use Identity Credentials to represent their identity and authenticate to other workloads. However, an Agent Identity Credential does not by itself establish who is accountable for the Agent's operation. Such accountability may be required for auditing and compliance.

This document describes an identity endorsement mechanism for AI Agents. During credential provisioning, one or more Endorsers may provide Identity Endorsements that vouch for their relationship with an Agent identity. The Identity Server verifies the endorsements and issues an agent Identity Credential that cryptographically binds the Agent identity to its endorsers.

--- middle

# Introduction

AI Agents are workloads that use credentials to represent their identity and authenticate to other workloads {{I-D.ietf-wimse-aims}}. However, an Agent is not in a position to self-assert which individual or organization is accountable for its operation. Such information therefore needs to be asserted by a trusted party other than the Agent.

In the RATS architecture, an Endorser is a role distinct from the Attester, capable of providing claims regarding properties that the Attester cannot self-assert {{I-D.ietf-rats-endorsements}}. Building on this concept, this document applies endorsement to Agent identity, allowing an Endorser to vouch for its relationship with an Agent.

This need is already reflected in emerging Agent identity management systems. Microsoft Entra Agent ID requires each Agent identity to have a Sponsor, who is accountable for the Agent's purpose, lifecycle decisions, and access reviews. Okta for AI Agents similarly requires AI agents to be registered with clear human ownership to strengthen accountability, governance, and compliance. These examples demonstrate a common need to associate an Agent identity with the principal accountable for it.

This document introduces identity endorsement for AI agents. During credential provisioning, one or more Endorsers may provide cryptographically verifiable endorsements for an Agent identity. The Identity Server verifies the endorsements and issues an agent Identity Credential that cryptographically binds the Agent identity to its endorsers.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

This document uses terms and concepts defined in the WIMSE Architecture {{I-D.ietf-wimse-arch}}, WIMSE Workload Credentials {{I-D.ietf-wimse-workload-creds}}, and AIMS {{I-D.ietf-wimse-aims}}, including Trust Domain, Identity Server, Identity Credential, Identity Proxy, and AI Agent.

The following terms extend the Endorser and Endorsement concepts defined in the RATS architecture {{RFC9334}}:

* Endorser: A role performed by an entity (typically an individual or organization accountable for an Agent) whose Identity Endorsements may help Identity Servers verify the relationship between the Agent identity and the principal accountable for the Agent. Depending on the deployment, an Endorser may be an Agent user, an Agent service provider, or an Agent developer.

* Identity Endorsement: A secure statement that an Endorser vouches for its relationship with an Agent identity and its accountability for the Agent's operation.

# Architecture

## Baseline AI Agent Identity Credential Provisioning

The Server and the Proxy are assumed to have established a secure channel. A basic workflow is shown in Figure 1.


  1.	As an intermediary between the Server and the Agents, the Proxy provides an Agent API that Agents can use to initiate Identity Credential requests. These requests include a public key and a signature as proof-of-possession to demonstrate control of the corresponding private key.
  2.	The Proxy forwards these requests, along with the attestation evidence for verifying the operational status of the Agent, to the Server for processing.
  3. The Server validates the evidence received from the Proxy, and issues the corresponding Identity Credentials.
  4.	Once issued, the Proxy forwards the Agent Identity Credentials.

~~~~

  +----------------------------+
  |       Identity Server      |
  +-------------- ^ + ---------+
      (2)identity | |(3)identity
      credential  | | credential
      request &   | |
      evidence    | |
+-----------------+-+-----------------------------------+
|  Trust Domain   | |            (1)identity            |
|                 | |            credential             |
| +--------------++ v ---------+ request      +-------+ |
| |              |             <--------------+       | |
| |Identity Proxy|  Agent API  +--------------> Agent | |
| |              |             | (4)identity  |       | |
| +--------------+----- ^ -----+ credential   +-------+ |
|              evidence |                               |
+-----------------------+-------------------------------+
|          Hosting Operating Systems and Hardware       |
+-------------------------------------------------------+

~~~~
*Figure 1: Basic Architecture and the Workflow*



## Attestation

During the request and issuance of identity credentials, the proxy should gather attestation evidence from the operating system and hardware to verify the operational status of the agent. This information is used by a RATS Verifier (could be the server) to support identity server's decision of whether or not to issue the identity credential of an agent, whether it is a bootstrapping or a renewal request. The structure and claims of this evidence may refer to the Entity Attestation Token (EAT) profile for Autonomous AI Agents, which defines specialized claims for AI agent integrity, training provenance, and runtime authorization.


# Identity Binding Extensions for WIMSE
The basic WIMSE architecture ensures a trusted entity owns a trusted workload identity that is secure to connect to. However, the agent often operates on behalf of a human user or an organization. Consequently, an agent requires a credential that cryptographically binds its identity to its owner’s identity. This dual-identity credential provides a necessary foundation for access control and accountability. This section describes the necessity of dual-identity credential through two representative use cases and defines three operational models to issue it.

## Use Cases

### Network Access Control in Campus

Campus administrators require authentication of agents before granting access to internal network via authentication protocols such as IEEE 802.1X (e.g., using EAP-TLS). In this scenario, the Authentication Server must verify the agent's credential and determine its network privileges. A single-identity credential only allows the Authentication Server to identify the agent itself. Without further information, Authentication Server may treat the agent as a guest, providing only limited public access or rejecting the request to protect sensitive zones. A dual-identity credential allows the Authentication Server to verify both the agent's identity and the specific user or department it represents. For instance, an agent representing Alice from the R&D department can be identified as a trusted entity, and can receive proper access privileges according to existing IAM information (like RBAC). This enables the network to implement user-specific segmentation, such as automatically assigning the agent to the R&D VLAN rather than a guest network.

### Cross-organization Interaction

In collaborative enterprise environments, it is essential to ensure that any agent requesting services is explicitly approved by its organization. This requirement spans the entire credential lifecycle, from issuance to interaction.

* Issuance: When an agent requests an identity credential, the identity server may require organizational oversight. By binding the agent's identity credential request to its corresponding organization, the server can verify the organizational approval before issuing the credential. In other words, dual-identity credential could be a manifestation of Human-in-the-Loop (HITL) mechanisms.
* Interaction: When an agent accesses another agent or a service across organizational boundaries, authentication is necessary to ensure the request is from a valid entity, as illustrated in the A2A protocol and WIMSE architecture{{I-D.ietf-wimse-arch}}. A dual-identity credential carries an organizational approval, which provides a strong basis for trust, ensuring both accountability and traceability for cross-organization interactions.

## Issuance Models
Identity binding can be integrated into the WIMSE workflow in several ways. we introduce the following three models according to the mediation point where the agent's identity and organizational authority are cryptographically bound. Before initiating the dual-identity issuance flow, a pre-established trust relationship must exist, where the identity server is provisioned with trust anchors (e.g., public keys, CA certificates, or hardware-backed credentials) to verify the owner’s signature. The mechanism by which these trust anchors are established, distributed, or updated is out of scope of this document.


### Agent-Mediated (Owner-Pre-Signed)
In this model, the owner acts as a local offline approvar, which provides a signature on the agent's request before it is submitted to the proxy. The identity binding phase, consisting of the following two steps, is added prior to the standard issuance flow defined in Figure 1:

a. The agent generates an identity credential request and sends it to the owner.

b. The owner signs the request and returns the signature to the agent. The agent then combines the original request and the owner's signature into a new composite request.

The following steps are similar to the basic architecture, that is, the agent send the new composite request to the server via the proxy (steps 1 and 2), then the server verifies the request and issues the dual-identity credential to the agent via the proxy (steps 3 and 4).

~~~~
  +----------------------------+
  |       Identity Server      |
  +-------------- ^ + ---------+
               (2)| |(3)
+-----------------+-+---------------------------------+
| Trust Domain    | |              +-------+          |
|                 | |              | owner |          |
|                 | |              +--^-+--+          |
|                 | |       (a)request| |(b)signature |
| +--------------++ v -------+ (1) +--+-v--+          |
| |              |           ------+       |          |
| |Identity Proxy| Agent API ------> Agent |          |
| |              |           | (4) |       |          |
| +--------------+-----------+     +-------+          |
+-----------------------------------------------------+
~~~~
*Figure 2: Agent-Mediate Model*

* Typical Application Scenarios: This model is ideal for local identity binding where the owner and agent operate on the same host . It supports asynchronous and offline owner confirmation, allowing the owner use local signing module (e.g., a hardware security key)  to pre-sign the agent's request. For instance, a developer using a personal hardware security key to sign an agent's request directly on their local machine. Therefore, this model obviously supports hardware-based identity binding, cryptographically anchoring the agent’s dual-identity to the specific physical device managed by the owner.

* Attack Surface: The primary attack surface lies at the local environment,  focusing on two main risks. First is human-agent trust exploitation: an attacked or rogue agent may manipulate human users to approve malicious request. Second is the compromise of signing keys: if the owner’s local private keys are not protected by hardware (e.g., TPM), an attacker can forge confirmations.

### Owner-Mediated (Gateway Mode)
In this model, the owner acts as the supervisory gatekeeper between the proxy and the server. It inspects requests relayed by the proxy to ensure compliance with organizational policies, providing cryptographic binding only after approval.

Such a mechanism is intergrated in the basic architecture as shown in Figure 3. Firstly, the agent generates an identity credential request and sends it to the proxy(step 1), then:

a. The proxy intercepts the request and relays it to the owner for administrative inspection.

b. The owner reviews and signs the request. It then combines the original request and the owner's signature into a new composite request to be submitted to the server, along with additional organizational materials, such as an oragnizational credential.

c. The server validates the received information and issues the dual-identity credential back to the owner, who then dispatches it to the proxy.

Finally, the proxy send the credential to the corresponding agents(step 4).

Figure 3 shows a one-to-one mapping case between the owner and the proxy. In this case, the function of owner can be integrated directly into the proxy, collapsing the hierarchy into a single entity to simplify the deployment. Moreover, this model also supports a one-to-many topology, allowing a central owner to manage multiple proxies across various trust domains.

~~~~
  +----------------------------+
  |      Identity Server       |
  +------------------^-+-------+
      (b)request     | | (c)dual-identity
      (signature)    | |  credential
  +------------------+-v-------+
  |           Owner            |
  +------------------^-+-------+
      (a)request     | | (c)
+--------------------+-+--------------------------------+
| Trust Domain       | |                                |
| +--------------+---+-v-------+     (1)      +-------+ |
| |              |             <---------------       | |
| |Identity Proxy|  Agent API  |--------------> Agent | |
| |              |             |     (4)      |       | |
| +--------------+-------------+              +-------+ |
+-------------------------------------------------------+
~~~~
*Figure 3: Owner-Mediate Model*

* Typical Application Scenarios: This model is ideal for enterprise governance. Since the owner sits in the middle, acting like a middlebox to ensure no request reaches server unless it complies with enterprise security policies and compliance requirements. It is particularly suitable for hierarchical environments where the owner acts as a centralized gateway for multiple proxies.

* Attack Surface: The owner becomes a high-value target and a single point of failure. If it is compromised, an attacker can forge approvals for any agent across the managed proxies. To mitigate this, mutual authentication and cryptographic integrity are mandatory between proxies and the owner. Furthermore, as a centralized gateway, the owner is vulnerable to Denial-of-Service attacks. It is essential to implement rate limiting and request queuing.

### Server-Mediated (Challenge-Response)
In this model, the owner acts as an independent verifier. The server orchestrates the binding phase by contacting the owner as a separate step in the issuance logic, decoupling the binding from the agent's request.

This mechanism is integrated into the basic architecture as shown in Figure 4. First, the agent sends an identity credential request to the server via the proxy (steps 1 and 2), then:

a. The server pauses the process and sends a binding challenge to the owner on whose behalf the requesting agent acts. This initiates the owner confirmation flow.

b. The owner reviews the challenge, signs it, and returns the response to the server.

After that, the server validates the owner's response, completes the identity binding, and issues the dual-identity credential to the agent via the proxy (steps 3 and 4).

~~~~
  +----------------------------+(a)challenge  +---- --+
  |                            +-------------->       |
  |       Identity Server      <--------------+ Owner |
  |                            |(b)signature  |       |
  +-------------- ^ +----------+              +---- --+
               (2)| |(3)
+-----------------+-+-----------------------------------+
| Trust Domain    | |                                   |
| +---------------+-v----------+     (1)      +-------+ |
| |              |             |--------------+       | |
| |Identity Proxy|  Agent API  |--------------> Agent | |
| |              |             |     (4)      |       | |
| +----------------------------+              +-------+ |
+-------------------------------------------------------+
~~~~
*Figure 4: Server-Mediate Model*

* Typical Application Scenarios: This model is suitable for scenarios requiring independent and real-time confirmation from an owner who is not involved in the initial request path. For instance, An agent is deployed by a service provider on behalf of an organization. When the agent requests an identity credential, the server initiates an out-of-band verification directly to the administrative center.

* Attack Surface: The primary risk lies in the out-of-band channel. Without nonces or mutual authentication, an attacker could intercept this channel to perform response forgery or replay attacks.

## Hardware Root of Trust
The owner can leverage the hardware root of trust to generate cryptographic signatures, thus binding an agent to a specific hardware device. Consequently, the above issuance models allow multiple virtual agent identities to be derived from a single hardware root of trust.

# Comparison with CHEQ
While both this document and CHEQ introduce a human element to enhance security,  their goals and the underlying mechanisms are different.

CHEQ focuses primarily on controlling the actions of AI agents. It requires user double confirmation when an AI Agent invokes an OAuth access token request, preventing possible deviation from user expectations.

The purpose of this document is to provide distinct identity and credentials to AI agents, whether or not it is bound to an owner user of device's parent identity. Whether or not the agent inherits access permission privileges from its user is out of scope of this document.

# Initial Trust Establishment

AI agents may operate in cloud or campus. In the cloud, the initial trust establishment between the proxy and the server has already been solved by solutions like SPIRE.  However, in campus scenarios,  the heterogeneity and limited manageability of devices make credential provisioning challenging, complicating initial trust establishment.

BRSKI provides a feasible method by introducing a cryptographically signed artifact called “voucher”.

In the BRSKI flow, the proxy (acting as a BRSKI pledge) discovers the server (acting as a BRSKI registrar), initiates a TLS handshake, and sends a voucher request including its immutable manufacturer credential—the IDevID (Initial Device Identifier). The server uses this IDevID to contact the manufacturer's service (MASA). After validating the request, the MASA issues a signed voucher.

The proxy then verifies the manufacturer's signature on the voucher, which securely transferring trust from the manufacturer to the local domain. This verified trust is a prerequisite for the server to issue a local domain device certificate (LDevID). This certificate enrollment step essentially follows the standard EST mechanism.

However, it should be noted that BRSKI is not necessarily the only way to achieve this secure integration. The core goal is to bridge the initial trust gap. If the proxy is pre-configured with the target server's public key or certificate and can securely locate it, the standard EST protocol alone may be sufficient to establish trust and obtain the LDevID certificate.

**Open Question:** What are the precise conditions and mechanisms for determining the use of various bootstrap methods (including but not limited to BRSKI and EST)?



# Security Considerations

TODO Security

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
