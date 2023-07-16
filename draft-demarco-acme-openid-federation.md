---
title: "Automatic Certificate Management Environment (ACME) with OpenID Connect Federation 1.0"
abbrev: "ACME OIDC Federation"
category: std

docname: draft-demarco-acme-openid-federation-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Automated Certificate Management Environment"
keyword:
 - OpenID Connect Federation
 - ACME
venue:
  group: "Automated Certificate Management Environment"
  type: "Working Group"
  mail: "acme@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/acme/"
  github: "peppelinux/draft-demarco-acme-openid-federation"

author:
 -
    fullname: Giuseppe De Marco
    organization: independent
    email: demarcog83@gmail.com

normative:
  RFC2818: RFC2818
  RFC2986: RFC2986
  RFC3696: RFC3696
  RFC8555: RFC8555

  OIDC-FED:
    title: "OpenID Connect Federation 1.0"
    author:
      -
        ins: R. Hedberg
        name: Roland Hedberg
      -
        ins: M.B. Jones
        name: Michael Jones
      -
        ins: A.Å. Solberg
        name: Andreas Solberg
      -
        ins: J. Bradley
        name: John Bradley
      -
        ins: G. De Marco
        name: Giuseppe De Marco
      -
        ins: V. Dzhuvinov
        name: Vladimir Dzhuvinov

informative:


--- abstract


The Automatic Certificate Management Environment (ACME) is the standard [RFC8555] that allows obtaining certificates for websites (HTTPS [RFC2818]) by verifing the "fully-qualified" domain names [RFC3696] and the web servers within these.

OpenID Connect Federation 1.0 [OIDC-FED] is the standard that allows building multilateral federations through a trust evaluation mechanism attesting the possession of public keys, signature capabilities, protocol specific metadata and several administrative and tecnical information in the form of trust marks, related to a specific entity belonging to an organization.

This document defines how the X.509 certificates can be issued by a trust anchor and its intermediates through the ACME protocol to all the organizations that are part of a federation built on top of OpenID Connect Federation 1.0.

--- middle

# Introduction

OpenID Connect Federation 1.0 allows an ACME server to issue X.509 certificates to one or more than a single organization without having pre-established any direct relationship or any stipulation of a contract. In a multilateral federation, composed by thousands of entities belonging to different organizations, all the participants adhere to the same regulation or trust framework. OpenID Connect Federation 1.0 allows each participant to recognize the other participant using a trust evaluation mechanism, with RESTful services and cryptographic materials.

Considering that a requestor is an entity requesting the issuance of a X.509 certificate to a server and the issuer is the ACME server that validates the entitlements of the requestor before issuing the X.509 certificate, this specification defines how ACME and OpenID Connect Federation 1.0 can be integrated allowing an efficient issuance of X.509 to a requestor, reducing both the bureaucratic and the implementative costs, since:

- It does not require the involvement of other resources than the ACME `newNonce` and `newOrder` endpoints, since the authentication and authorization of the requestor is asserted with OpenID Federation 1.0.
- Instead of the `/.well-known/acme-challenge/{token}` endpoint it defines how to use and validate a basic OpenID Connect Federation component, called Entity Configuration, that is a signed JWT published in a well-known resource (`/.well-known/openid-federation`).
- It removes the requirement for the authentication of an entity and the provisioning of the *acme-challenge token*, since the authorization mechanisms is built on top of the trust evaluation model as defined in OpenID Connect Federation 1.0.
- It extends the ACME `newOrder` endpoint, defining a new payload identifier type called `openid-federation`.
- It defines how the OpenID Federation Entity Statements can be used for the publication of the X.509 certificates, by a trust anchor or intermediate, that was previously issued with ACME.

# Audience Target and Use Cases

The audience of the document are the multilateral federations that require automatic issuance of X.509 certificates using an infrastructure of trust based on OpenID Connect Federation 1.0.

This specification can be implemented by:

- Federation Entities that joins to a federation staging area using HTTP only transport to attests themselves as trustworhty, and then ask X.509 certificates for their official HTTPs Federation Entity ID.
- Federation Entities that want to ask and obtain X.509 certificate for every Federation Key contained in their Entity Configuration, as made recognizable in a Federation Trust Chain.


