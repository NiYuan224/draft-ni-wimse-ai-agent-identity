---
title: "WIMSE Applicability for AI Agents"
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
    email:	mcr+ietf@sandelman.ca


normative:
  RFC2119:
  RFC8174:
informative:
  I-D.ietf-wimse-arch:


--- abstract

This document discusses WIMSE applicability to Agentic AI, so as to establish independent identities and credential management mechanisms for AI agents. It also discusses mechanisms for cryptographically binding an AI agent identity to an accountable user or organization.

--- middle

# Introduction

AI agents are autonomous software entities that receive an intent, process contextual information, and execute decisions at machine speed with minimal human intervention. Without appropriate guardrails, they may give rise to significant risks:

* Blurred Network Boundaries: AI agents may operate across systems and platforms, which expands the attack surface and amplifies security risks.
* Arbitrary and Unpredictable Access Patterns: AI agents may perform unexpected actions or access sensitive resources susceptible to malicious manipulation or logical errors.
* Lack of Accountability: Tracing an AI agent's actions is inherently difficult, making it difficult to identify the entity accountable for those actions.
* Context Rot: A gradual degradation of their ability to maintain relevant and coherent call contexts over time.

Therefore, for AI agents, the traditional perimeter-based security model has to transform into an identity-based security model, which is a prerequisite to implementing precise access control and ensuring security visibility.

To realize this goal, a mechanism should be designed considering the following requirements:

* Independent, Trustworthy Identities: AI agents should have independent and trustworthy identities and credentials, distinct from those of devices and users. This allows the AI agent to act as an independently identifiable workload while maintaining a verifiable relationship with the entity accountable for its operation.
* Automated Credential Management: An automated mechanism is necessary for managing credentials with reduced validity periods to minimize security exposure.
* Minimal Privileged Access Tokens: AI agents should have task-oriented, fine-grained access tokens with short validity periods.
* Explicit Workflows: AI agents need explicit workflow management in order to avoid random agentic access. The workflow could be long-term and static, or could be short-term and task-triggered, but the call context must always be visible and preserved.

This document discusses the possibility of using WIMSE architecture to provide AI agent identities and credentials. It accords with the original WIMSE use case in Section 3.4.1 Bootstrapping Workload Identifiers and Credentials of {{?I-D.ietf-wimse-arch}}. We also discuss requirements for extending the WIMSE architecture to bind an AI agent identity to the identity of an accountable user or organization.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

This document uses terms and concepts defined by the WIMSE architecture. For a complete glossary please refer to {{?I-D.ietf-wimse-arch}}:

* Trust Domain: A logical grouping of systems that share a common set of security controls and policies. Agent credentials are issued under the authority of a trust domain.
* AI Agent: The autonomous software entity that initiates the credential request. This document may refer to it as the "agent", but it is essentially the workload instead of the agent in the WIMSE architecture.
* Identity Server: A trusted entity issuing agent identities and credentials. For simplicity, this document may refer to this component as the "server".
* Identity Proxy: An intermediary component between an agent and the Identity Server. It exposes an Agent API locally to agents. For simplicity, this document may refer to this component as the "proxy".

In addition, this document introduces the following new terms:

* Owner: An entity (individual or organization) accountable for the operation of an agent and capable of providing a cryptographic signature to establish a verifiable relationship with the agent identity. The owner's approval may be provided through mechanisms such as manual confirmation, a hardware security module, or an automated policy engine based on pre-defined security policies.
* Dual-Identity Credential: A credential that contains the identifiers and associated public keys of both an agent and its owner. The credential is cryptographically bound to both entities.

# Architecture

## Bootstrapping AI Agent Identity and Credentials
   The server and the proxy are assumed to have established a secure channel.	A basic workflow is shown in Figure 1.


  1.	As an intermediary between the server and the agents, the proxy provides an agent API that agents can use to initiate identity credential requests. These requests include a public key and a signature as proof-of-possession to demonstrate control of the corresponding private key.
  2.	The proxy forwards these requests,  along with the attestation evidence for verifying the operational status of the agent, to the server for processing.
  3. The server validates the evidence received from the proxy, and issues the corresponding identity credentials.
  4.	Once issued, the proxy forwards the agent identity credentials to the agents.

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

During the request and issuance of identity credentials, the proxy should gather attestation evidence from the operating system and hardware to verify the operational status of the agent. This information is used by a RATS Verifier (could be the server) to support the identity server's decision on whether or not to issue an identity credential to an agent, for either a bootstrapping or a renewal request.


# Identity Binding Extensions for WIMSE
The basic WIMSE architecture ensures that a workload has a trusted identity that can be authenticated when connecting to other workloads. However, workload identity alone may not identify the user or organization accountable for an AI agent's operation. Consequently, an agent may require a credential that cryptographically binds its identity to the identity of such an accountable entity. This dual-identity credential provides a foundation for accountability and audit. This section describes the necessity of a dual-identity credential through two representative use cases and defines three operational models to issue it.

## Use Cases

### Network Access Control in Campus

Campus administrators require authentication of agents before granting access to an internal network via authentication protocols such as IEEE 802.1X (e.g., using EAP-TLS). In this scenario, the Authentication Server must verify the agent's credential and determine its network privileges. A single-identity credential only allows the Authentication Server to identify the agent itself. It does not directly provide information about the user or organization accountable for the agent. Without further information, Authentication Server may treat the agent as a guest, providing only limited public access or rejecting the request to protect sensitive zones.

A dual-identity credential allows the Authentication Server to verify both the agent's identity and the identity of the accountable user or organization associated with the agent. For instance, an agent associated with Alice from the R&D department can be identified as a trusted entity, and can receive proper access privileges according to existing IAM information (like RBAC). This enables the network to implement user-specific segmentation, such as automatically assigning the agent to the R&D VLAN rather than a guest network.

