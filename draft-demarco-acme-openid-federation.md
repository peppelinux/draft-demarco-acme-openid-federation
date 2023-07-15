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

ACME in conjuction with OpenID Connect Federation 1.0 is able to issue X.509 certificates to one or more than a single organization without having pre-established any direct relationship or the stipulation of a contract between the parties. In a federation all the participants adhere to the same regulation or trust framework, OpenID Connect Federation 1.0 allows each participant to recognize the other participant using a trust evaluation mechanisms, using RESTful services and cryptographic materials.

This specification integrates ACME with OpenID Connect Federation 1.0, allowing the issuance of X.509 using an authorization pattern that identifies the requesting organization and attesting to it the reliability and the right to request a certificate, since:

- It does not require the involvement of other resources than the `newOrder`, since the authentication and authorization of the requestor is asserted with OpenID Federation 1.0.
- Instead of the `/.well-known/acme-challenge/{token}` endpoint it defines how to use and validate the basic OpenID Connect Federation component, called Entity Configuration, that is a signed JWT published in a well-known resource (.well-known/openid-federation).
- It removes the requirement for the authentication of an entity and the provisioning of acme-challenge tokens, since the authorization mechanisms is built on top of the trust evaluation meachanisms already in use with OpenID Connect Federation 1.0.
- It extends the ACME `newOrder` resource, defining the payload identifier type `openid-federation`.
- It defines how the OpenID Federation Entity Statements are used for the publication of the X.509 certificates automatically issued with ACME.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Audience Target audience/Usage

The audience of the document are the multilateral federations that require automatic issuance oif X.509 certificates using an infrastructure of trust based on OpenID Connect Federation 1.0.

# Scope

This specification defines how a [OIDC-FED] Federation API is used by ACME in the authorization phase for the issuance of the certificates.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
