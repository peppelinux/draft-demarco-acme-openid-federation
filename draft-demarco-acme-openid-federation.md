---
title: "Automatic Certificate Management Environment (ACME) with OpenID Federation 1.0"
abbrev: "ACME OpenID Federation"
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
 - OpenID Federation
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
  RFC1035: RFC1035
  RFC2818: RFC2818
  RFC2985: RFC2985
  RFC2986: RFC2986
  RFC5280: RFC5280
  RFC8555: RFC8555

  OPENID-FED:
    title: "OpenID Federation 1.0"
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

OpenID Federation 1.0 defines how to build a trust infrastructure using a
trusted third-party model. It uses a trust evaluation mechanism to attest the
possession of public keys, protocol specific metadata and several administrative
and technical information related to a specific entity.

This document defines how X.509 certificates associating a given OpenID
Federation Entity with a key included in that Entity's Configuration can be
issued by a trust anchor and its intermediates through the ACME protocol to all
the organizations that are part of a federation built on top of OpenID
Federation 1.0.

--- middle

# Introduction

OpenID Federation 1.0 allows an ACME server to issue X.509 Certificates
associating a given OpenID Entity to a key included in that Entity's
Configuration. X.509 Certificates can be provided to one or more organizations,
without having pre-established any direct relationship or any stipulation of a
contract.

In a multilateral federation, composed by thousands of entities belonging to
different organizations, all the participants adhere to the same regulation or
trust framework. OpenID Federation 1.0 allows each participant to recognize the
other participant using a trust evaluation mechanism, with RESTful services and
cryptographic materials.

Considering that a requestor is an entity requesting the issuance of an X.509
Certificate to a server and the issuer is the ACME server that validates the
entitlements of the requestor before issuing the X.509 Certificate, this
specification defines how ACME and OpenID Federation 1.0 can be integrated to
allow efficient issuance of X.509 Certificates to a requestor via the
introduction of a new ACME challenge type. The new challenge type extends the
ACME protocol in the following ways:

- It associates a cryptographic key with an OpenID Entity, rather than a domain,
  since the authentication and authorization of the requestor is asserted with
  OpenID Federation 1.0.

- It defines how to use and validate a basic OpenID Federation component, called
  Entity Configuration, that is a signed JWT published in a well-known resource
  (`/.well-known/openid-federation`) without requiring the
  `/.well-known/acme-challenge/{token}` endpoint.

- It defines how the OpenID Federation Subordinate Statements can be used for the
  publication of the X.509 Certificates, by a Trust Anchor or Intermediate, that
  were previously issued with ACME.

- It extends the ACME newOrder resource, as defined in Section 7.4 of
  [`RFC8555`], defining a new payload identifer type called
  `openid-federation`.

# Audience Target and Use Cases

The audience of the document are the multilateral federations that require
automatic issuance of X.509 Certificates using an infrastructure of trust based
on OpenID Federation 1.0.

This specification can be implemented by:

- Federation Entities that join to a federation staging area using HTTP only
  transport to attest themselves as trustworthy, and then retrieve X.509
  Certificates for their official HTTPS Federation Entity ID.

- Federation Entities that want to ask and obtain X.509 Certificate for every
  public cryptographic key in their Entity Configuration, be these
  Federation Entity Keys or metadata specific keys, as made reliable in a
  Federation Trust Chain.

# Terminology

