---
title: "Scrubbing BGP ORIGIN Attribute"
category: std

docname: draft-marenamat-idr-scrub-bgp-origin-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
updates: 4271
date:
consensus: true
v: 3
area: "Routing"
workgroup: "Inter-Domain Routing"
keyword:
 - EGP
 - attribute manipulation
venue:
  group: "Inter-Domain Routing"
  type: "Working Group"
  mail: "idr@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/idr/"
  github: "marenamat/ietf-draft-marenamat-idr-scrub-bgp-origin"
  latest: "https://marenamat.github.io/ietf-draft-marenamat-idr-scrub-bgp-origin/draft-marenamat-idr-scrub-bgp-origin.html"

author:
  - fullname: "Maria Matejka"
    organization: CZ.NIC
    street: Milesovska 1136/5
    city: Praha
    code: 13000
    country: Czechia
    email:
    - maria.matejka@nic.cz
    - mq@jmq.cz

normative:
  RFC4271: bgp

#informative:
#  RFC6472: as-set-not-use
#  RFC9774: as-set-deprecation

--- abstract

The BGP Origin attribute in its original meaning has been out of use for years.
Yet, the BGP Origin attribute has high priority in the best route
selection algorithm, right after the AS Path length, and it's being used
inconsistently over the Internet to manipulate the route preference.

This document updates RFC 4271 by making the BGP Origin attribute half-optional
and explicitly allowing its scrubbing to zero (IGP).

--- middle

# Introduction

TODO Introduction


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Tolerance of missing or malformed BGP Origin attribute

Unless explicitly configured by a network operator to do otherwise,
if the ORIGIN attribute has an undefined value, BGP speakers SHOULD
replace it by a value of 0 (IGP).

BGP speakers SHOULD NOT generate UPDATE Message Errors with the subcode 6,
Invalid ORIGIN Attribute.

Per the above specification, this document updates {{Section 4.5 of -bgp}}
by accepting routes without the ORIGIN attribute.

# Update to the BGP Origin attribute values

The values of the BGP Origin attributed are originally specified
in {{Section 4.3 of -bgp}}, the Path attribute part, a) ORIGIN (Type Code 1).

Unless explicitly configured by a network operator to do otherwise,
BGP speakers MUST NOT advertise BGP UPDATE messages with the ORIGIN
attribute containing any other value than 0 (IGP), including manually
configured static routes.

Additionally, BGP speakers MAY rewrite any ORIGIN attribute value
to 0 (IGP) if they contain any other value.

Per the above specification, this document updates {{Section 4.3 of -bgp}} by
deprecating ORIGIN attribute values 1 (EGP) and 2 (INCOMPLETE), and
{{Section 5.1.1 of -bgp}}.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
