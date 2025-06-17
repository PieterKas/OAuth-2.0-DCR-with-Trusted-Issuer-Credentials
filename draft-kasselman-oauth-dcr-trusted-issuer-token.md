---
title: "OAuth 2.0 Dynamic Client Registration with Trusted Issuer Credentials"
abbrev: "DCR with Trusted Issuer Credentials"
category: info

docname: draft-kasselman-oauth-dcr-trusted-issuer-token-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Web Authorization Protocol"
keyword:
 - dynamic client registration
 - SPIFFE
 - platform credentials
venue:
  group: "Web Authorization Protocol"
  type: "Working Group"
  mail: "oauth@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/oauth/"
  github: "PieterKas/OAuth-2.0-DCR-with-Trusted-Issuer-Credentials"
  latest: "https://PieterKas.github.io/OAuth-2.0-DCR-with-Trusted-Issuer-Credentials/draft-kasselman-oauth-dcr-trusted-issuer-token.html"

author:
 -
    fullname: "Pieter Kasselman"
    organization: SPIRL
    email: "pieter@spirl.com"
    fullname: "Ismael Kazzouzi"
    organization: SPIRL
    email: "ismael@spirl.com"

normative:
 RFC7519:
 RFC7591: # OAuth 2.0 Dynamic Client Registration Protocol
 RFC6749: # The OAuth 2.0 Authorization Framework
 SPIFFE:
   title: SPIFFE
   target: https://github.com/spiffe/spiffe/blob/main/standards/SPIFFE.md
 SPIFFE_X509:
    title: X509-SVID
    target: https://github.com/spiffe/spiffe/blob/main/standards/X509-SVID.md
 SPIFFE_JWT:
    title: JWT-SVID
    target: https://github.com/spiffe/spiffe/blob/main/standards/JWT-SVID.md
 SPIFFE_ID:
    title: SPIFFE-ID
    target: https://github.com/spiffe/spiffe/blob/main/standards/SPIFFE-ID.md
 VC-JWT:
    title: "Securing Verifiable Credentials using JSON Web Tokens"
    author:
       - name: Brent Zundel
       - name: Orie Steele
       - name: Kristina Yasuda
    date: 2023-06-14
    status: "W3C Working Draft"
    target: https://www.w3.org/TR/vc-jwt/
 SPIFFE-OAUTH-CLIENT-AUTH:
    title: OAuth SPIFFE Client Authentication
    target: foo
 MCP:
  title: "Model Context Protocol"
  traget: "https://modelcontextprotocol.io/specification/"

informative:

...

--- abstract

An OAuth 2.0 client requires specific information to interact with an authorization server, including a client identifier issued by that server. The OAuth 2.0 Dynamic Client Registration Protocol {{RFC7591}} defines a mechansims for dynamic client registration that removes the need for manual registration. This provides a more scalable mechanism that can be used by clients that do not have a pre-existing relationship with an authorization server, or where manually configuring such relationships is prohibitive from a cost and scale perspective. Examples of deployments that benefit from dynamic client registration includes modern cloud architectures where microservices are created on demand to meet scale requirements or for use with emerging protocols like the Model Context Protocol (MCP) {{MCP}} which requires clients to register with an authorization server even if they do not have a predefined relationship with the authorization server. Similar to modern cloud native workloads, MCP service integrations may be ephemeral in nature. The OAuth 2.0 Dynamic Client Registration Protocol {{RFC7591}} defines a software statement that includes metadata about the client and is signed by a developer or trusted third party. This specification describes the use of specific third party credentials issued to workloads and applications as software statements. The specification describes two types of credentials that may be used as software statements. The first is Secure Production Identity Framework For Everyone (SPIFFE) credentials. The second is the use of JWT representation of a Verifiable Credentials {{VC-JWT}}

--- middle

