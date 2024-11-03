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
  RFC2985: RFC2985
  RFC2986: RFC2986
  RFC8555: RFC8555

  OPENID-FED:
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

The Automatic Certificate Management Environment (ACME) protocol [RFC8555]
allows server operators to obtain TLS certificates for their websites (HTTPS
[RFC2818]), based on a demonstration of control over the website's domain via a
fully-automated challenge/response protocol.

OpenID Federation 1.0 defines how to build a trust infrastructure
using a trusted third-party model.
It uses a trust evaluation mechanism to attest the
possession of public keys, protocol specific metadata
and several administrative and technical information
related to a specific entity.

This document defines how X.509 certificates associating a given OpenID Federation Entity
with a key included in that Entity's Configuration can be issued by a trust
anchor and its intermediates through the ACME protocol to all the organizations
that are part of a federation built on top of OpenID Federation 1.0.

--- middle

# Introduction

OpenID Federation 1.0 allows an ACME server to issue X.509 certificates
associating a given OpenID Entity to a key included in that Entity's
Configuration. X.509 Certificates can be provided to one or more organizations,
without having pre-established any direct relationship or any stipulation of a
contract.

In a multilateral federation, composed by thousands of entities belonging to
different organizations, all the participants adhere to the same regulation or
trust framework. OpenID Federation 1.0 allows each participant to
recognize the other participant using a trust evaluation mechanism, with RESTful
services and cryptographic materials.

Considering that a requestor is an entity requesting the issuance of a X.509
Certificate to a server and the issuer is the ACME server that validates the
entitlements of the requestor before issuing the X.509 certificate, this
specification defines how ACME and OpenID Federation 1.0 can be
integrated to allow efficient issuance of X.509 certificates to a requestor via
the introduction of a new ACME challenge type. The new challenge type extends
the ACME protocol in the following ways:

- It associates a cryptographic key with an OpenID Entity, rather than a domain,
  since the authentication and authorization of the requestor is asserted with
  OpenID Federation 1.0.

- It defines how to use and validate a basic OpenID Federation
  component, called Entity Configuration, that is a signed JWT published in a
  well-known resource (`/.well-known/openid-federation`) without requiring the
  `/.well-known/acme-challenge/{token}` endpoint.

- It defines how the OpenID Federation Entity Statements can be used for the
  publication of the X.509 Certificates, by a Trust Anchor or Intermediate, that
  were previously issued with ACME.

# Audience Target and Use Cases

The audience of the document are the multilateral federations that require
automatic issuance of X.509 certificates using an infrastructure of trust based
on OpenID Federation 1.0.

This specification can be implemented by:

- Federation Entities that join to a federation staging area using HTTP only
  transport to attest themselves as trustworthy, and then retrieve X.509
  certificates for their official HTTPS Federation Entity ID.

- Federation Entities that want to ask and obtain X.509 certificate for every
  Federation Key contained in their Entity Configuration, as made reliable in a
  Federation Trust Chain.


# Terminology

