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

This document defines how the X.509 certificates can be issued by a trust anchor and its intermediates through the ACME protocol to all the organizations that are part of a federation built on top of OpenID Connect Federation 1.0.

--- middle

# Introduction

The Automatic Certificate Management Environment (ACME) is the standard [RFC8555] that allows obtaining certificates for websites (HTTPS [RFC2818]) by verifing the "fully-qualified" domain names [RFC3696] and the web servers within these.

OpenID Connect Federation 1.0 [OIDC-FED] is the standard that allows building multilateral federations through a trust evaluation mechanism attesting the possession of public keys, signature capabilities, protocol specific metadata and several administrative and tecnical information in the form of trust marks, related to a specific entity belonging to an organization.

OpenID Connect Federation 1.0 allows an ACME server to issue X.509 certificates to one or more than a single organization without having pre-established any direct relationship or any stipulation of a contract to the requestor. In a multilateral federation, composed by thousands of entities belonging to different organizations, all the participants adhere to the same regulation or trust framework. OpenID Connect Federation 1.0 allows each participant to recognize the other participant using a trust evaluation mechanism, with RESTful services and cryptographic materials.

Considering that a requestor is an entity requesting the issuance of a X.509 certificate to a server and the issuer is the ACME server that validates the entitlements of the requestor before issuing the X.509 certificate, this specification defines how ACME and OpenID Connect Federation 1.0 can be integrated allowing an efficient issuance of X.509 to a requestor, reducing both the bureaucratic and the implementative costs, since:

- It does not require the involvement of other resources than the ACME `newOrder` endpoint, since the authentication and authorization of the requestor is asserted with OpenID Federation 1.0.
- Instead of the `/.well-known/acme-challenge/{token}` endpoint it defines how to use and validate a basic OpenID Connect Federation component, called Entity Configuration, that is a signed JWT published in a well-known resource (`/.well-known/openid-federation`) by the requestor.
- It removes the requirement for the authentication of an entity and the provisioning of the *acme-challenge token*, since the authorization mechanisms is built on top of the trust evaluation model as defined in OpenID Connect Federation 1.0.
- It extends the ACME `newOrder` endpoint, defining a new payload identifier type called `openid-federation`.
- It defines how the OpenID Federation Entity Statements can be used for the publication of the X.509 certificates, by a trust anchor or intermediate, that was previously issued with ACME.

# Audience Target audience/Usage

The audience of the document are the multilateral federations that require automatic issuance oif X.509 certificates using an infrastructure of trust based on OpenID Connect Federation 1.0.

# Terminology

   ACME    Automated Certificate Management Environment, a certificate management protocol [RFC8555.

   TA      OpenID Connect Federation Trust Anchor, see CA

   CA      Certification Authority, also known as Trust Anchor or Intermediate, specifically one that implements the ACME protocol though an ACME server.

   CSR     Certificate Signing Request, specifically a PKCS#10 [RFC2986] as supported by ACME.

   FQDN    Fully Qualified Domain Name.

   Requestor ...

   Issuer ...

The terms "Trust Anchor", "Intermediate", "Entity Configuration", "Entity Statement", "Trust Mark" and "Trust Chain" used in this document are defined in [OIDC-FED].

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Protocol Flow

This section presents the protocol flow.

## Discovery Preconditions

The protocol assumes the following discovery preconditions are met, where for discovery is intended the phase where a Requestor search an Issuer to requests an X.509 certificate.

1. The Requestor and the Issuer MUST publish their Entity Configuration as defined in ... OIDC FED ref here.
2. The Requestor and the Issuer MUST be able to establish the trust to each other obtaining the Trust Chain of each other, as defined in ... OIDC FED ref here.
3. The Trust Anchor and its Intermediate SHOULD implement an ACME server with at least the `newOrder` endpoint as extended in this specification.
4. The Issuer MUST publish in its Entity Configuration, within the metadata parameter (JSON Object), the metadata type `acme_provider` according to the Section ... of this specification, **TBD**.
5. The Issuer MAY be a Leaf, in these cases a specific Trust Mark SHOULD be issued by the Trust Anchor, or on behalf of this latter through the allowed Trust Mark issuers configured in the federation, and then published within the Leaf Entity Configuration.

Where the precondition number 4 and number 5 are not met, there MAY be some cases where the Requestor known a priori which are the Issuers in one or more federations, in this case the requestor directly requests the issuance of the X.509 certificate to the trusted issuer.

## Overview

TBD: high level design and ascii sequence diagram.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
