---
title: "Automatic Certificate Management Environment (ACME) with OpenID Federation 1.0"
abbrev: "ACME OpenID Federation"
category: std

docname: draft-demarco-acme-openid-federation-latest
submissiontype: IETF
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
    ins: G. De Marco
    organization: independent
    email: demarcog83@gmail.com
 -
    name: Ameer Ghani
    org: ISRG
    email: inahga@divviup.org
 -
    name: Brandon Pitman
    org: ISRG
    email: bran@bran.land

normative:
  RFC2818: RFC2818
  RFC2986: RFC2986
  RFC5280: RFC5280

  OPENID-FED:
    title: "OpenID Federation 1.0 - draft 41"
    target: https://openid.net/specs/openid-federation-1_0-41.html
    date: 2024-12-04
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

  IANA-ACME:
    title: "Automated Certificate Management Environment (ACME) Protocol"
    target: https://www.iana.org/assignments/acme
    author:
      organization: IANA

  IANA-OAUTH:
    title: "OAuth Parameters"
    target: https://www.iana.org/assignments/oauth-parameters
    author:
      organization: IANA

informative:


--- abstract

The Automatic Certificate Management Environment (ACME) protocol [RFC8555]
allows server operators to obtain TLS certificates for their websites (HTTPS
[RFC2818]), based on a demonstration of control over the website's domain via a
fully-automated challenge/response protocol.

OpenID Federation 1.0 defines how to build a trust infrastructure using a
trusted third-party model. It uses a trust evaluation mechanism to attest the
possession of public keys, protocol specific metadata and various administrative
and technical information related to a specific entity.

This document defines how X.509 certificates associated with a given OpenID
Federation Entity can be
issued by an X.509 Certification Authority through the ACME protocol to the
organizations which are part of a federation built on top of OpenID
Federation 1.0.

--- middle

# Introduction

This document describes extensions to the ACME protocol that integrate with
OpenID Federation 1.0, allowing an ACME server to issue X.509 Certificates
associated with a given OpenID Federation Entity.
X.509 Certificates can be provided to one or more organizations,
without having pre-established any direct relationship or any stipulation of a
contract.

In a multilateral federation, composed by thousands of entities belonging to
different organizations, all the participants adhere to the same regulation or
trust framework. OpenID Federation 1.0 allows each participant to recognize
other participants using a trust evaluation mechanism, with RESTful services and
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
  (`/.well-known/openid-federation`).

- It defines how the OpenID Federation Subordinate Statements can be used for the
  publication of the X.509 Certificates, by a Superior Entity, that
  were previously issued with ACME.

- It extends the ACME newOrder resource, as defined in
  {{Section 7.4 of !RFC8555}}, defining a new payload identifier type called
  `openid-federation`.

# Target Audience and Use Cases

The audience of the document are the multilateral federations that require
automatic issuance of X.509 Certificates using an infrastructure of trust based
on OpenID Federation 1.0.

This specification can be implemented by:

- Federation Entities that join to a federation staging area using HTTP only
  transport to attest themselves as trustworthy, and then retrieve X.509
  Certificates for their official HTTPS Federation Entity ID.

- Federation Entities that want to ask for and obtain X.509 Certificate for use
  in other protocols.

# Terminology

The terms "Federation Entity", "Trust Anchor", "Entity Configuration",
"Subordinate Statement", "Superior Entity", "Immediate Superior Entity",
"Federation Entity Keys", "Trust Mark" and "Trust Chain" used in this
document are defined in {{Section 1.2 of OPENID-FED}}{: relative="#section-1.2"}.
The term "CSR" used in this document is defined in [RFC2986]. The
term Certification Authority used in this document is defined in [RFC5280]. The
terms "ACME Client" and "ACME Server" are defined in [RFC8555].

The specification also defines the following terms:

Requestor:
: A Federation Entity which wants to request X.509 certificates. It operates
  a web server for hosting its Entity Configuration. It also operates an ACME
  client, extended according to this document.

Certificate Issuer:
: A Federation Entity which issues X.509 certificates. It operates a web server
  for hosting its Entity Configuration. It also operates an ACME server,
  extended according to this document.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Certificates issued using OpenID Federation

The Certificate Issuer establishes the authorization of a Federation Entity to obtain
X.509 Certificates for the identifier configured in the Requestor's Entity
Configuration.

The Certificate Issuer establishes if a Federation Entity is eligible to obtain X.509
Certificates for the identifier configured in the Requestor's Entity
Configuration.

