---

title: "A Profile for Forwarding Commitments (FCs)"
abbrev: "RPKI FC Profile"
category: std

docname: draft-guo-sidrops-fc-profile-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
# keyword:
# - next generation
# - unicorn
# - sparkling distributed ledger
venue:
  #  group: WG
  #  type: Working Group
  #  mail: WG@example.com
  #  arch: https://example.com/WG
  github: "BasilGuo/fc-signed-object"
  latest: "https://BasilGuo.github.io/fc-signed-object/draft-guo-fc-so.html"

author:
  -
      fullname: Ke Xu
      org: Tsinghua University
      city: Beijing
      country: China
      email: xuke@tsinghua.edu.cn
  -
      fullname: Yangfei Guo
      org: Zhongguancun Laboratory
      city: Beijing
      country: China
      email: guoyangfei@zgclab.edu.cn
  -
      fullname: Xiaoliang Wang
      org: Tsinghua University
      city: Beijing
      country: China
      email: wangxiaoliang0623@foxmail.com
  -
      fullname: Zhuotao Liu
      org: Tsinghua University
      city: Beijing
      country: China
      email: zhuotaoliu@tsinghua.edu.cn
  -
      fullname: Qi Li
      org: Tsinghua University
      city: Beijing
      country: China
      email: qli01@tsinghua.edu.cn

normative:
    RFC3779:
    RFC5652:
    RFC6268:
    RFC6481:
    RFC6485:
    RFC6487:
    RFC6488:
    X.680:
      title: "Information technology -- Abstract Syntax Notation One (ASN.1): Specification of basic notation"
      target: "https://itu.int/rec/T-REC-X.680-202102-I/en"
      date: Feb. 2021
      author:
        - ins: ITU-T
      seriesinfo: "Recommendation ITU-T X.680"
    X.690:
      title: "Information technology - ASN.1 encoding rules: Specification of Basic Encoding Rules (BER), Canonical Encoding Rules (CER) and Distinguished Encoding Rules (DER)"
      target: "[https://itu.int/rec/T-REC-X.680-202102-I/en](https://www.itu.int/rec/T-REC-X.690-202102-I/en)"
      date: Feb. 2021
      author:
        - ins: ITU-T
      seriesinfo: "Recommendation ITU-T X.690"

informative:
    RFC4271:
    RFC6480:
    RFC7908:
    RFC9582: # A Profile for Route Origin Authorizations (ROAs)

--- abstract

This document defines a Cryptographic Message Syntax (CMS) protected content type for Forwarding Commitments (FCs) objects used in Resource Public Key Infrastructure (RPKI). An FC is a digitally signed object that provides a means of verifying whether an IP address block is received from `AS a` to `AS b` and announced from `AS b` to `AS c`. When validated, an FC's eContent can be used for the detection and mitigation of route hijacking, especially providing protection for the AS_PATH attribute in BGP-UPDATE.


--- middle

# Introduction

The Border Gateway Protocol (BGP) {{RFC4271}} was designed with no mechanisms to validate the security of BGP attributes. There are two types of BGP security issues, BGP Hijacks and BGP Route Leaks {{RFC7908}}, plague Internet security.

The primary purpose of the Resource Public Key Infrastructure (RPKI) is to improve routing security.  (See {{RFC6480}} for more information.) As part of this system, a mechanism is needed to allow entities to verify that an IP address holder has permitted an AS to advertise a route along the propagation path. A Forwarding Commitment (FC) provides this function.

A Forwarding Commitment (FC) is a digitally signed object through which the issuer (the holder of an Autonomous System identifier), can authorize one or more other Autonomous Systems (ASes) as its upstream ASes or one or more other ASes as its downstream ASes. The upstream ASes, or previous ASes, mean that the issuer AS can receive BGP route updates from these ASes. The downstream ASes, or nexthop ASes, mean that the issuer AS would advertise the BGP route to these ASes.

In this propagation model, it uses a Web of Trust, i.e., the issuer AS trusts its previous ASes and authorizes nexthop ASes to propagate its received routes. Then, all nexthop ASes would also accept the routes and proceed to send them to their next hops. The relationship among them is the signed FC, which attests that a downstream AS has been selected by the directly linked upstream AS to announce the routes.

