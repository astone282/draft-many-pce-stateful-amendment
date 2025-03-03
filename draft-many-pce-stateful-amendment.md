---
title: "Amendments to Stateful PCE Communication Protocol (PCEP)"
abbrev: "PCEP-STATEFUL-AMEND"
category: std

docname: draft-many-pce-stateful-amendment-latest
updates: 8231, 8664
submissiontype: IETF
number:
date: {DATE}
consensus: true
v: 3
area: "Routing"
workgroup: "PCE Working Group"

author:
-
    fullname: Andrew Stone (Editor)
    organization: Nokia
    email: andrew.stone@nokia.com
-
    fullname: Mike Koldychev
    organization: Ciena
    email: mkoldych@proton.me
-
    fullname: Siva Sivabalan
    organization: Ciena
    email: ssivabal@ciena.com
-
    fullname: Diego Achavel
    organization: Nokia
    email: diego.achaval@nokia.com
-
    fullname: Hari Kotni
    organization: Juniper Networks, Inc
    email: hkotni@juniper.net
-
    fullname: Samuel Sidor
    organization: Cisco
    email: ssidor@cisco.com

normative:
    RFC5440:
    RFC8231:
    RFC8664:

informative:
    I-D.draft-koldychev-pce-operational:
    RFC9603:

--- abstract

This document updates RFC8231 and RFC8664 to reflect operationalized implementations and define optimizations in the PCEP protocol.

--- middle

# Introduction

The PCEP protocol has evolved from a stateless protocol [RFC5440] to a stateful protocol [RFC8231], incorporating numerous extensions.

During interoperability testing it was observed that various implementations have implemented optimizations in the protocol.
This document serves to optimize the original procedure in [RFC8231] to optionally drop the PCReq and PCReply exchange, which
greatly simplifies implementation and optimizes the protocol.

In addition, [RFC8664] introduced extensions for Segment Routing and the encoding of segments in the ERO and RRO objects in PCEP.
This document serves as an update to [RFC8664] to permit the exclusion of the RRO object for Segment Routed paths

Note: the content in this document originated from [I-D.draft-koldychev-pce-operational] version 07, which has been branched
to become a standards updating document while [I-D.draft-koldychev-pce-operational] is to become an informational document.

## Requirements Language

{::boilerplate bcp14-tagged}

# Stateful Bringup

[RFC8231] Section 5.8.2, allows delegation of an LSP in operationally
down state, but at the same time mandates the use of PCReq, before
sending PCRpt.  In this document, we would like to make it clear that
sending PCReq is optional.

We shall refer to the process of sending PCReq before PCRpt as
"stateless bringup".  In reality, stateless bringup introduces
overhead and is not possible to enforce from the PCE, because the
stateless PCE is not required to keep any per-LSP state about
previous PCReq messages.  It was found that many vendors choose to
ignore this requirement and send the PCRpt directly, without going
through PCReq.  Even though this behavior is against [RFC8231], it
offers some advantages and simplifications, as will be explained in
this section.  This document therefore updates [RFC8231].

Even though all the major vendors today are moving to the stateful
PCE model, it does not deprecate the need for stateless PCEP.  The
key property of stateless PCEP is that PCReq messages do not modify
the state of the PCE LSP-DB.  Therefore, PCReq messages are useful
for many OAM ping/traceroute applications where the PCC wishes to
probe the network topology without having any effect on the existing
LSPs.

## Updates to RFC 8231

[RFC8231] Section 5.8.2, says "The only explicit way for a PCC to
request a path from the PCE is to send a PCReq message.  The PCRpt
message MUST NOT be used by the PCC to attempt to request a path from
the PCE."  In this document we update [RFC8231] to remove the quoted
text.

As part of the new bringup procedure, the PCC MAY delegate an empty
LSP (no ERO or empty ERO) to the PCE and then wait for the PCE to
send PCUpd, without sending PCReq.  We shall refer to this process as
"stateful bringup".  The PCE MUST support the original stateless
bringup, for backward compatibility purposes.  Supporting stateful
bringup should not require introducing any new behavior on the PCE,
because as mentioned earlier, the PCE does not modify LSP-DB state
based on PCReq messages.  So whether the PCE has received a PCReq or
not, it would process the PCRpt all the same.

An example of stateful bringup follows.  In our example the PCC
starts off by using LSP-ID of 0.  The value 0 does not hold any
special meaning, any other 16-bit value could have been used.

PCC has no LSP yet, but wants to establish a path.  PCC sends
PCRpt(R-FLAG=0, D-flag=1, OPER-FLAG=DOWN, PLSP-ID=100, LSP-ID=0,
ERO={}).

| TUNNEL      | LSP                                    |
| :---        | :----:                                 |
| PLSP-ID=100 | OLSP-ID=0, D-flag=1, OPER=DOWN, ERO={} |
{: title="Content of LSP DB after first PcRpt"}

PCC received a PCUpd from the PCE and has decided to install the
ERO={A} from that PCUpd.  PCC sends PCRpt(R-FLAG=0, D-flag=1, OPER-
FLAG=UP, PLSP-ID=100, LSP-ID=0, ERO={A}).

| TUNNEL      | LSP                                    |
| :---        | :----:                                 |
| PLSP-ID=100 | LSP-ID=0, D-flag=1, OPER=UP, ERO={A}   |
{: title="Content of LSP DB after PcUpd"}


# Use of SR-RRO and SRv6-RRO objects

[RFC8231] defines a PCRpt message which contains `<intended-path>`
known as the ERO object and `<actual-path>` known as the RRO object.
[RFC8664] defines SR-ERO and SR-RRO sub-objects for SR-TE LSPs.
[RFC9603] further defines SRv6-ERO and
SRv6-RRO sub-objects for SRv6-TE paths.

In practice RRO data is the result of signalling via a protocol such
as RSVP-TE, which allows collection of per-hop information along the
path.  The ERO and RRO values may be different as the path encoded in
the ERO may differ than the RRO such as during protection conditions
or if the ERO contains loose hops which are expanded upon.  As
Segment Routing LSP does not perform any signalling, the values of an
SR-ERO/SRv6-ERO and SR-RRO/SRv6-RRO (respectively) are in practice
the same, therefore some implementations have omitted the RRO when
reporting a SR-TE LSP while others continue to send both ERO and RRO
values.

This document updates [RFC8664] by clarifying and relaxing requirement for
both an ERO and RRO object to exist for SR-TE paths. If both ERO and RRO are present
for the same LSP, it SHOULD be interpreted as the RRO being the
actual path the LSP is taking but MAY interpret only the ERO as the
actual path.  In the absence of RRO a PCE MUST interpret the ERO as
the actual path for the LSP.  Until SR-TE introduces some form of
signaling similar to RSVP-TE, the use of RRO is discouraged for SR-TE
LSPs.

# Security Considerations

TODO

# Managability Considerations

TODO

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}
The authors would like to thank Adrian Farrel for useful review comments on [I-D.draft-koldychev-pce-operational]
which have been carried over and have been applied into this document.