The terms "Federation Entity", "Trust Anchor", "Intermediate", "Entity
Configuration", "Entity Statement", "Trust Mark" and "Trust Chain" used in this
document are defined in the [Section
1.2](https://openid.net/specs/openid-federation-1_0.html#name-terminology)
of [OPENID-FED].

**TA**:
: OpenID Federation Trust Anchor, see CA.

**CA**:
: Certification Authority, also known as Trust Anchor or Intermediate,
  specifically one that implements the ACME protocol by serving an ACME server.

**CSR**:
: Certificate Signing Request, specifically a PKCS#10 [RFC2986] as supported by
  ACME.

**FQDN**:
: Fully Qualified Domain Name.

**Requestor**:
: Federation Entity that requests a X.509 certificate to a CA.

**Issuer**:
: Federation Entity that serves an ACME Server. The Federation Entity is then a
  CA.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Certificates issued using OpenID Federation

The Issuer establishes the authorization of a Federation Entity to obtain
certificates for the identifier configured in the Requestor's Entity
Configuration.

The Federation Entity Keys are used to satisfy the Issuer's challenge, and the
public portion of the keys are included in the issued X.509 certificates.

The protocol assumes the following discovery preconditions are met. Then the
Issuer has the guarantee that:

1. The Requestor controls the private key related to the public part published
   in its Entity Configuration, attested by the superior Subordinate Statement.

2. The Requestor controls the identifier in question, having published the
   Entity Configuration.

3. The CSR MUST include the public key, attested within the Trust Chain,
   used by the Requestor to satisfy the Issuer's challenge.

This process may be repeated to request multiple certificates related to the
Federation Entity Keys and linked to a single Entity.

# Protocol Flow

This section presents the protocol flow. The protocol flow is subdivided in the
following phases:

- **Discovery**, the Requestor obtains the available CAs within a federation.
- **Order request**, the Requestor requests a X.509 certificate to a CA using
  the ACME protocol.

## Discovery Preconditions

The protocol assumes the following discovery preconditions are met, where for
discovery is intended the phase where a Requestor searches an Issuer to requests
an X.509 certificate.

1. The Requestor and the Issuer MUST publish their Entity Configuration as
   defined in the [Section
   6](https://openid.net/specs/openid-federation-1_0.html#name-obtaining-federation-entity)
   of [OPENID-FED].

2. The Requestor and the Issuer MUST be able to establish the trust to each
   other obtaining the Trust Chain of each other, as defined in the [Section
   3.2](https://openid.net/specs/openid-federation-1_0.html#name-trust-chain)
   of [OPENID-FED].

3. The Trust Anchor and its Intermediates SHOULD implement an ACME server,
   extended according to this document.

4. The Issuer MUST publish in its Entity Configuration, within the metadata
   parameter as defined in the [Section
   4](https://openid.net/specs/openid-federation-1_0.html#name-metadata-type-identifiers)
   of [OPENID-FED], the metadata type `acme_provider` according to the
   [Metadata](#metadata) of this specification.

5. The Issuer MAY be a Leaf, in these cases a specific Trust Mark SHOULD be
   issued by the Trust Anchor, or on behalf of it by an allowed Trust Mark
   issuer as configured in the federation, and the Trust Mark MUST then be
   published within the Leaf Entity Configuration.

Where the precondition number 4 and number 5 are not met, there MAY be some
cases where the Requestor knows a priori the identity of the Issuers in one or
more federations; in these cases the Requestor directly requests the issuance of
the X.509 certificate to the trusted Issuer.

## Overview

TBD: high level design and ascii sequence diagram.

1. The Requestor checks if its superior Federation Entity supports the ACME
   protocol for OpenID Federation 1.0. If not, the Requestor starts the
   discovery process to find which are the Issuers within the federation.

2. The Requestor requests and obtains a new nonce from the Issuer, by sending a
   HTTP HEAD request to the Issuer's `newNonce` resource;

3. The Issuer evaluates the trust to the Requestor by checking if it is part of
   the federation. If not the `newNonce` request MUST be rejected (**TBD** the
   error to return). There are two ways the Issuer is able to check if a
   Requestor is part of the federation, these are listed below:

    - The Requestor adds the Trust Chain JWS header parameter related to itself,
      this option is RECOMMENDED since it reduces the effort of the Issuer in
      evaluating the trust to the Requestor;

    - The Requestor doesn't add the Trust Chain in the request, then the Issuer
      MUST start a [Federation Entity
      Discovery](https://openid.net/specs/openid-federation-1_0.html#section-8)
      to obtain the Trust Chain related to the Requestor.

4. The Requestor begins the certificate issuance process by sending a HTTP POST
   request to the Issuer's `newOrder` resource, and follows the remainder of the
   ACME protocol as specified in [RFC8555], using the new challenge defined in
   {{challenge-type}}.

## Metadata

The Issuer MUST publish its Entity Configuration including the `acme_provider`
metadata within it.

This section describe how to use the parameters defined in the [Section
7.1.1](https://datatracker.ietf.org/doc/html/rfc8555#section-7.1.1) of [RFC8555]
in the federation Entity Configuration of the Issuer.

~~~~
{
  "metadata":
    "acme_provider": {
      "newNonce": "https://issuer.example.com/acme/new-nonce",
      "newOrder": "https://issuer.example.com/acme/new-order",
      "revokeCert": "https://issuer.example.com/acme/revoke-cert",
      "meta": {
        "termsOfService": "https://issuer.example.com/acme/terms/2017-5-30",
        "website": "https://www.issuer.example.com/",
        "caaIdentities": ["issuer.example.com"],
        "externalAccountRequired": false
   }
}
~~~~

## OpenID Federation challenge type {#challenge-type}

The OpenID Federation challenge type allows a client to prove control of a
domain and its underlying endpoints using the trust evaluation mechanism
provided by OpenID Federation 1.0. The client demonstrates control of a
cryptographic public key published in its OpenID Federation Entity
Configuration, which the ACME server uses to validate that the client is in
control of the domain.

The openid-federation-01 ACME challenge object has the following format:

type (required, string):  The string "openid-federation-01"

token (required, string):  A random value that uniquely identifies the
    challenge. This value MUST have at least 128 bits of entropy. It MUST NOT
    contain any characters outside the base64url alphabet as described in
    Section 5 of [RFC4648]. Trailing '=' padding characters MUST be stripped.
    See [RFC4086] for additional information on randomness requirements.

```
   {
     "type": "openid-federation-01",
     "url": "https://issuer.example.com/acme/chall/prV_B7yEyA4",
     "status": "pending",
     "token": "LoqXcYV8q5ONbJQxbmR7SCTNo3tiAXDfowyjxAjEuX0"
   }
```

The client responds with an object with the following format:

sig (required, string):  a base64url encoding of a JWS, signing the token
    encoded in UTF-8 with one of the keys published in the client's OpenID
    Federation Entity Configuration.

trust_chain (optional, array of string):  an array of base64url-encoded bytes
    containing a signed JWT and representing the trust chain of the client in
    the OpenID Federation. See section 4.3 of [OPENID-FED]. The client SHOULD use
    a trust anchor it has in common with the server. It is RECOMMENDED that the
    client include this field; otherwise, the ACME server MUST start
    Federation Entity Discovery to obtain the trust chain related to the client.

entity_identifier (optional, string):  the Entity Identifier of the client,
    which is used by the server to perform Federation Entity Discovery in the
    case that no trust chain is provided. The client SHOULD include this field
    only when the `trust_chain` field is not provided.

A non-normative example for an authorization with `trust_chain` specified:

```
   POST /acme/chall/prV_B7yEyA4
   Host: issuer.example.com
   Content-Type: application/jose+json

   {
     "protected": base64url({
       "alg": "ES256",
       "kid": "https://issuer.example.com/acme/acct/evOfKhNU60wg",
       "nonce": "UQI1PoRi5OuXzxuX7V7wL0",
       "url": "https://issuer.example.com/acme/chall/prV_B7yEyA4"
     }),
     "payload": base64url({
      "sig": "wQAvHlPV1tVxRW0vZUa4BQ...",
      "trust_chain": ["eyJhbGciOiJFU...", "eyJhbGci..."]
     }),
     "signature": "Q1bURgJoEslbD1c5...3pYdSMLio57mQNN4"
   }
```

A non-normative example for an authorization with `entity_identifier` specified:

```
   POST /acme/chall/prV_B7yEyA4
   Host: issuer.example.com
   Content-Type: application/jose+json

   {
     "protected": base64url({
       "alg": "ES256",
       "kid": "https://issuer.example.com/acme/acct/evOfKhNU60wg",
       "nonce": "UQI1PoRi5OuXzxuX7V7wL0",
       "url": "https://issuer.example.com/acme/chall/prV_B7yEyA4"
     }),
     "payload": base64url({
      "sig": "wQAvHlPV1tVxRW0vZUa4BQ...",
      "entity_identifier": "https://requestor.example.com"
     }),
     "signature": "Q1bURgJoEslbD1c5...3pYdSMLio57mQNN4"
   }
```


On receiving a response, the server retrieves the public keys associated with
the given entity (possibly performing Federation Entity Discovery to do so),
then:

* Verifies that the requested domain names match the FQDN contained within the
  `sub` parameter of the client's Entity Configuration. For example, if the
  `sub` parameter within the Entity Configuration contains the value
  `https://requestor.example.com/oidc/rp`, the extracted FQDN is then
  `requestor.example.com`. Since the Entity Configuration can contain at most
  one FQDN, this effectively means that this challenge type works with requests
  for a single domain name only.

* Verifies that the sig field of the payload includes a valid JWS, signed with
  one of the keys published in the client's Entity Configuration.

If all of the above verifications succeed, then the validation is successful.
Otherwise, it has failed. In either case, the server responds according to
section 7.5.1 of [RFC8555].

A non-normative example for the challenge object post-validation:

```
   {
     "type": "openid-federation-01",
     "url": "https://issuer.example.com/acme/chall/prV_B7yEyA4",
     "status": "valid",
     "validated": "2024-10-01T12:05:13.72Z",
     "token": "LoqXcYV8q5ONbJQxbmR7SCTNo3tiAXDfowyjxAjEuX0"
   }
```

TODO: update text below here

# Publication of the Certificates within the Federation

**TBD**, when the Issuer is the Trust Anchor or Intermediate, the X.509
certificate linked to Federation Entity Key represented in JWK in the Entity
Statement related to the Requestor, SHOULD be extended with the claim `x5c`,
containing the issued certificate.

# Certificate Lifecycle and Revocation

**TBD**.

The issued Certificates are related to the Federation Key attested within a
Trust Chain, their expiration time MUST be equal to the expiration of the Trust
Chain.

When a Federation Key is removed from the Entity Statement that attests it, and
then it cannot be attested though a Trust Chain, the certificate related to it
MUST be revoked by its Issuer, if not expired.

A Requestor SHOULD request the revocation of its Certificate when the related
Federation Entity Key is revoked and published in the Federation Historical Key
Registry.

The certficate revocation request is defined in the [Section
7.6](https://datatracker.ietf.org/doc/html/rfc8555#section-7.6) of [RFC8555].

# Security Considerations

TBD.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
