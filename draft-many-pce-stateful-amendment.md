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
    fullname: Shuping Peng
    organization: Huawei Technologies
    email: pengshuping@huawei.com
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
This document serves as an update to [RFC8664] to permit the exclusion of the RRO object for Segment Routed paths.

## Requirements Language

{::boilerplate bcp14-tagged}

# Stateful Bringup

[RFC8231] Section 5.8.2 allows delegation of an LSP in an operationally
down state, but at the same time mandates the use of PCReq
before sending PCRpt. This document clarifies that sending PCReq is optional.

The process of sending PCReq before PCRpt is referred to in
this document as "stateless bringup". In practice, stateless
bringup introduces overhead and the PcRpt sent from PCC cannot be
enforced by the PCE, because a stateless PCE is not required to
maintain any per-LSP state about previous PCReq messages. It has been
observed that many implementations choose to ignore this requirement and send
the PCRpt directly, without first sending a PCReq. Although this
behavior is not compliant with [RFC8231], it offers message processing
advantages and simplifications. As a result, this document updates [RFC8231].

The adoption of stateful PCE does not eliminate the utility of stateless PCEP.
A characteristic of stateless PCEP is that PCReq messages does require altering
the LSP path state information in the PCE. As a result, PCReq messages can be used
in scenarios such as OAM functions (e.g., ping and traceroute), where it is necessary
to probe the network topology without impacting existing LSPs and LSP state management
in the PCE.

## Updates to RFC 8231

[RFC8231] Section 5.8.2, says "The only explicit way for a PCC to
request a path from the PCE is to send a PCReq message.  The PCRpt
message MUST NOT be used by the PCC to attempt to request a path from
the PCE."  This document updates [RFC8231] to remove the quoted
text.

As part of the new bringup procedure, the PCC MAY delegate an empty
LSP (no ERO or empty ERO) to the PCE and then wait for the PCE
to send a PCUpd, without first sending a PCReq. This process is
referred to as "stateful bringup". The PCE MUST support the
original stateless bringup for backward compatibility.
Supporting stateful bringup does not require introducing new
behavior on the PCE, since, as previously noted, a PCE implementation
may choose not to modify LSP-DB state based on PCReq messages.
Therefore, regardless of whether a PCReq has been received, the PCE
processes the PCRpt in the same manner.

An example of stateful bringup follows. In this example, the PCC
starts by using an LSP-ID of 0. The value 0 does not hold any
special meaning; any other 16-bit value could have been used.

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