### Cross-organization Interaction

In collaborative enterprise environments, it is essential to ensure that an agent can be associated with an identifiable organization that is accountable for its operation. This requirement spans the entire credential lifecycle, from issuance to interaction.

* Issuance: When an agent requests an identity credential, the identity server may require organizational oversight. By requiring the accountable organization to approve the credential request, the server can establish a cryptographically verifiable relationship between the agent identity and the accountable organization before issuing the credential.

* Interaction: When an agent accesses another agent or a service across organizational boundaries, a dual-identity credential allows the receiving entity to identify both the agent and the organization accountable for it, providing stronger accountability and traceability for cross-organization interactions.

## Issuance Models
Identity binding can be integrated into the WIMSE workflow in several ways. The following three models differ in where the binding between the agent identity and the accountable owner is established. Before initiating the dual-identity issuance flow, a pre-established trust relationship must exist, where the identity server is provisioned with trust anchors (e.g., public keys, CA certificates, or hardware-backed credentials) to verify the owner’s signature. The mechanism by which these trust anchors are established, distributed, or updated is out of scope of this document.


### Agent-Mediated (Owner-Pre-Signed)
In this model, the owner acts as a local offline approver, countersigning the agent's request before it is submitted to the proxy. The identity binding phase, consisting of the following two steps, is added prior to the standard issuance flow defined in Figure 1:

a. The agent generates and signs an identity credential request and sends it to the owner.

b. The owner reviews and countersigns the request and returns the countersigned request to the agent.

The following steps are similar to the basic architecture, that is, the agent sends the countersigned request to the server via the proxy (steps 1 and 2), then the server verifies both signatures and issues the dual-identity credential to the agent via the proxy (steps 3 and 4).

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
*Figure 2: Agent-Mediated Model*

* Typical Application Scenarios: This model is ideal for local identity binding where the owner and agent operate on the same host. It supports asynchronous and offline owner confirmation, allowing the owner to use a local signing module (e.g., a hardware security key)  to countersign the agent's request. For instance, a developer may use a personal hardware security key to countersign an agent's request directly on their local machine. Therefore, this model supports hardware-based identity binding, cryptographically anchoring the agent’s dual-identity to the specific physical device managed by the owner.

* Attack Surface: The primary attack surface lies in the local environment,  focusing on two main risks. The first is human-agent trust exploitation: an attacked or rogue agent may manipulate human users to approve a malicious request. The second is the compromise of signing keys: if the owner’s local private keys are not protected by hardware (e.g., TPM), an attacker can forge confirmations.

### Owner-Mediated (Gateway Mode)
In this model, the owner acts as the supervisory gatekeeper between the proxy and the server. It inspects requests relayed by the proxy to ensure compliance with organizational policies, providing cryptographic binding only after approval.

Such a mechanism is integrated into the basic architecture as shown in Figure 3. First, the agent generates and signs an identity credential request and sends it to the proxy (step 1), then：

a. The proxy intercepts the request and relays it to the owner for administrative inspection.

b. The owner reviews and countersigns the request. It then forwards the countersigned request to the server.

c. The server verifies both signatures and issues the dual-identity credential back to the owner, who then dispatches it to the proxy.

Finally, the proxy sends the credential to the corresponding agent (step 4).

Figure 3 shows a one-to-one mapping case between the owner and the proxy. In this case, the function of the owner can be integrated directly into the proxy, collapsing the hierarchy into a single entity to simplify the deployment. Moreover, this model also supports a one-to-many topology, allowing a central owner to manage multiple proxies across various trust domains.

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
*Figure 3: Owner-Mediated Model*

* Typical Application Scenarios: This model is ideal for enterprise governance. Since the owner sits in the middle, it acts as a gateway to ensure that no request reaches the server unless it complies with enterprise security policies and compliance requirements. It is particularly suitable for hierarchical environments where the owner acts as a centralized gateway for multiple proxies.

* Attack Surface: The owner becomes a high-value target and a single point of failure. If it is compromised, an attacker can forge approvals for any agent across the managed proxies. Furthermore, as a centralized gateway, the owner is vulnerable to Denial-of-Service attacks. It is essential to implement rate limiting and request queuing.

### Server-Mediated (Challenge-Response)
In this model, the owner acts as an independent verifier. The server orchestrates the binding phase by contacting the owner as a separate step in the issuance logic, decoupling the binding from the agent's request.

This mechanism is integrated into the basic architecture as shown in Figure 4. First, the agent sends an identity credential request to the server via the proxy (steps 1 and 2), then:

a. The server pauses the process and sends a binding challenge to the owner accountable for the agent's operation. This initiates the owner confirmation flow.

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
*Figure 4: Server-Mediated Model*

* Typical Application Scenarios: This model is suitable for scenarios requiring independent and real-time confirmation from an owner who is not involved in the initial request path. For instance, an agent is hosted by a third-party service provider, while the organization providing the agent remains accountable for its operation. When the agent requests an identity credential, the server initiates an out-of-band verification directly with the administrative center.

* Attack Surface: The primary risk lies in the out-of-band channel. Without nonces or mutual authentication, an attacker could impersonate the owner or replay a previous confirmation response.

## Hardware Root of Trust
The owner can leverage the hardware root of trust to generate cryptographic signatures, thus binding an agent to a specific hardware device. Consequently, the above issuance models allow multiple virtual agent identities to be derived from a single hardware root of trust.




# Security Considerations

TODO Security

# IANA Considerations

This document has no IANA actions.

Document History

* Since Draft 01

- Added three identity binding models.

* Since Draft 02

- Fixed editorial issues, and streamlined the content.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