Initially, all ASes on the propagation path should sign one or more FCs independently if they want to propagate the route to its downstream ASes, and then be able to detect and filter malicious routes (e.g., route leaks and route hijacks). In addition, the FC can also attest that all ASes on a propagation path have received and selected this AS_PATH, which can be certified as a trusted path.

The FC uses the template for RPKI digitally signed objects {{RFC6488}} for the definition of a Cryptographic Message Syntax (CMS) {{RFC5652}} wrapper for the FC content as well as a generic validation procedure for RPKI signed objects.  As RPKI certificates issued by the current infrastructure are required to validate FC, we assume the mandatory-to-implement algorithms in {{RFC6485}} or its successor.

To complete the specification of the FC (see {{Section 4 of RFC6488}}), this document defines:

1.  The object identifier (OID) that identifies the FC-signed object. This OID appears in the eContentType field of the encapContentInfo object as well as the content-type signed attribute within the signerInfo structure.
2.  The ASN.1 syntax for the FC content, which is the payload signed by the BGP speaker. The FC content is encoded using the ASN.1 {{X.680}} Distinguished Encoding Rules (DER) {{X.690}}.
3.  The steps required to validate an FC beyond the validation steps specified in {{RFC6488}}.

## Requirements Language

{::boilerplate bcp14-tagged}


# The FC Content-Type

The content-type for an FC is defined as ForwardingCommitment and has the numerical value of 1.2.840.113549.1.9.16.1.(TBD).

This OID MUST appear both within the eContentType in the encapContentInfo object as well as the content-type signed attribute in the signerInfo object (see {{RFC6488}}).

# The FC eContent{#The-FC-eContent}

The content of an FC identifies a forwarding commitment that represents an AS's routing intent. Upon receiving a BGP-UPDATE message, other ASes can perform AS-path verification according to the validated FCs. An FC is an instance of ForwardingCommitmentAttestation, formally defined by the following ASN.1 {{X.680}} module:

~~~~~~
RPKI-FC-2025
  { iso(1) member-body(2) us(840) rsadsi(113549) pkcs(1)
     pkcs-9(9) smime(16) modules(0) id-mod-rpki-FC-2025(TBD) }

DEFINITIONS EXPLICIT TAGS ::=
BEGIN

IMPORTS
  CONTENT-TYPE
  FROM CryptographicMessageSyntax-2010 -- RFC 6268
    { iso(1) member-body(2) us(840) rsadsi(113549) pkcs(1)
      pkcs-9(9) smime(16) modules(0) id-mod-cms-2009(58) } ;

id-ct-FC OBJECT IDENTIFIER ::= { iso(1) member-body(2) us(840)
    rsadsi(113549) pkcs(1) pkcs-9(9) id-smime(16) id-ct(1) fc(TDB) }

ct-FC CONTENT-TYPE ::=
    { TYPE ForwardingCommitmentAttestation IDENTIFIED BY id-ct-FC }

ForwardingCommitmentAttestation ::= SEQUENCE {
    version [0]         INTEGER DEFAULT 0,
    asID                ASID,
    routingIntent       SEQUENCE (SIZE(1..MAX)) OF ROUTING-INTENT }

ROUTING-INTENT ::= SEQUENCE {
    previousASes        SEQUENCE (SIZE(1..MAX)) OF ASID,
    nexthopASes         SEQUENCE (SIZE(1..MAX)) OF ASID,
    roaASes             SEQUENCE (SIZE(0..MAX)) OF ASID
}

ASID ::= INTEGER (0..4294967295)

