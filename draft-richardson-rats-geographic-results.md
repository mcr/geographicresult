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
  obp:
    target: "https://www.iso.org/obp/ui/#search"
    title: "ISO Online Browsing Platform"
    date: 2026-03-01
    author:
       org: "International Standards Organization"
  ptp:
    target: https://en.wikipedia.org/wiki/Precision_Time_Protocol
    title: "Precision Time Protocol"
    author:
      org: Wikipedia
    date: 2025-10-07

--- abstract

Many workloads have limitations on what geography they are allowed to operate in.
This is often due to a regulation that requires that the computation occur in a particular jurisdiction.

There are many mechanisms by which Evidence of location may be created and then evaluated by a Verifier.
No matter which mechanism is appropriate for a given situation, the result of the Verification can be expressed in a similiarly defined EAT Attestation Result.

This document is therefore about encoding a variety of geographical conclusions in an Attestation Result.
In addition, one mechanism of directly creating a geographic result in the form of an Endorsement is described in an appendix.

--- middle

# Introduction

Resolving the question of where certain computations are done can be critical to assessing how much trust to put into the result.

MORE USE CASES HERE.

{{I-D.ietf-rats-ear}} provides a framework that allows an {{RFC9334}} Verifier to return a conclusion as to geographic region for an Target Environment.

While {{RFC9711, Section 4.2.10}} provides a very good WGS84 based location claim, often very suitable as Evidence, it is not ideal for the use by Relying Parties.

There are a few reasons:

* the latitude and longitude describe a location on the Earth. The Relying Party is seldom interested in that level of detail.  It needs to know if it's in the correct place.

* the geographic position leaks significant amount of private information that is not necessary for the Relying Party to know.

* for many activities, it is the Legal Jurisdiction that matters, not the actual location.  Jurisdictions often do not have well defined concentric boundaries.  For instance, the South Korean Consultate in Los Angelos is often for Legal purposes, in Korea.  Yet, only a few meters away, possibly below the level of WGS84 accuracy, the jurisdiction would be different.

* there are many other situations involving exclaves, and even exclaves inside other exclaves where the boundaries are quite complex.

This document offers a new set of structured abstract claims that provides an evaluated view of where a Target Environment is.

The mechanism to do this appraisal may depend upon a number of factors which may be related to physical geographic position, but also include other considerations.

The most obvious way is through various Global Navitation Satellite Systems (GNSS): the United States Global Positioning System (GPS), Russia's Global Navigation Satellite System (GLONASS), China's BeiDou Navigation Satellite System (BDS) and the European Union's Galileo.
These signals can be spoofed, manipulated and suppressed.
There are many environments, such as inside a (Faraday) cage in a Data Center, where GNSS signals do not reach.
Whether or not they are a good measure of location is not the subject of this document.
Rather, if a Verifier believes that information trustworthy for the purpose intended, then it may use the format described here to document it's conclusion.

There are also other radio methods, such as time of travel calculations from a mobile (LTE) tower.

An example of mechanically determination of location without radio could be a claim that a Target Environment is less than 1ns (as light travels in a fiber optic cable) away from another Target Enviroment whose location is known.
A typical fiber optic cable has a speed of 200,000 kilometers per second (slower than light in a vacuum to the index of refraction of the glass involved).
So if the round trip time between environments is 1ns, then the distance between Target Environments can be appraised to be within 1m of each other.
Other work contemplates claims like this one

Finally, one method to find out where a device is for a trusted human to go and look at it.  Perform an audit.
The result is not an Attestation Result according to {{RFC9334}}, but instead it is an Endorsement.
{{presence}} describes a protocol that could be used for a human auditor to make such an evaluation.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Claim definition

This claim definition goes into the EAR submods map.

Geographic Results can contain one or more of the following claims.

1. jurisdiction-country = ISO3361 country code.
2. jurisdiction-country-exclave = booleann
3. jurisdiction-subdivision   = country-specific list
4. jurisdiction-subdivision-exclave = country-specific-list
5. jurisdiction-city    = subdivision-specific list
6. jurisdiction-city-exclave = subdivision-specific-list
7. enclosing-exclave-country = ISO3361 country code
8. near-to = another entity+distance
9. rack-U-number = ordinal, numbered from bottom RU as 1.
10. cabinet-number = ordinal, DC specific ordering, might ignore hallway, room and floor
11. hallway-number = ordinal
12. room-numbr = string
13. floor-number = string, usually representing an integer.
14. Data Center

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
  ? grc.jurisdiction-subdivision-label => tstr .size (2..16)
  ? grc.jurisdiction-subdivision-exclave-label => bool
  ? grc.jurisdiction-city-label => tstr .size(2..16)
  ? grc.jurisdiction-city-exclave-label => bool
  ? grc.enclosing-exclave-country-label => iso-3361-alpha-2-country-code
  ? grc.near-to-label => corim.uuid-type
  ? grc.rack-U-number-label => uint .gt 0
  ? grc.cabinet-number => uint .gt 0
  ? grc.hallway-number => uint
  ? grc.room-number => tstr .size (2..64)
  ? grc.floor-number => int
  ? grc.data-center-name => tstr .size (2..64)
}>

ear.geographic-result-label = eat.JC<"TBD02", TBD01>