# Introduction
The OAuth framework {{RFC6749}} is a widely deployed authorization protocol standard that enables applications to obtain limited access to user resources. OAuth clients must be registered with the OAuth authorization server, which poses significant operational challenges in dynamically scaling environments. Manual registration and subsequent lifecycle management is common, but is costly, difficult to scale and hard to maintain throughout the lifecycle of a client. These challenges are exacerbated by the increasingly dynamic nature of modern cloud native workloads that are instantiated as needed. In adddition, emeging protocols like Model Context Prototol (MCP) requires clients to register with an authorization server, but these clients may not always have a predefined relationship with the authorization server, and similar to modern cloud native workloads, may be ephemeral in nature.

{{RFC7591}} defines a mechansims for dynamic client registration that removes the need for manual registration. This provides a more scalable mechanism that can be used by clients that do not have a pre-existing relationship with an authorization server, or where manually configuring such relationships is prohibitive from a cost and scale perspective. It defines the concept of a software statement, which is a JSON Web Token (JWT) {{RFC7519}} that asserts metadata values about the client that is signed by a developer or trusted third party.

This specification describes the use of SPIFFE JWT-SVIDs and Verifiable Credeentials as software statements that can be submitted as software statements when using OAuth 2.0 Dynamic Client Registration Protocol {{RFC7591}}.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Overview
This specification describes the use of two credential types as software statements, namely SPIFFE credentials and Verifiable Credentials.