The cryptographic keys published within the Requestor's Entity Configuration
are used to satisfy the Certificate Issuer's challenge.

The protocol assumes the following discovery preconditions are met. The
Issuer has the guarantee that:

1. The Requestor controls the private key related to the public part published
   in its Entity Configuration.

2. The Requestor controls its identifier, having published the
   Entity Configuration.

# Protocol Flow

This section presents the protocol flow. The protocol flow is subdivided in the
following phases:

- **Discovery**, the Requestor obtains the available CAs within a federation,
inspecting the ACME issuer entity types.
- **Order request**, the Requestor requests a X.509 Certificate to a Certificate Issuer using
  the ACME protocol.

## Preconditions

The protocol requires the following preconditions are met.

1. The Requestor and the Issuer MUST publish their Entity Configuration as
   defined in {{Section 9 of OPENID-FED}}{: relative="#section-9"}.

2. The Requestor and the Issuer MUST be able to establish a Trust Chain to each
   other, as defined in {{Section 4 of OPENID-FED}}{: relative="#section-4"},
   from their respective Trust Anchors.

3. The Issuer MUST implement an ACME server, extended according to this document.

4. The Requestor MUST publish the entity type `acme_requestor` in its Entity
   Configuration, according to {{requestor-metadata}}.

5. The Issuer MUST publish the entity type `acme_issuer` in its Entity
   Configuration, according to {{issuer-metadata}}.

## Discovery

The Requestor's ACME client may either be configured to use a particular ACME
server, or to automatically discover a Certificate Issuer through the
federation.
Requestors that use discovery MAY select any entity with an entity type of
`acme_issuer`, or they may additionally require that such entities have a
valid Trust Mark with a particular Trust Mark Identifier.

## Overview

1. The Requestor checks if its superior Federation Entity supports the ACME
   protocol for OpenID Federation 1.0. If not, the Requestor starts the
   discovery process to find Issuers within the federation.

3. The Requestor requests and obtains a new nonce from the Certificate Issuer,
   by sending a HTTP HEAD request to the Issuer's `newNonce` resource,
   as described in {{Section 7.2 of !RFC8555}}.

2. The Requestor creates an ACME Account with the Issuer, as described in
   {{Section 7.3 of !RFC8555}}.

4. The Requestor begins the X.509 Certificate issuance process by sending a HTTP POST
   request to the Certificate Issuer's `newOrder` resource,
   as described in {{neworder-request}}, and follows the remainder of the
   ACME protocol as specified in {{!RFC8555}}, using the new challenge defined in
   {{challenge-type}}.

5. The Certificate Issuer evaluates the trust to the Requestor by checking if it
   is part of its federation. If not the CSR request MUST be rejected, with
   error type `urn:ietf:params:acme:error:openIDFederationEntity`,
   and an error code of `invalid_trust_chain`
   ({{Section 8.9 of OPENID-FED}}{: relative="#section-8.9"}).

There are two ways the Certificate Issuer is able to check if a
Requestor is part of the federation, these are listed below:

  - The Requestor adds the Trust Chain JWT header parameter related to itself,
    as described in {{Section 4.3 of OPENID-FED}}{: relative="#section-4.3"}.
    This option is RECOMMENDED since it reduces the effort of the Certificate
    Issuer in evaluating the trust to the Requestor.

  - The Requestor doesn't add the Trust Chain in the request. The Certificate Issuer
    MUST start Federation Entity Discovery as described in
    {{Section 9 of OPENID-FED}}{: relative="#section-9"}
    to obtain the Trust Chain related to the Requestor.

The following diagram illustrates a successful interaction between Issuer and
Requestor to retrieve an X.509 Certificate. The diagram assumes the Requestor
has already discovered the Issuer, and the Requestor has already created an
ACME account with the Issuer.