grc.jurisdiction-country-label = eat.JC<"grc.jurisdiction-country", 0>
grc.jurisdiction-country-exclave-label = eat.JC<"grc.jurisdiction-country-exclave", 1>
grc.jurisdiction-subdivision-label = eat.JC<"grc.jurisdiction-subdivision", 2>
grc.jurisdiction-subdivision-exclave-label = eat.JC<"grc.jurisdiction-subdivision-exclave", 3>
grc.jurisdiction-city-label = eat.JC<"grc.jurisdiction-city", 4>
grc.jurisdiction-city-exclave-label = eat.JC<"grc.jurisdiction-city-exclave", 5>
grc.enclosing-exclave-country-label = eat.JC<"grc.enclosing-exclave-country", 6>
grc.near-to-label  = eat.JC<"grc.near-to", 7>
grc.rack-U-number-label = eat.JC<"grc.rack-U-number", 8>
grc.cabinet-number = eat.JC<"grc.cabinet-number", 9>
grc.hallway-number = eat.JC<"grc.hallway-number", 10>
grc.room-number = eat.JC<"grc.room-number", 10>
grc.floor-number = eat.JC<"grc.floor-number", 11>
grc.data-center-name = eat.JC<"grc.data-center-name", 12>

iso-3361-alpha-2-country-code = tstr .size 2
~~~~

There are a number of different fields in this claim, and all of them are marked optional.
But, at least some result MUST be provided.
Which one will be needed is subject to the target usage and the needs of the Relying Party.

The explicitely geographic based jurisdiction fields are arranged in a hierarchy of values.
The outer most values are REQUIRED when any inner value is also present.
This includes the claims: country, subdivision, and city, or the equivalent exclave claims (country-exclave, subdivision-exclave, and city-exclave).

Note that the ISO 3166 term for a part of a country is called a "subdivision".
This should be understood to mean "state" (e.g., in the USA, Australia), "province" and "territory" (e.g., Canada, France).
There are no IANA maintained values for "subdivision", but the ISO 3166-2 has codes for many subdivisions, which can be seen in the ISO's Online Browsing Platform {{obp}}.

In general, subdivision lists are maintained by countries.
It is usually the case that city lists are maintained by subdivisions, and there are no lists of these in any hierarchial databases, such as the ISO might maintain for countries.
It is therefore not unsual for there to be a Paris, Texas, USA, and a Paris, France, and a London, Ontario, Canada, as well as a London, England, UK.
Equally, there are many cities called Springfield in different states of the USA.

Exclaves can exist at all levels: one part of a city might be within another city, but both are in the same country and subdivision.

Even when in an exclave, it is acceptable for a Verifier to only return the non-exclave version, hiding that an exclave is involved.
In that case, the Relying Party will receive the country, subdivision and city of where the computation is.

In general, only one of country or country-exclave, subdivision or subdivision-exclave, and city or city-exclave will be present.
When the exclave versions are present, if the Relying Party needs to indicate where the exclave is located, it may use the enclosing-exclave-country label.

The near-to-label is an arbitrary relative reference, and it refers to some other claim that the Relying Party is assumed to already know about.
The definition of "near" is up to the Verifier, but in general, it is expected that it is close enough that it is in the same jurisdiction as the other entity.

The series of claims, Data-Center, Floor-Number, Room-Number, Hallway-Number, Cabinet-Number and Rack-U-Number form a partial hierarchy designed to identify where a piece of equipment is.
This set of claims is more useful when a location Endorsement is created by an auditor, such as through the process described in {{presence}}.

Not all data centers are numbered in the same way.
For some, the Room-Number implies the Floor-Number.
For others, the Hallway-Number implies both room and floor, and for still others, the Cabinet-Number is unique per building.

Rack-U numbers refer to the system within a cabinet, with the bottom most position labelled as 1.  This accomodates cabinets of varying heights and capacities.



# Security Considerations

These considerations do not cover the protocol described in {{presence}}.

This document describes attributes that would go into an EAT format Attestation Result (EAR).
This is an artifact that is communicated from a Verifier to a Relying Party.
The threat model for this artifact is governed by a typical "CIA" list: Confidentiality, Integrity and Availability.

## Confidentiality Threats

This artifact contains abtracted information about a location.
The location could include where a person operating the computation is, such as if this describes the location of a person's smartphone, in which case the location, even abstracted would be PII.
It is almost always preferable for this artifact to remain private, however the integrity of the artifact is not compromised if that is not the case.

## Integrity Threats

The most important threat is that claims could be corrupted or intentionally changed as they are transfered from Verifier to Relying Party.
EAR are signed objects, and the signature from the Verifier maintains the integrity of that object.
The claims present in this document will most often be combined with other claims in the same artifact.

When an EAT format Endorsement is created by an auditor, the auditor signs the artifact.
The Endorsement may be provided to a Verifier through out of band means, or it can be stored by the Attesting Environment, and carried through another protocol from Attester to Verifier.

## Availability Threats

This artifact is the result of a calculation or audit that establishes a result.
Once created, it can be valid for a period of time from seconds (GNSS result on a smartphone) to months (human audit of a data center).

Distruptions to the mechanism of location calculation do not make the result of the previous calculation invalid.
However, it is possible that such an attack could make subsequent calculations infeasible.  For instance, the GNSS signals can easily be jammed.

# IANA Considerations

IANA is asked to allocate TBD01 from the "CBOR Web Token Claims" registry
{{IANA.cwt}} (from a Specification Required Integer range), and TBD02 (suggestion: "ear.geographic-result-claims") from the "JSON Web Token Claims" registry {{IANA.jwt}}.


--- back

# Proof of Presense {#presence}

Hello.

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