## Secure Production Identity Framework For Everyone (SPIFFE)
The Secure Production Identity Framework For Everyone (SPIFFE) is a graduated Cloud Native Compute Foundation project designed to dynamically attest and verify workload identity (see {{SPIFFE}}. SPIFFE is commonly deployed to support large scale deployment of workloads in enterprise and cloud native compute environments. It ensures that every workload is attested and issued with X.509-SVID {{SPIFFE_X509}} and JWT-SVID {{SPIFFE_JWT}} credentials. JWT-SVIDs contains a minimal set of claims, but may be extended to include additional claims, including those defined in the Dynamic Client Registration specification (see Section 2 of {{RFC7591}}). A SPIFFE issuer may attest a workload, add additional metadata based on the attestation infromation and issue a JWT-SVID. Workloads that are provisioned with JWT-SVIDs presents these credentials as software statements when using Dynamic Client Registration. The registration endpoint is configured to trust the SPIFFE issuer and can verify the JWT-SVID, as well as retrieve additional claims that represents metadata attested to by the SPIFFE issuer. Upon succesfull validation of the software statement, the registration endpoint completes the client registration. When SPIFFE JWT-SVIDs are used, the authorization server MUST NOT not provision additional client secrets, as the workload can use it's JWT-SVID or X.509 SVID to authenticate when initiating any OAuth flow. This removes the risk of additional secrets proliferation, reduces the overheads of seuring and managing secrets and improves the overall risk posture.

## Verifiable Credentials (VC)
A Verifiable Credential (VC) is a tamper-evident digital assertion made by an issuer about a subject, which can be cryptographically verified by a third party. VCs follow an issuer–holder–verifier model where the issuer creates and signs the credential, the holder stores and presents it, and the verifier checks its authenticity and validity. VCs can be represented as a JSON Web Token (JWT) {{VC-JWT}}. A VC encoded as a JWT can serve as a software statement in dynamic client registration by including metadata as defined in Section 2 of {{RFC7591}} about the client as additional claims in the VC. Workloads that are provisioned with VCs presents these credentials as software statements when using Dynamic Client Registration. The registration endpoint is configured to trust the VC issuer and can verify the VC and use the additional metadata claims included by the issuer. Upon succesfull validation of the software statement, the registration endpoint completes the client registration. The use of Verifiable Credentials for client authentication is currently undefined and it is up to the authorization server to make a risk based decision on which authentication methods to use.

## SPIFFE and VC Deployment Models
SPIFFE and VC address different deployment models. SPIFFE is more commonly deployed within large enterprise compute environments and provides high levels of assurance about a workload identity (in this case an OAuth client) along with a highly robust, low latency provsioning pipeline with built in credential lifecycel management to maximise availability. It is commonly deployed as an alternative to managing secrets.

VCs provide an option for clients that are deployed outside an enterprise where SPIFFE is not deployed. These clients use VC issuance protocols to obtain VCs. The use of these protocols or any attstation that needs to be provided is out of scope for this specification.

## Protocol Flow

    +-------------+ (A) Client Registration Endpoint Establishes
    |   SPIFFE    |     Trust with SPIFFE or VC Issuer
    |     or      |
    |     VC      |
    |   Issuer    |<------------------------------------------+
    +-------------+                                           |
           ^                                                  |
           |                                                  |
           |                                                  |
          (B) SPIFFE JWT-SVID or VC provisioned to client     |
           |                                                  |
           v                                                  V
    +-----------+                                      +---------------+
    | Client or |--(C)- Client Registration Request -->|    Client     |
    | Developer |       with SPIFFE JWT-SVID or VC     |  Registration |
    |           |         as software statement        |    Endpoint   |
    |           |                                      |               |
    |           |<-(D)- Client Information Response ---|               |
    |           |        or Client Error Response      +---------------+
    +-----------+

   Figure 1: Abstracted Dynamic Client Registration Flow with Software Statement

The OAuth 2.0 Client Dynamic Registration flow illustrated in Figure 1 describes the interaction between the SPIFFE or VC issuers, the client and the client registration endpoint defined in {{RFC7591}}. It includes the following steps:

* (A) The client registration endpoint establishes a trust relationship with the SPIFFE or VC Issuer. Once trust is established, the client registration endpoint can verify credentials issued by the SPIFFE or VC Issuer when presented as software statements.
* (B) The client is attested and credentialed by the SPIFFE Issuer {{SPIFFE}}, after vwhich it assigns a SPIFFE ID {{SPIFFE_ID}}, along with SPIFFE credentials (e.g. JWT-SVID {{SPIFFE_JWT}} and X.509-SVID {{SPIFFE_X509}}). Alternatively a VC issuer issues and provisions a VC. Issuers may include additional claims containing metadata (e.g the request URI).
* (C) The client calls the client registration endpoint with the client's desired registration metadata, and the software statement (either a SPIFFE JWT-SVID or a VC).
* (D) The client registration endpoint verifies the SPIFFE JWT-SVID or VC that was presented as a sotware statement. It extracts any additional claims that should be used as metadata before registering the client. It may return additional metadata, client identifiers and optionally credentials if a VC is used. Clients with SPIFFE credentials MUST use the SPIFFE credentials when authenticating as part of an OAuth flow.

# Issuer and Client Registration Endpoint Trust Relationship

## SPIFFE Issuer and Client Registration Endpoint Trust Relationship
SPIFFE makes provision for multiple Trust Domains, which are represented in the workload identifier. Trust Domains offers additional segmentation withing a SPIFFE deployment and each Trust Domain has its own keys for signing credentials. The OAuth authorization server may choose to trust one or more trust domains as defined in {{SPIFFE-OAUTH-CLIENT-AUTH}}.

## Verifiable Credentials and Client Registration Endpoint Trust Relationship
TODO - Describe how trust relationship is established between VC Issuer and client registration endpoint.

# Client Registration Endpoint Processing
TODO processing rules for the client registration endpoint.

## SPIFFE JWT-SVIDs
TODO SPIFFE specific processing

## VCs
TODO VC specific processing

# Security Considerations

## Client Secrets
Dynamic client registration is designed to increase the ease with which clients are registered in large scale deployments. The increased use of client secrets amplifies the risks of secret theft and information compromise. This risk is further amplified is those secrets are long lived or infrequently rotated. Clients that are provisioned with SPIFFE JWT-SVIDs or X.509-SVIDs, and use this specification MUST use them not only as software statements, but also when authenticating to the authorization server in subsequent OAuth flows.

The use of VCs as client authentication mechansims in OAuth is undefine. Consequently, when using VCs as software statements, the authorization server MAY provision additional credentials, but SHOULD avoid provisioning client secrets to limit the risks of secret proliferation and the consequences of secret theft.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