~~~~ ascii-art
,-----------------.
|Requestor's      |  ,-----------.
|OpenID Federation|  |Requestor's|               ,------------------------.  ,-----------------------.
| Web Server      |  |ACME Client|               |X.509 Certificate Issuer|  |Federation Trust Anchor|
`--------+--------'  `-----+-----'               `-----------+------------'  `-----------+-----------'
        |                  |           POST /acme/new-order  |                           |
        |                  |--------------------------------->                           |
        |                  |                                 |                           |
        |                  |Authorization at                 |                           |
        |                  |/acme/authz/[authz-id]           |                           |
        |                  |Finalize at                      |                           |
        |                  | /acme/order/[order-id]/finalize |                           |
        |                  |<- - - - - - - - - - - - - - - - -                           |
        |                  |                                 |                           |
        |                  | POST /acme/authz/[authz-id]     |                           |
        |                  |--------------------------------->                           |
        |                  |                                 |                           |
        |                  |  openid-federation-01 Challenge |                           |
        |                  |  at /acme/chall/[chall-id]      |                           |
        |                  |<- - - - - - - - - - - - - - - - -                           |
        |                  |                                 |                           |
        |                  ----.                             |                           |
        |                      | Sign challenge token        |                           |
        |                      | with private key            |                           |
        |                  <---'                             |                           |
        |                  |                                 |                           |
        |                  | POST /acme/chall/[chall-id]     |                           |
        |                  | with signed                     |                           |
        |                  | token and entity ID             |                           |
        |                  | set to Requestor's ID           |                           |
        |                  |--------------------------------->                           |
        |                  |                                 |                           |
        |                  |      Challenge validation       |                           |
        |                  |      beginning                  |                           |
        |                  |<- - - - - - - - - - - - - - - - -                           |
        |                  |                                 |                           |
        |          GET /.well-known/openid-federation        |                           |
        |<----------------------------------------------------                           |
        |                  |                                 |                           |
        |           Requestor's Entity Configuration         |                           |
        | - - -  - - - - - - - - - - - - - - - - - - - - - - >                           |
        |                  |                                 |                           |
        |                  |                                 |----.                      |
        |                  |                                 |    | Check                |
        |                  |                                 |    | Entity Configuration |
        |                  |                                 |    | sub matches          |
        |                  |                                 |    | Entity identifier    |
        |                  |                                 |<---' in the order         |
        |                  |                                 |                           |
        |                  |                                 |                           |
        |                  |                                 |----.                      |
        |                  |                                 |    | Check challenge      |
        |                  |                                 |    | sig is signed        |
        |                  |                                 |    | with key in          |
        |                  |                                 |<---' Entity Configuration |
        |                  |                                 |                           |
        |                  |                                 |                           |
        |                  |                                 |                           |
        |                  |           ______________________________________________________
        |                  |           ! OPT  /  If requestor didn't provide Trust Chain |  !
        |                  |           !_____/               |                           |  !
        |                  |           !                     |  Determine Trust Chain    |  !
        |                  |           !                     |  from Issuer's            |  !
        |                  |           !                     |  Trust Anchor to Requestor|  !
        |                  |           !                     |  (Federation Discovery)   |  !
        |                  |           !                     | <------------------------>|  !
        |                  |           !~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~!
        |                  |                                 |                           |
        |                  |                                 |----.                      |
        |                  |                                 |    | Evaluate trust chain |
        |                  |                                 |<---'                      |
        |                  |                                 |                           |
        |  _________________________________________________________________________     |
        |  ! LOOP  /  Poll until authz status                |                      !    |
        |  !      /  is "valid" or "invalid"                 |                      !    |
        |  !_____/         |                                 |                      !    |
        |  !               |    POST-as-GET                  |                      !    |
        |  !               |    /acme/authz/[authz-id]       |                      !    |
        |  !               |--------------------------------->                      !    |
        |  !               |                                 |                      !    |
        |  !               |           Current authz status  |                      !    |
        |  !               |<---------------------------------                      !    |
        |  !~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~!    |
        |                  |                                 |                           |
        |                  |                                 |                           |
        |  ___________________________________________________________________________   |
        |  ! OPT  /  If the authz status is "valid"          |                        !  |
        |  !_____/         |                                 |                        !  |
        |  !               | POST                            |                        !  |
        |  !               | /acme/orders/[order-id]/finalize|                        !  |
        |  !               | with CSR                        |                        !  |
        |  !               |--------------------------------->                        !  |
        |  !               |                                 |                        !  |
        |  !               |                                 |----.                   !  |
        |  !               |                                 |    | Check CSR         !  |
        |  !               |                                 |    | validity          !  |
        |  !               |                                 |    | according to      !  |
        |  !               |                                 |    | protocol          !  |
        |  !               |                                 |<---' and CA policy     !  |
        |  !               |                                 |                        !  |
        |  !               |                                 |                        !  |
        |  !               |  Order object with certificate  |                        !  |
        |  !               |  at /acme/cert/[cert-id]        |                        !  |
        |  !               |<- - - - - - - - - - - - - - - - -                        !  |
        |  !               |                                 |                        !  |
        |  !               |       POST /acme/cert/[cert-id] |                        !  |
        |  !               |--------------------------------->                        !  |
        |  !               |                                 |                        !  |
        |  !               |  Newly issued X.509 Certificate |                        !  |
        |  !               |<- - - - - - - - - - - - - - - - -                        !  |
        |  !~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~!  |
,--------+--------.  ,-----+-----.               ,-----------+------------.  ,-----------+-----------.
|Requestor's      |  |Requestor's|               |X.509 Certificate Issuer|  |Federation Trust Anchor|
|OpenID Federation|  |ACME Client|               `------------------------'  `-----------------------'
| Web Server      |  `-----------'
`-----------------'
~~~~

