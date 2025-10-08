---
###
title: "Geographic Attestation Results"
abbrev: "geoAR"
category: std

docname: draft-richardson-rats-geographic-results-latest
submissiontype: IETF
consensus: true
v: 3
area: Security
workgroup: RATS (if adopted)
keyword:
 - geofence
 - workload
 - soverign computing
venue:
  group: RATS
  type: Working Group
  mail: rats@ietf.org
  github: mcr/geographicresult

author:
- ins: M. Richardson
  name: Michael Richardson
  org: Sandelman Software Works
  email: mcr+ietf@sandelman.ca

normative:
  RFC9334:
  RFC9711:
  I-D.ietf-rats-ear:

informative:
  IANA.cwt:
  IANA.jwt:
  speed:
    target: "https://www.genuinemodules.com/how-fast-does-fiber-optics-travel_a6553"
    title: "How fast does fiber optics travel?"
    date: 2025-10-07
    author:
      org: "Genuine Modules"
  ptp:
    target: https://en.wikipedia.org/wiki/Precision_Time_Protocol
    title: "Precision Time Protocol"
    author:
      org: Wikipedia
    date: 2025-10-07

--- abstract

Many workloads have limitations on what geography they are allowed to operate in.
This is often due to a regulation that requires that the computation occur in a particular jurisdiction.

This document is about encoding a variety of geographical conclusions in an Attestation Result.

--- middle

# Introduction

Resolving the question of where certain computations occurs can be critical to assessing how much trust to put into the result.

Example.
Example.
Example.

{{I-D.ietf-rats-ear}} provides a framework that allows an {{RFC9334}} Verifier to return a conclusion as to geographic region for an Target Environment.

While {{RFC9711, Section 4.2.10}} provides a very good WGS84 based location claim, often very suitable as Evidence, it is not ideal for the use by Relying Parties.
There are a few reasons:

* the latitude and longitude describe a location on the Earth. The Relying Party is seldom interested in that level of detail.  It needs to know if it's in the correct place.

* the geographic position leaks significant amount of private information that is not necessary for the Relying Party to know.

* for many activities, it is the Legal Jurisdiction that matters, not the actual location.  Jurisdictions often do not have well defined concentric boundaries.  For instance, the Korean Consultate in Los Angelos is usually for Legal purposes, in Korea.  Yet, only a few meters away, possibly below the level of WGS84 accuracy, the jurisdiction would be different.

This document offers a new set of structured abstract claims that provides an evaluated view of where a Target Environment is.
The mechanism to do this appraisal may depend upon a number of factors which may be related to physical geographic position, but also include other considerations.

For instance, a claim that Target Environment is less than 1ns (as light travels in a fiber optic cable) away from another Target Enviroment whose location is known.
A typical fiber optic cable has a speed of 200,000 kilometers per second (slower than light in a vacuum to the index of refraction of the glass involved.
So if the round trip time between environments is 1ns, then the distance between Target Environments can be appraised to be within 1m of each other.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Claim definition

This claim definition goes into the EAR submods map.
Mumble mumble. AR4SI + $ear CDDL TBD.

Geographic Results can contain one or more of the following claims.

1. jurisdiction-country = ISO3361 country code.
2. jurisdiction-country-exclave = booleann
3. jurisdiction-state   = country-specific list
4. jurisdiction-state-exclave = country-specific-list
5. jurisdiction-city    = state-specific list
6. jurisdiction-city-exclave = state-specific-list
7. enclosing-exclave-country = ISO3361 country code
8. near-to = another entity+distance
9. rack-U-number = ordinal, numbered from bottom RU as 1.
10. cabinet-number = ordinal, DC specific ordering, might ignore hallway, room and floor
11. hallway-number = ordinal
12. room-numbr = string
13. floor-number = string, usually representing an integer.

There are some additional things which may be received as Evidence, but which is sometimes important to convert to  Results,  having verified some aspects. (TBD)
1. range-to-tower = designation of tower, distance-readings
2.

(NOTE: There are apparently exclaves that ar inside other countries exclaves, like Nahwa. Unclear if exclave information is even relevant, or if second order matters at all)

# CDDL Definition

~~~~
; # import rfc9711 as eat
; # import rfcXXXX as corim

$$ear-appraisal-extension //= (
    ear.geographic-result-label => geographic-result-claims
)

geographic-result-claims = non-empty<{
  ? grc.jurisdiction-country-label => iso-3361-alpha-2-country-code
  ? grc.jurisdiction-country-exclave-label => bool
  ? grc.jurisdiction-state-label => tstr .size (2..16)
  ? grc.jurisdiction-state-exclave-label => bool
  ? grc.jurisdiction-city-label => tstr .size(2..16)
  ? grc.jurisdiction-city-exclave-label => bool
  ? grc.enclosing-exclave-country-label => iso-3361-alpha-2-country-code
  ? grc.near-to-label => corim.uuid-type
  ? grc.rack-U-number-label => uint .gt 0
  ? grc.cabinet-number => uint .gt 0
  ? grc.hallway-number => uint
  ? grc.room-number => tstr .size (2..64)
  ? grc.floor-number => int
}>

ear.geographic-result-label = eat.JC<"TBD02", TBD01>

grc.jurisdiction-country-label = eat.JC<"grc.jurisdiction-country", 0>
grc.jurisdiction-country-exclave-label = eat.JC<"grc.jurisdiction-country-exclave", 1>
grc.jurisdiction-state-label = eat.JC<"grc.jurisdiction-state", 2>
grc.jurisdiction-state-exclave-label = eat.JC<"grc.jurisdiction-state-exclave", 3>
grc.jurisdiction-city-label = eat.JC<"grc.jurisdiction-city", 4>
grc.jurisdiction-city-exclave-label = eat.JC<"grc.jurisdiction-city-exclave", 5>
grc.enclosing-exclave-country-label = eat.JC<"grc.enclosing-exclave-country", 6>
grc.near-to-label  = eat.JC<"grc.near-to", 7>
grc.rack-U-number-label = eat.JC<"grc.rack-U-number", 8>
grc.cabinet-number = eat.JC<"grc.cabinet-number", 9>
grc.hallway-number = eat.JC<"grc.hallway-number", 10>
grc.room-number = eat.JC<"grc.room-number", 10>
grc.floor-number = eat.JC<"grc.floor-number", 11>

iso-3361-alpha-2-country-code = tstr .size 2
~~~~



# Security Considerations

TODO Security


# IANA Considerations

IANA is asked to allocate TBD01 from the "CBOR Web Token Claims" registry
{{IANA.cwt}}, and TBD02 (suggestion: "ear.geographic-result-claims") from the
"JSON Web Token Claims" registry {{IANA.jwt}}.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