The terms "Federation Entity", "Trust Anchor", "Intermediate", "Entity
Configuration", "Subordinate Statement", "Trust Mark" and "Trust Chain" used in this
document are defined in the [Section
1.2](https://openid.net/specs/openid-federation-1_0.html#name-terminology)
of [OPENID-FED]. The term "FQDN" used in this document is defined in [RFC1035].
The term "CSR" used in this document is defined in [RFC2986]. The
term Certificate Authority used in this document is defined in [RFC5280].


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Certificates issued using OpenID Federation

The Certificate Issuer establishes the authorization of a Federation Entity to obtain
X.509 Certificates for the identifier configured in the Requestor's Entity
Configuration.

The Certificate Issuer establishes if a Federation Entity is eligible to obtain X.509
Certificates for the identifier configured in the Requestor's Entity
Configuration.

The Federation Entity Keys are used to satisfy the Certificate Issuer's challenge, and the
public portion of the keys included in the issued X.509 Certificates.

The protocol assumes the following discovery preconditions are met. The
Issuer has the guarantee that:

1. The Requestor controls the private key related to the public part published
   in its Entity Configuration.

2. The Requestor controls its identifier, having published the
   Entity Configuration.

The CSR MUST include the public key, attested within the Trust Chain, used by
the Requestor to satisfy the Certificate Issuer's challenge.

This process may be repeated to request multiple X.509 Certificates related to the
Federation Entity Keys and linked to a single Entity.

# Protocol Flow

This section presents the protocol flow. The protocol flow is subdivided in the
following phases:

- **Discovery**, the Requestor obtains the available CAs within a federation,
inspecting the ACME provider metadata types.
- **Order request**, the Requestor requests a X.509 Certificate to a Certificate Issuer using
  the ACME protocol.

## Discovery Preconditions

The protocol assumes the following discovery preconditions are met, where for
discovery is intended the phase where a Requestor searches an Certificate Issuer to requests
an X.509 Certificate.

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

4. The Certificate Issuer MUST publish in its Entity Configuration, within the metadata
   parameter as defined in the [Section
   4](https://openid.net/specs/openid-federation-1_0.html#name-metadata-type-identifiers)
   of [OPENID-FED], the metadata type `acme_provider` according to the
   [Metadata](#metadata) of this specification.

5. The Certificate Issuer MAY be a Leaf, in these cases a specific Trust Mark
   enabling the issuance of X.509 Certificates within the federation MAY be
   issued by the Trust Anchor, or on behalf of it by an allowed Trust Mark
   issuer as configured in the federation. When used, the Trust Mark MUST be
   published within the Leaf's Entity Configuration.

Where the precondition number 4 and number 5 are not met, there MAY be some
cases where the Requestor knows a priori the identity of the Issuers in one or
more federations; in these cases the Requestor directly requests the issuance of
the X.509 Certificate to the trusted Certificate Issuer.

## Overview

TBD: high level design and ascii sequence diagram.

1. The Requestor checks if its superior Federation Entity supports the ACME
   protocol for OpenID Federation 1.0. If not, the Requestor starts the
   discovery process to find which are the Issuers within the federation.

2. The Requestor requests and obtains a new nonce from the Certificate Issuer,
   by sending a HTTP HEAD request to the Issuer's `newNonce` resource;

3. The Certificate Issuer evaluates the trust to the Requestor by checking if it is part of
   the federation. If not the `newNonce` request MUST be rejected (**TBD** the
   error to return). There are two ways the Certificate Issuer is able to check if a
   Requestor is part of the federation, these are listed below:

    - The Requestor adds the Trust Chain JWT header parameter related to itself.
      This option is RECOMMENDED since it reduces the effort of the Certificate Issuer in
      evaluating the trust to the Requestor;

    - The Requestor doesn't add the Trust Chain in the request. The Certificate Issuer
      MUST start a [Federation Entity
      Discovery](https://openid.net/specs/openid-federation-1_0.html#section-8)
      to obtain the Trust Chain related to the Requestor.

4. The Requestor begins the X.509 Certificate issuance process by sending a HTTP POST
   request to the Certificate Issuer's `newOrder` resource, and follows the remainder of the
   ACME protocol as specified in [RFC8555], using the new challenge defined in
   {{challenge-type}}.

## Metadata

The Issuer MUST publish its Entity Configuration including the `acme_provider`
metadata within it.

This section describe how to use the parameters defined in the [Section
7.1.1](https://datatracker.ietf.org/doc/html/rfc8555#section-7.1.1) of [RFC8555]
in the federation Entity Configuration of the Issuer.

TBD: the example below is non-normative. A table describing each REQUIRED or OPTIONAL
parameter is required.

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
}
~~~~

## newOrder Request

The Requestor begins certificate issuance by sending a HTTP POST request to the
Issuer's `newOrder` resource, as specified in Section 7.4 of [RFC8555]. However,
the request payload uses a new identifier `openid-federation`, whose value is
the `sub` parameter of the requestor's Entity Configuration, as defined in
Section 1.2 of [OPENID-FED].

A non-normative example of an ACME newOrder request:

~~~~
   POST /acme/new-order HTTP/1.1
   Host: issuer.example.com
   Content-Type: application/jose+json

   {
     "protected": base64url({
       "alg": "ES256",
       "kid": "https://issuer.example.com/acme/acct/evOfKhNU60wg",
       "nonce": "5XJ1L3lEkMG7tR6pA00clA",
       "url": "https://issuer.example.com/acme/new-order"
     }),
     "payload": base64url({
       "identifiers": [
         {
           "type": "openid-federation",
           "value": "https://requestor.example.com"
         }
       ],
       "notBefore": "2016-01-01T00:04:00+04:00",
       "notAfter": "2016-01-08T00:04:00+04:00"
     }),
     "signature": "H6ZXtGjTZyUnPeKn...wEA4TklBdh3e454g"
   }
~~~~

The maximum length of the JSON array contained in the identifiers parameter
MUST be 1, since there cannot be more than a single URI corresponding to a
Federation Entity.


## OpenID Federation Challenge Type {#challenge-type}

The OpenID Federation challenge type allows a Requestor to prove control of a
domain and its underlying endpoints using the trust evaluation mechanism
provided by OpenID Federation 1.0. The Requestor demonstrates control of a
cryptographic public key published in its OpenID Federation Entity Configuration.

The openid-federation-01 ACME challenge object has the following format:

type (required, string):  The string "openid-federation-01"

token (required, string):  A random value that uniquely identifies the
    challenge. This value MUST have at least 128 bits of entropy. It MUST NOT
    contain any characters outside the base64url alphabet as described in
    Section 5 of [RFC4648]. Trailing '=' padding characters MUST be stripped.
    See [RFC4086] for additional information on randomness requirements.

~~~~
   {
     "type": "openid-federation-01",
     "url": "https://issuer.example.com/acme/chall/prV_B7yEyA4",
     "status": "pending",
     "token": "LoqXcYV8q5ONbJQxbmR7SCTNo3tiAXDfowyjxAjEuX0"
   }
~~~~

The Requestor responds with an object with the following format:

sig (required, string):  a base64url encoding of a JWT, signing the token
    encoded in UTF-8 with one of the keys published in the requestor's OpenID
    Federation Entity Configuration: either in the top-level `jwks` claim as
    defined in Section 3 of [OIDC-FED] or referenced by the `signed_jwks_uri`,
    `jwks_uri`, or `jwks` claims in the entity metadata as defined in Section
    5.2.1 of [OIDC-FED]. It is REQUIRED that this JWT include a `kid` claim
    corresponding to a valid key; The Credential Issuer MUST only use keys with a
    corresponding `kid` value when evaluating the challenge response. Otherwise,

trust_chain (optional, array of string):  an array of base64url-encoded bytes
    containing a signed JWT and representing the Trust Chain of the Requestor.
    See section 4.3 of [OPENID-FED]. The Requestor SHOULD use
    a Trust Anchor it has in common with the ACME server. It is RECOMMENDED that the
    Requestor include this field; otherwise, the ACME server MUST start
    Federation Entity Discovery to obtain the Trust Chain related to the Requestor.

entity_identifier (optional, string):  the Entity Identifier of the Requestor,
    which is used by the ACME server to perform Federation Entity Discovery in the
    case that no Trust Chain is provided. The Requestor SHOULD include this field
    only when the `trust_chain` field is not provided.

A non-normative example for an authorization with `trust_chain` specified:

~~~~
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
~~~~

A non-normative example for an authorization with `entity_identifier` specified:

~~~~
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
~~~~

On receiving a response, the Certificate Issuer retrieves the public keys associated with
the given entity (possibly performing Federation Entity Discovery to do so),
then:

* Verifies that the requested `openid-federation` value matches the `sub`
  parameter of the Requestor's Entity Configuration. Since the Entity
  Configuration MUST contain at most one Entity Identifier, this effectively
  means this challenge type works with requests for a single Federation Entity
  only.

* Verifies that the `sig` field of the payload includes a valid JWT over the
  challenge token, signed with one of the keys published in the Requestor's
  Entity Configuration, either in the top-level `jwks` claim as defined in
  Section 3 of [OPENID-FED] or referenced by the `signed_jwks_uri`, `jwks_uri`, or
  `jwks` claims in the entity metadata as defined in Section 5.2.1 of
  [OPENID-FED]. If the Requestor provides a `kid` value in its challenge response,
  only keys (JWKs) in the Entity Configuration with a matching `kid` value are
  considered.

If all of the above verifications succeed, then the validation is successful.
Otherwise, it has failed. In either case, the Certificate Issuer responds according to
section 7.5.1 of [RFC8555]. In the event that the verification succeeds, the
eventual CSR MUST include the public key, attested within the Trust Chain, used
by the Requestor to satisfy the Certificate Issuer's challenge.


A non-normative example for the challenge object post-validation:

~~~~
   {
     "type": "openid-federation-01",
     "url": "https://issuer.example.com/acme/chall/prV_B7yEyA4",
     "status": "valid",
     "validated": "2024-10-01T12:05:13.72Z",
     "token": "LoqXcYV8q5ONbJQxbmR7SCTNo3tiAXDfowyjxAjEuX0"
   }
~~~~

### CSR and Certificate Requirements

When using this challenge type, both the certificate signing request (CSR)
and the X.509 Certificate:

* MUST include a public key corresponding to
  the key used to satisfy the challenge.

* MUST include no Common Name, and must include
  a single Subject Alternative Name value corresponding to an `otherName` with an
  ID of **TBD**, containing an Octet String value corresponding to a UTF-8
  encoding of the Requestor's Entity ID, that is, the value of the `sub` claim
  of the Requestor's Entity Configuration.

TODO: determine the OID for use in the Subject Alternative Name.

# Publication of the Certificates within the Federation

**TBD**, when the Certificate Issuer is the Trust Anchor or Intermediate, the X.509
Certificate linked to Federation Entity Key represented in JWK in the Subordinate
Statement related to the Requestor, SHOULD be extended with the claim `x5c`,
containing the issued X.509 Certificate.

# Certificate Lifecycle and Revocation

**TBD**.

The issued X.509 Certificates are related to the Federation Key attested within a
Trust Chain, their expiration time MUST be equal to the expiration of the Trust
Chain.

When a Federation Key is removed from the Requestor Entity Configuration
the X.509 Certificate related to it
SHOULD be revoked by its Credential Issuer, if not expired.

A Requestor SHOULD request the revocation of its X.509 Certificate when the related
cryptographic material is revoked and published in the Federation Historical Key
Registry.

The X.509 Certficate revocation request is defined in the [Section
7.6](https://datatracker.ietf.org/doc/html/rfc8555#section-7.6) of [RFC8555].

# Security Considerations

TBD.


# IANA Considerations

## ACME Identifier Types

Per this document, a new type has been added to the "ACME Identifyer Types"
registry, defined in
[Section 9.7.7](https://datatracker.ietf.org/doc/html/rfc8555#section-9.7.7) of
[RFC8555] with label "openid-federation" and reference
"draft-demarco-acme-openid-federation".


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