## Entity Configuration Metadata

This section describes the metadata a Requestor and Issuer MUST publish in their
respective Entity Configurations.

### Issuer Metadata

The Issuer MUST publish its Entity Configuration including the `acme_issuer`
metadata within it. The body of the `acme_issuer` metadata is the ACME
Directory, as defined in {{Section 7.1.1 of !RFC8555}}.

Requestors MUST use the ACME Directory provided in the Issuer's Entity
Configuration for client configuration of ACME endpoints.

The following is a non-normative example of an Entity Configuration including
the `acme_issuer` metadata:

~~~~
{
  "iss": "https://issuer.example.com",
  "sub": "https://issuer.example.com",
  "iat": 1516239022,
  "exp": 1516298022,
  "jwks": {
    "keys": [
      {
        "kty": "RSA",
        "alg": "RS256",
        "use": "sig",
        "kid": "NzbLsXh8uDCcd-6MNwXF4W_7noWXFZAfHkxZsRGC9Xs",
        "n": "pnXBOusEANuug6ewezb9J_...",
        "e": "AQAB"
      }
    ]
  },
  "metadata": {
    "acme_issuer": {
      "newNonce": "https://issuer.example.com/acme/new-nonce",
      "newOrder": "https://issuer.example.com/acme/new-order",
      "revokeCert": "https://issuer.example.com/acme/revoke-cert",
      "meta": {
        "termsOfService": "https://issuer.example.com/acme/terms/2017-5-30",
        "website": "https://www.issuer.example.com/",
        "externalAccountRequired": false
      }
    }
  }
}
~~~~

### Requestor Metadata

The Requestor MUST publish in its Entity Configuration an `acme_requestor`
metadata containing a JWK set, according to
{{Section 5.2.1 of OPENID-FED}}{: relative="#section-5.2.1"}.
The keys in the set are used to respond to ACME challenges.

The following is a non-normative example of an Entity Configuration including
the `acme_requestor` metadata and using the `jwks` metadata parameter.

~~~~
{
  "iss": "https://requestor.example.com",
  "sub": "https://requestor.example.com",
  "iat": 1516239022,
  "exp": 1516298022,
  "jwks": {
    "keys": [
      {
        "kty": "RSA",
        "alg": "RS256",
        "use": "sig",
        "kid": "NzbLsXh8uDCcd-6MNwXF4W_7noWXFZAfHkxZsRGC9Xs",
        "n": "pnXBOusEANuug6ewezb9J_...",
        "e": "AQAB"
      }
    ]
  },
  "metadata": {
    "acme_requestor": {
      "jwks": {
        "keys": [
          {
            "kty": "RSA",
            "kid": "SUdtUndEWVY2cUFDeD...",
            "n": "y_Zc8rByfeRIC9fFZrD...",
            "e": "AQAB"
          },
          {
            "kty": "EC",
            "kid": "MFYycG1raTI4SkZvVDBIMF9CNGw3VEZYUmxQLVN2T21nSWlkd3",
            "crv": "P-256",
            "x": "qAOdPQROkHfZY1daGofOmSNQWpYK8c9G2m2Rbkpbd4c",
            "y": "G_7fF-T8n2vONKM15Mzj4KR_shvHBxKGjMosF6FdoPY"
          }
        ]
      }
    }
  }
}
~~~~

The Issuer MUST only use the Requestor's `acme_requestor` to validate an ACME
challenge. Therefore, after completing the challenge, the Requestor MAY remove
the `acme_requestor` metadata from its Entity Configuration.

## OpenID Federation Identifier {#identifier-type}

This document defines a new ACME identifier type for OpenID Federation entities, `openid-federation`,
whose value is the `sub` parameter of the requestor's Entity Configuration,
as defined in {{Section 1.2 of OPENID-FED}}{: relative="#section-1.2"}.

## newOrder Request