END
~~~~~~
{: #fig-eContentFC title="eContent of FC signed object"}

Note that this content appears as the eContent within the encapContentInfo (see {{RFC6488}}).

## version

The version number of the ForwardingCommitmentAttestation MUST be 0.

## asID

The asID field contains the AS number of the issuer AS associated with this FC.

## routingIntent

This routingIntent field contains a routing intent of the issuer AS. A routing intent typically has an upstream AS, a downstream AS, and a route set. But, for saving spaces, it can aggregate routing intents that have the same route set.

### previousASes

The previousASes field contains the upstream ASes' number of the issuer AS that can advertise the prefixes to the issuer AS.

### nexthopASes

The nexthopASes field contains the downstream ASes' number of the issuer AS that can receive advertised routes from the issuer AS.

### roaASes

The roaASes field contains a set of ASes. It associates with ROAs {{RFC9582}}. This is an optional field. When it is blank, it means that all traffic received from upstream ASes defined in previousASes field could be advertised to downstream ASes defined in nexthopASes field.


# Forwarding Commitment Validation

To validate an FC, the relying party MUST perform all the validation checks specified in {{RFC6488}} as well as the following additional FC-specific validation steps.

- The contents of the CMS eContent field MUST conform to all the constraints described in {{The-FC-eContent}}.
- The Autonomous System Identifier Delegation Extension described in {{RFC3779}} is also used in Forwarding Commitment and MUST be present in the EE certificate contained in the CMS certificates field.
- The AS identifier present in the ForwardingCommitmentAttestation eContent 'asID' field MUST be contained in the AS Identifiers present in the certificate extension.
- The Autonomous System Identifier Delegation extension MUST NOT contain "inherit" elements.
- The IP Address Delegation Extension {{RFC3779}} is not used in Forwarding Commitment, and MUST NOT be present in the EE certificate.

# Operational Consideration

Multiple valid Forwarding Commitment objects which contain the same asID could exist. In such a case, the union of these objects forms the complete routing intent set of this AS. For a given asID, it is RECOMMENDED that a CA maintains a single Forwarding Commitment. If an AS holder publishes a Forwarding Commitment, then relying parties SHOULD assume that this object is complete for that issuer AS.

If one AS receives a BGP UPDATE message with the issuer AS in the AS_PATH attribute which cannot match any routing intents of this issuer AS, it implies that there is an AS-path forgery in this message.

# Security Considerations

The security considerations of {{RFC6480}}, {{RFC6481}}, {{RFC6485}}, {{RFC6487}} and {{RFC6488}} also apply to FCs.

# IANA Considerations

## SMI Security for S/MIME Module Identifier registry

Please add the id-mod-rpki-fc-2025 to the SMI Security for S/MIME Module Identifier (1.2.840.113549.1.9.16.0) registry (https://www.iana.org/assignments/smi-numbers/smi-numbers.xml#security-smime-0) as follows:


    Decimal  |  Description                 | Specification
    ----------------------------------------------------------------
    TBD      | id-mod-rpki-fc-2025          | [RFC-to-be]


## SMI Security for S/MIME CMS Content Type registry

Please add the FC to the SMI Security for S/MIME CMS Content Type (1.2.840.113549.1.9.16.1) registry (https://www.iana.org/assignments/smi-numbers/smi-numbers.xml#security-smime-1) as follows:

    Decimal  |  Description                | Specification
    ----------------------------------------------------------------
    TBD      |  id-ct-FC                   | [RFC-to-be]

## RPKI Signed Object registry

Please add Forwarding Commitment to the RPKI Signed Object registry (https://www.iana.org/assignments/rpki/rpki.xhtml#signed-objects) as follows:

    Name       | OID                         | Specification
    ----------------------------------------------------------------
    Forwarding |                             |
    Commitment | 1.2.840.113549.1.9.16.1.TBD | [RFC-to-be]

## RPKI Repository Name Scheme registry

Please add an item for the Forwarding Commitment file extension to the "RPKI Repository Name Scheme" registry created by {{RFC6481}} as follows:

    Filename  |
    Extension | RPKI Object                   | Reference
    -----------------------------------------------------------------
         .for | Forwarding Commitment         | [RFC-to-be]

## Media Type registry

The IANA is requested to register the media type application/rpki-fc in the "Media Type" registry as follows:

      Type name: application
      Subtype name: rpki-fc
      Required parameters: N/A
      Optional parameters: N/A
      Encoding considerations: binary
      Security considerations: Carries an RPKI FC [RFC-to-be].
          This media type contains no active content. See
          Section xxx of [RFC-to-be] for further information.
      Interoperability considerations: None
      Published specification: [RFC-to-be]
      Applications that use this media type: RPKI operators
      Additional information:
        Content: This media type is a signed object, as defined
            in [RFC6488], which contains a payload of a list of
            AS identifiers (ASIDs) as defined in [RFC-to-be].
        Magic number(s): None
        File extension(s): .for
        Macintosh file type code(s):
      Person & email address to contact for further information:
        Yangfei Guo <guoyangfei@zgclab.edu.cn>
      Intended usage: COMMON
      Restrictions on usage: None
      Change controller: IETF

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