# Terminology

   **ACME**,    Automated Certificate Management Environment, a certificate management protocol [RFC8555].

   **TA**,      OpenID Connect Federation Trust Anchor, see CA

   **CA**,      Certification Authority, also known as Trust Anchor or Intermediate, specifically one that implements the ACME protocol though an ACME server.

   **CSR**,     Certificate Signing Request, specifically a PKCS#10 [RFC2986] as supported by ACME.

   **FQDN**,    Fully Qualified Domain Name.

   **Requestor**, Federation Entity that requests a X.509 certificate to a CA.

   **Issuer**,  Trust Anchor or Intermediate, a CA.

The terms "Federation Entity", "Trust Anchor", "Intermediate", "Entity Configuration", "Entity Statement", "Trust Mark" and "Trust Chain" used in this document are defined in the Section 1.2 of [OIDC-FED].

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Protocol Flow

This section presents the protocol flow.

## Discovery Preconditions

The protocol assumes the following discovery preconditions are met, where for discovery is intended the phase where a Requestor searches an Issuer to requests an X.509 certificate.

1. The Requestor and the Issuer MUST publish their Entity Configuration as defined in the Section 6 of [OIDC-FED].
2. The Requestor and the Issuer MUST be able to establish the trust to each other obtaining the Trust Chain of each other, as defined in the Section 3.2 of [OIDC-FED].
3. The Trust Anchor and its Intermediates SHOULD implement an ACME server with at least the `newNonce` and`newOrder` endpoints as extended in this specification.
4. The Issuer MUST publish in its Entity Configuration, within the metadata parameter as defined in the Section 4 of [OIDC-FED], the metadata type `acme_provider` according to the Section ... of this specification, **TBD**.
5. The Issuer MAY be a Leaf, in these cases a specific Trust Mark SHOULD be issued by the Trust Anchor, or on behalf of it through by the allowed Trust Mark issuers configured in the federation, and then published within the Leaf Entity Configuration.

Where the precondition number 4 and number 5 are not met, there MAY be some cases where the Requestor known a priori which are the Issuers in one or more federations, in this case the requestor directly requests the issuance of the X.509 certificate to the trusted Issuer.

## Overview

TBD: high level design and ascii sequence diagram.

1. The Requestor looks to its superiors entities, Trust Anchor or Intermediates, if they support the ACME protocol for OpenID Connect Federation 1.0. If not, the Requestor starts the discovery process to find which are the Issuers within the federation.
2. The Requestor obtains a new nonce from the Issuer, by senting a HTTP HEAD request to the `newNonce` resource of the Issuer;
3. The Issuer evaluiate the trust to the Requestor, by checking if it is part of the federation. If not the request MUST be rejected (**TBD** the error to return). There are two ways the Issuer has to check if a Requestor is part of the federation, these are the followings:
  - The Requestor adds the Trust Chain JWS header parameter related to itself, this is RECOMMENDED;
  - The Requestor doesn't add the Trust Chain in the request, then the Issuer MUST start a Federation Discovery to obtain the Trust Chain related to the Requestor.
3. The Requestor begins the certificate issuance process by sending a POST request to the Issuer's `newOrder` resource.


| Action                | Request                                | Succesful Response |
|-----------------------|----------------------------------------|--------------------|
| Discovery             | GET Entity Configuration               | 200                |
|                       |                                        |                    |
| Get nonce             | HEAD newNonce                          | 200                |
|                       |                                        |                    |
| Submit order          | POST newOrder                          | 201                |
|                       |                                        |                    |
| Fetch challenges      | POST-as-GET order's authorization urls | 200                |
|                       |                                        |                    |
| Respond to challenges | POST authorization challenge urls      | 200                |
|                       |                                        |                    |
| Poll for status       | POST-as-GET order                      | 200                |
|                       |                                        |                    |
| Finalize order        | POST order's finalize url              | 200                |
|                       |                                        |                    |
| Poll for status       | POST-as-GET order                      | 200                |
|                       |                                        |                    |
| Download certificate  | POST-as-GET order's certificate url    | 200                |
|                       |                                        |                    |



## Metadata


The Issuer MUST publish its Entity Configuration including the `acme_provider` metadata within it.