The Requestor begins certificate issuance by sending a HTTP POST request to the
Issuer's `newOrder` resource, as specified in {{Section 7.4 of !RFC8555}}.

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

## OpenID Federation Challenge Type {#challenge-type}

The OpenID Federation challenge type allows a Requestor to prove control of a
Federation Entity using the trust evaluation mechanism
provided by {{OPENID-FED}}. The Requestor demonstrates control of a
cryptographic public key published in its OpenID Federation Entity Configuration.

The openid-federation-01 ACME challenge object has the following format:

type (required, string):  The string "openid-federation-01"

token (required, string):  A random value that uniquely identifies the
    challenge. This value MUST have at least 128 bits of entropy. It MUST NOT
    contain any characters outside the base64url alphabet as described in
    {{Section 5 of !RFC4648}}. Trailing '=' padding characters MUST be stripped.
    See {{!RFC4086}} for additional information on randomness requirements.

trustAnchors (optional, array of string):  An array of strings containing
    Entity Identifiers of the Issuer's trust anchors. When solving the
    challenge, the Requestor can construct a trust chain from itself to one of
    these trust anchors. It is RECOMMENDED that the Issuer includes this field
    to make it easier for the Requestor to construct a trust chain.

A non-normative example of a challenge with `trustAnchors` specified:

~~~~
   {
     "type": "openid-federation-01",
     "url": "https://issuer.example.com/acme/chall/prV_B7yEyA4",
     "status": "pending",
     "token": "LoqXcYV8q5ONbJQxbmR7SCTNo3tiAXDfowyjxAjEuX0",
     "trustAnchors": [
       "https://trust-anchor-1.example.com",
       "https://trust-anchor-2.example.com"
     ]
   }
~~~~

The `openid-federation-01` challenge MUST NOT be used to issue X.509 Certificates
for any identifiers except `openid-federation` identifiers.

The `openid-federation` identifier MUST NOT be validated except by the
`openid-federation-01` challenge.

The Requestor responds to the challenge with an object with the following format:

sig (required, string):  the compact JSON serialization (as described in
    {{Section 7.1 of !RFC7515}}) of a JWS, signing the key authorization
    encoded in UTF-8.
    The key authorization is computed from the token in the challenge and the
    Requestor's ACME account key, as defined in {{Section 8.1 of !RFC8555}}.
    The signature must be made by one of the keys published in the Requestor's
    `acme_requestor` metadata in its Entity Configuration, as specified in
    {{requestor-metadata}}.
    The JWS MUST include a `kid` header parameter corresponding to the key used
    to sign the key authorization and a `typ` header parameter set to
    "signed-acme-challenge+jwt".

trust_chain (optional, array of string):  an array of strings containing signed
    JWTs, representing a Trust Chain from the Requestor to one of the Issuer's
    trust anchors (see
    {{Section 4.3 of OPENID-FED}}{: relative="#section-4.3"}). It is RECOMMENDED
    that the Requestor includes this field; otherwise, the ACME server MUST
    start Federation Entity Discovery to obtain the Trust Chain related to the
    Requestor.

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

On receiving a challenge response, the Certificate Issuer retrieves the public keys associated with
the given entity (possibly performing Federation Entity Discovery to do so),
then:

* Verifies that the requested `openid-federation` identifier value matches the `sub`
  parameter of the Requestor's Entity Configuration.

* Verifies that the `sig` field of the payload includes a valid JWT over the
  key authorization, signed with one of the keys published in the Requestor's
  `acme_requestor` metadata in its Entity Configuration, as specified in
  {{requestor-metadata}}. The Issuer MUST only consider the key
  whose `kid` matches the `kid` claim in the Requestor's challenge response.
  The Issuer also MUST only consider keys published in the Requestor's
  `acme_requestor` metadata.

If all of the above verifications succeed, then the validation is successful.
Otherwise, it has failed. In either case, the Certificate Issuer responds according to
{{Section 7.5.1 of !RFC8555}}.

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

### CSR and Certificate Fields {#openidfed-othername-id}

Depending on the Certificate Issuer's X.509 Certificate profile, the CSR and
X.509 Certificate MAY associate the X.509 Certificate to the Federation Entity by
including the Entity ID in the X.509 Certificate.
To do so, the Issuer includes a Subject Alternative Name extension
containing an `otherName` with
a `type-id` of `id-on-OpenIdFederationEntityId`.
The value of the name is an Octet String
containing the UTF-8 encoding of the Entity ID
(i.e., the URI in the corresponding `openid-federation` identifier
from the `newOrder` request).

