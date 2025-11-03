---
title: "Scrubbing BGP ORIGIN Attribute"
category: std

docname: draft-marenamat-idr-scrub-bgp-origin-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
updates: 4271, 7606
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
  RFC7606: bgp-update-error-handling

informative:
  RFC904: egp
  RFC1105: bgp-experimental
  RFC1163: first-bgp-with-origin-attribute
  RFC1771: legacy-bgp
  RFC1772: bgp-is-better-than-egp
  RFC8205: bgpsec
  bird-bgp-commit:
    title: "BIRD commit: adding real comparison of BGP routes (inspired by the Cisco one)"
    target: https://gitlab.nic.cz/labs/bird/-/commit/56a2bed46bf7713cd773b0fd0c097bcfc6345cc1
    date: 2000-04-17
    author:
      ins: M. Mareš
      name: Martin Mareš
  rewriting-bgp-origin:
    title: Rewriting BGP Origin
    target: https://ripe91.ripe.net/programme/meeting-plan/sessions/30/GJ8PFJ/
    date: 2025-10-22
    author:
      ins: J. Bensley
      name: James Bensley
      org: Inter.link
#  RFC6472: as-set-not-use
#  RFC9774: as-set-deprecation

--- abstract

The BGP Origin attribute in its original meaning has been out of use for years.
Yet, the BGP Origin attribute has high priority in the best route
selection algorithm, right after the AS Path length, and it's being used
inconsistently over the Internet to manipulate the route preference.

This document updates RFC 4271 and RFC 7606 by making the BGP Origin attribute
half-optional and explicitly allowing its scrubbing to zero (IGP).

--- middle

# Introduction

Origins of the BGP Origin attribute stem from times when BGP was not the only
full internet routing protocol, and was slowly replacing EGP ({{-egp}}). First
seen in the first published BGP draft as the Direction field in the UPDATE
message (see {{Section 3.4 of -bgp-experimental}}), later refined into the
Origin attribute (see {{Section 5 of -first-bgp-with-origin-attribute}})
with just three values: IGP, EGP and Incomplete.

While the attribute itself has been long established, even in 1995 in {{-legacy-bgp}},
it is not being formally specified to be used for best route selection. Instead,
using the origin attribute was suggested in {{-bgp-is-better-than-egp}} to be used
as an indicator of better route, together with AS Path Length and more criteria.

It's hard to dig through the archives to find out what happened when and why,
but it's safe to assume that most of the BGP implementations of that time
simply used a very similar tie breaking algorithm without formal specification,
as documented e.g. in {{bird-bgp-commit}}. With that, the tie breaking in its
current form, specified in {{Section 9.1.2.2 of -bgp}}, was most probably
simply copied from the existing stuff out there.

In the year 2006, there might still have been some remnants of EGP infrastructure,
and deprecating the BGP Origin attribute altogether wasn't probably a good idea then.

In September 2025, a short look {{rewriting-bgp-origin}} into the global routing table
shows that about 9% of the DFZ routing table bears BGP Origin value INCOMPLETE
and under 1% there are still some routes with BGP Origin value EGP. With these routes,
several transit providers rewrite the BGP Origin value to IGP to raise their priority
in the best route selection. Rather curiously, there are even several IPv6
routes with BGP Origin value EGP.

Therefore, this document makes the first needed steps to deprecate the BGP
Origin attribute in the future by updating the rules for its origination,
relaxing its handling and deprecating all other its values than zero.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Tolerance of missing or undefined BGP Origin attribute

The attribute is considered malformed if its length is not 1.

Per the above specification, this document updates
{{Section 7.1 of -bgp-update-error-handling}}
by not considering malformed an ORIGIN attribute with
an undefined value.

Unless explicitly configured by a network operator to do otherwise,
if the ORIGIN attribute is present but not malformed, BGP speakers MUST
accept such UPDATE message and SHOULD replace the ORIGIN attribute
by a value of 0 (IGP).

Unless explicitly configured by a network operator to do otherwise,
if the ORIGIN attribute is absent, BGP speakers MUST
accept such UPDATE message and SHOULD add the ORIGIN attribute
with a value of 0 (IGP).

Per the above specification, this document updates
{{Section 4.3 of -bgp}}, the Paths Attributes part, a),
{{Section 5 of -bgp}}, {{Section 5.1.1 of -bgp}},
{{Section 6.3 of -bgp}}
by not considering an ORIGIN attribute mandatory, but
optional. Though for transition period it is RECOMMENDED
to add the attribute for compatibility with legacy BGP speakers.

# Update to the BGP Origin attribute values

The values of the BGP Origin attribute are specified in
{{Section 4.3 of -bgp}}, the Path attribute part, a) ORIGIN (Type Code 1).

Unless explicitly configured by a network operator to do otherwise,
BGP speakers MUST NOT advertise BGP UPDATE messages with the ORIGIN
attribute containing any other value than 0 (IGP), including manually
configured static routes.

Additionally, BGP speakers SHOULD rewrite any ORIGIN attribute value
to 0 (IGP) if they contain any other value.

Per the above specification, this document updates {{Section 4.3 of -bgp}} by
deprecating ORIGIN attribute values 1 (EGP) and 2 (INCOMPLETE), and
{{Section 5.1.1 of -bgp}}.

# Security Considerations

Originating a route with a non-zero value of the ORIGIN attribute makes
the route prone to unwanted prioritization by the intermediate AS's.
Scrubbing the attribute removes this possible traffic redirection problem.
All the results achievable by manipulating this attribute may be instead achieved
by other techniques. Most notably, stuffing the AS Path by own AS Number is
directly supported by the Secure Path attribute as of {{Section 3.1 of -bgpsec}},
being more secure and with higher priority than the ORIGIN attribute.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

The authors wish to thank James Bensley, Gert Doering, Wolfgang Tremmel,
Ondřej Zajíček, and Alexander Zubkov for valuable comments, suggestions and
discussions on the topic and text of the document.