This section describe how to use the parameters defined in the [Section 7.1.1](https://datatracker.ietf.org/doc/html/rfc8555#section-7.1.1) of [RFC8555] in the federation Entity Configuration of an Issuer.

````json
  {
   "metadata":
    "acme_provider": {
     "newNonce": "https://issuer.example.com/acme/new-nonce",
     "newOrder": "https://issuer.example.com/acme/new-order",
     "revokeCert": "https://issuer.example.com/acme/revoke-cert",
     "keyChange": "https://issuer.example.com/acme/key-change",
     "meta": {
       "termsOfService": "https://issuer.example.com/acme/terms/2017-5-30",
       "website": "https://www.issuer.example.com/",
       "caaIdentities": [!"issuer.example.com"],
       "externalAccountRequired": false
     }
   }
````

## newNonce request

The Requestor MUST obtain a new nonce from the Issuer, according to the [Section 7.2](https://datatracker.ietf.org/doc/html/rfc8555#section-7.2) of [RFC8555].


## ACME newOrder request within a Federation

The certificate issuance request is made by sending a HTTP POST to the Issuer `newOrder` resource, where the body of the POST is a JWS object whose JSON payload is a subset of the *ACME order object*, as defined in the [Section 7.4](https://datatracker.ietf.org/doc/html/rfc8555#section-7.4) of [RFC8555].

The *ACME order object* represents the request for a certificate issuance and is used to track the progress of that order through to issuance (see the [Section 7.1.6](https://datatracker.ietf.org/doc/html/rfc8555#section-7.1.6) of [RFC8555] for any further information about the statuses).


### Federation Extensions and Constraints

To the *ACME order object* properties already defined in the [Section 7.1.3](https://datatracker.ietf.org/doc/html/rfc8555#section-7.1.3) of [RFC8555] are added those defined by this document and listed below, to be intended as extensions to [RFC8555].

| place             | parameter     | type              | presence | reference                 |
|-------------------|---------------|-------------------|----------|---------------------------|
| protected headers | `trust_chain` | JSON Array of JWS | OPTIONAL | [OIDC-FED], Section 3.2.1 |


When OpenID Connect Federation 1.0 is used by the Issuer to attest the reliabiability of a Requestor and then authorize its request, this specification adds the following constraints to the `payload.identifiers` JSON Array:

- `type` MUST be set to `openid-federation`;
- `value` MUST correspond to the FQDN contained within the `iss` parameter of the Requestor's Entity Configuration. Since the Federation Entity ID is a HTTP URL, the corresponding FQDN MUST be extracted it. For example, if the Entity Configuration `iss` parameter contains the value `https://requestor.example.org/oidc/rp`, then the extracted FQDN is `requestor.example.org` and MUST correspond to the value of the identifier contained in the order object;
- the maximum length of the JSON Array contained in the `identifiers` parameter MUST be 1, since there cannot be more than a single FQDN corresponding to a single Federation Entity. If other identifiers  are present in the request and different from the type `openid-federation`, these SHOULD be ignored.


```http

   POST /acme/new-order HTTP/1.1
   Host: issuer.example.com
   Content-Type: application/jose+json

   {
     "protected": base64url({
       "alg": "ES256",
       "kid": "1",
       "nonce": "5XJ1L3lEkMG7tR6pA00clA",
       "url": "https://issuer.example.com/acme/new-order",
       "trust_chain": \["eyJhbGciOiJFU ...", "eyJhbGci ..."\]
     }),
     "payload": base64url({
       "identifiers": \[{ "type": "openid-federation", "value": "requestor.example.org" }\],
       "notBefore": "2024-01-01T00:04:00+04:00",
       "notAfter": "2024-01-08T00:04:00+04:00"
     }),
     "signature": "H6ZXtGjTZyUnPeKn...wEA4TklBdh3e454g"
   }

```

# Federation Identifiers Types

The "ACME Identifier Types" registry defined in the [Section 9.7.7](https://datatracker.ietf.org/doc/html/rfc8555#section-9.7.7) of [RFC8555] is extended with the types of identifiers listed below.

Template:

-  Label: The value to be put in the "type" field of the identifier object.
-  Reference: Where the identifier type is defined.

Contents:

| Label             | Reference                            |
|-------------------|--------------------------------------|
| openid-federation | draft-demarco-acme-openid-federation |
|                   |                                      |


# Security Considerations

TBD.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