~~~~
   id-on-OpenIdFederationEntityId OBJECT IDENTIFIER ::= { id-on XXX }

   OpenIdFederationEntityId ::= UTF8String
~~~~

# Publication of the Certificates within the Federation

All X.509 Certificates issued SHOULD be published to one or more Certificate
Transparency logs {{!RFC9162}}. Selection of those logs, their versions, and
embedding or discovery of Signed Certificate Timestamps, are policy decisions
and are out of scope for this document.

The X.509 Certificates issued by federation Immediate Superior Entities
pertaining to one or more Federation Entity Keys in control of their
Subordinates MAY publish this information by including the `x5c` member
in each JWK contained within the matching Subordinate Statement. The contents
of the published `x5c` member, including whether it contains a full or
partial Trust Chain, and if so, to what Trust Anchor, are policy decisions
out of scope for this document.

# Certificate Lifecycle and Revocation

The identity of the Requestor is verified through proof of possession of a
private key corresponding to a public key
attested within a Trust Chain. It is up to the Certificate Issuer to decide
the expiration time of the X.509 Certificate. In some cases, and when required,
it MAY be set to match the expiration of the Trust Chain.

A Requestor SHOULD request the revocation of its X.509 Certificate when the related
cryptographic material is revoked. The Requestor SHOULD publish the revoked or
expired cryptographic keys in the Federation Historical Key Registry.

The X.509 Certificate revocation request is defined in {{Section 7.6 of !RFC8555}}.

# Errors {#error-type}

This document defines one new error type URI to be used in problem documents
{{!RFC9457}}, as described in {{Section 6.7 of !RFC8555}}.

The error type `urn:ietf:params:acme:error:openIDFederationEntity` can
be used to encapsulate any OAuth error code returned while resolving OpenID
Federation Entities. The title of this error type is "OpenID Federation Error".
The `detail` member of the problem document MAY include the description of the
particular OAuth error code that caused the error. The problem document for this
error type SHOULD include an extension member named `error_code`. The
`error_code` member SHOULD be set to the OAuth error code, taken from the IANA
"OAuth Extensions Error Registry" {{IANA-OAUTH}}.

# Security Considerations

The `openid-federation-01` challenge defined in {{challenge-type}} defends
against replay attacks by malicious ACME servers because the signature in
challenge responses is over an ACME key authorization, which binds the ACME
account key.

The cryptographic keys in the `acme_requestor` metadata SHOULD NOT be reused
for other purposes than signing responses to `acme-federation-01` challenges.
For example, the same keys SHOULD NOT be reused in the issued X.509 Certificate.
If the keys are reused for other purposes, cross-protocol attacks MUST be
considered.

The cryptographic keys in the `acme_requestor` metadata SHOULD be rotated
periodically.

# IANA Considerations

IANA is kindly asked to make the following updates to registries:

## ACME Registry Group

The following updates are all assignments in the "Automated Certificate
Management Environment (ACME) Protocol" registry group {{IANA-ACME}}.

### ACME Identifier Types

IANA is asked to add to the "ACME Identifier Types" registry, defined in
{{Section 9.7.7 of !RFC8555}}, the entry below, as specified here in
{{identifier-type}}:

|Label|Reference|
|-----|---------|
|openid-federation|this document|

### ACME Validation Methods

IANA is also asked to add to the "ACME Validation Methods" registry, defined
in {{Section 9.7.8 of !RFC8555}}, the entry below, as specified here
in {{challenge-type}}:

|Label|Identifier Type|Reference|
|-----|---------------|---------|
|openid-federation-01|openid-federation|this document|

### ACME Error Types

IANA is also asked to add to the "ACME Error Types" registry, defined
in {{Section 9.7.4 of !RFC8555}}, the entry below, as specified here in
{{error-type}}:

|Type|Description|Reference|
|----|-----------|---------|
|openIDFederationEntity|An error occurred while resolving an OpenID Federation entity|this document|

## Assign X.509 PKIX Other Name

IANA is asked to add to the "PKIX Other Name Forms" registry
([1.3.6.1.5.5.7.8](https://www.iana.org/assignments/smi-numbers/smi-numbers.xhtml#smi-numbers-1.3.6.1.5.5.7.8)) the entry below, as
specified here in {{openidfed-othername-id}}

|Decimal|Description|Reference|
|-------|-----------|---------|
|TBA    |id-on-OpenIdFederationEntityId|this document|

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
