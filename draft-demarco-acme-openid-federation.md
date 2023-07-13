---
title: "Automatic Certificate Management Environment (ACME) for OpenID Connect Federation 1.0"
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

The Automatic Certificate Management Environment (ACME) is the standard [RFC8555] that allows obtaining certificates for websites (HTTPS [RFC2818]) by verifing the "fully-qualified" domain names [RFC3696] and the web servers that publishes contents within them.

OpenID Connect Federation 1.0 [OIDC-FED] is a general purpose trust evaluation mechanism that allow to attests the public keys and several administrative and protocol specific information related to a specific organization, with the involvement of a trusted third party and the signed material published in their web services.

This document defines how an ACME server can issue X.509 certificates to organizations at a large scale thanks to the OpenID Connect Federation 1.0 trust evaluation mechanism, where domains, web service and cryptographic materials are attested under the sole control of an entity and this latter attested as trusted and reliable.

--- middle

# Introduction

ACME in conjuction of OpenID Connect Federation 1.0 is able to issue X.509 certificates to one or more than a single organization without having pre-established any direct relationship of trust to them or stipulation of a contract. OpenID Connect Federation 1.0 enables trust building in multilateral federated contexts, where all participants adhere to the same rules or trust framework, giving the proof that an organization is part of the common regulation, established by the federation. To assure this, the organization gives the proof of being in full control of the cryptographic material involved in the trust mechanisms, and the web services published within its domains.

This specification harmonizes the capabilities and features of OpenID Connect Federation 1.0 and ACME, since:

- It does not require the involvement of other resources than the `newOrder`, since the authentication and authorization of the requestor is asserted with OpenID Federation 1.0.
- It extends the ACME `newOrder` resource, defining the payload identifier type `openid-federation`
- It defines how the OpenID Federation Entity Statements are used for the publication of the X.509 certificates issued with ACME.

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
