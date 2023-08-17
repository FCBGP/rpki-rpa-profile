---

title: "A Profile for Forwarding Commitments (FCs)"
abbrev: "Forwarding Commitments"
category: std

docname: draft-guo-fc-so-latest
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
      fullname: Xiaoliang Wang
      org: Tsinghua University
      city: Beijing
      country: China
      email: wangxiaoliang0623@foxmail.com
  -
      fullname: Yangfei Guo
      org: Zhongguancun Labratory
      city: Beijing
      country: China
      email: guoyangfei@zgclab.edu.cn
  -
      fullname: Jiangou Zhan
      org: Tsinghua University
      city: Beijing
      country: China
      email: "904542587@qq.com" # TODO: use your edu.cn email
  -
      fullname: Jianping Wu
      org: Tsinghua University
      city: Beijing
      country: China
      email: jianping@cernet.edu.cn

normative:
    RFC3779:
    RFC4271:
    RFC5652:
    RFC6480:
    RFC6485:
    RFC6488:
    RFC7908:

informative:
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


--- abstract

This document defines a standard profile for Forwarding Commitment (FC) used in Resource Public Key Infrastructure (RPKI). An FC is a digitally signed object that provides a means of verifying that an IP address prefix is announced from `AS a` to `AS b`. When validated, an FC's eContent can be used for the detection and mitigation of route hijacking and provide protection for the AS_PATH attribute in BGP-UPDATE.


--- middle

# Introduction


The Border Gateway Protocol (BGP) {{RFC4271}} was designed with no mechanisms to validate the security of BGP attributes. There are two types of BGP security issues, BGP Hijacks and BGP Route Leaks {{!RFC7908}}, plagues Internet security.

The primary purpose of the Resource Public Key Infrastructure (RPKI) is to improve routing security.  (See {{RFC6480}} for more information.) As part of this system, a mechanism is needed to allow entities to verify that an AS has been given permission by an IP address holder to advertise a route along the propagation path. An FC provides this function.

Forwarding Commitment (FC) is a signed object that binds the IP prefix with AS and its next hops, eventually, it could compose and protect the path of BGP-UPDATE propagation. It uses a Web of Trust in this propagation model, i.e., the originator AS trusts its next hop ASes and authorizes them to propagate its own prefix. Then, all next-hop ASes would also accept the prefix and proceed to send it to their next hops. The relationship among them is the signed FC, which attests that a downstream AS has been selected by the directly linked upstream AS to announce the IP prefix. For an IP prefix, all ASes on the propagation path should sign such an FC independently, and then be able to detect and filter malicious routes (e.g., route leaks and route hijacks). In addition, the FC can also attest that all ASes on a propagation path have received and selected this AS_PATH, which can be certified as a trusted path.

The FC uses the template for RPKI digitally signed objects {{RFC6488}} for the definition of a Cryptographic Message Syntax (CMS) {{RFC5652}} wrapper for the FC content as well as a generic validation procedure for RPKI signed objects.  As RPKI certificates issued by the current infrastructure are required to validate FC, we assume the mandatory-to-implement algorithms in {{RFC6485}} or its successor.

To complete the specification of the FC (see {{Section 4 of RFC6488}}), this document defines:

1.  The object identifier (OID) that identifies the FC signed object. This OID appears in the eContentType field of the encapContentInfo object as well as the content-type signed attribute within the signerInfo structure.
2.  The ASN.1 syntax for the FC content, which is the payload signed by the BGP speaker. The FC content is encoded using the ASN.1 {{X.680}} Distinguished Encoding Rules (DER) {{X.690}}.
3.  The steps required to validate an FC beyond the validation steps specified in {{RFC6488}}.

## Requirements Language

{::boilerplate bcp14-tagged}


# The FC Content-Type

The content-type for an FC is defined as ForwardingCommitment and has the numerical value of 1.2.840.113549.1.9.16.1.(TBD).

This OID MUST appear both within the eContentType in the encapContentInfo object as well as the content-type signed attribute in the signerInfo object (see {{RFC6488}}).

# The FC eContent

The content of an FC identifies a forwarding commitment and forwarding binding that an AS announces to other nodes upon receiving a BGP-UPDATE message. Other on-path ASes can validate the FC and perform path verification for traffic forwarding based on the AS-path information. Off-path ASes can utilize this FC for collaborative filtering. An FC is an instance of ForwardingCommitmentAttestation, formally defined by the following ASN.1 {{X.680}} module:

~~~~~~
RPKI-FC-2023
  { iso(1) member-body(2) us(840) rsadsi(113549) pkcs(1)
     pkcs-9(9) smime(16) modules(0) id-mod-rpki-FC-2023(TBD) }

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
    originatorASID      ASID,
    prefix              SEQUENCE (SIZE(1)) OF Prefix,
    fc ForwardingCommitment }

ASID ::= INTEGER (0..4294967295)

Prefix ::= SEQUENCE {
    afi                 AFI,
    address             IPAddress,
    prefixLength        INTEGER (SIZE(0..128)}

AFI ::= OCTET STRING (SIZE(2))

IPAddress ::= BIT STRING (SIZE(0..128))

ForwardingCommitment ::= SEQUENCE {
    id                  BIT STRING
    signature           BIT STRING }

END
~~~~~~
{: #fig-eContentFC title="eContent of FC signed object"}

Note that this content appears as the eContent within the encapContentInfo (see {{RFC6488}}).

## version

The version number of the ForwardingCommitmentAttestation MUST be 0.

## originatorASID

The originatorASID field contains the AS number of the originator AS associated with this FC in the BGP-UPDATE message.

## Type Prefix

Within the Prefix structure, the prefix field encodes the set of IP address prefixes announced by the originator AS in AS_PATH.

### Element afi

Within the Prefix structure, afi contains the Address Family Identifier of an IP address family. This specification only supports IPv4 and IPv6. Therefore, afi MUST be either 0001 or 0002.

### Element address

The address field contains the IP address.

### Element prefixLength

The prefixLength field contains the length of the IP address prefix.

## Type ForwardingCommitment

Within the ForwardingCommitment structure, the fc field encodes the forwarding commitment generated by this AS and will be validated by other AS.

### Element id

The id field contains the hash of the current AS-path in the associated BGP-UPDATE plus the next hop AS and above prefix filed.

### Element signature

The signature field is a signature signed by the BGP speaker who issues this FC.

# FC Validation


Only when finished the validation of the FC object will a relying party sign a new FC to announce a trusted and selected routing announcement. To validate an FC, the relying party MUST perform all the validation checks specified in {{RFC6488}} as well as the following additional FC-specific validation step.

- The IP Address Delegation extension {{RFC3779}} is present in the end-entity (EE) certificate (contained within the FC), and the IP address prefix in the FC payload is contained within the set of IP addresses specified by the EE certificate's IP Address Delegation extension.

- The EE certificate's IP Address Delegation extension MUST NOT contain "inherit" elements described in {{RFC3779}}.

- The Autonomous System Identifier Delegation Extension described in {{RFC3779}} is also used in Forwarding Commitment and MUST be present in the EE certificate.



# FC based BGP AS_PATH Verification {#fc-verification}

## FC Generation

The FC generation procedure follows the BGP-UPDATE. When one AS establishes a connection with its peer, it announces its route to peers. It SHOULD generate FC for each IP prefix in the BGP-UPDATE. For example, there are 3 ASes and the topology of them is in a line as {{fig-example}} shows.

~~~~~~
+----------+     +----------+     +----------+
| AS 65536 | --- | AS 65537 | --- | AS 65538 |
+----------+     +----------+     +----------+
~~~~~~
{: #fig-example title="A topology example of 3 AS"}

Suppose that the simplified RIB table of AS 65536 is as follows, which means 1 BGP-UPDATE is announced from AS 65538 to AS 65537 and 1 BGP-UPDATE is announced from AS 65537 to AS 65536. One FC will be generated for prefix 2001:db8:c::/48 when BGP-UPDATE is announced from AS 65538 to AS 65537. But two FCs will be generated when BGP-UPDATE is announced from AS 65537 to AS 65536, one is for the prefix 2001:db8:c::/48 and the other is for the prefix 2001:db8:b::/48.

~~~~~~
Network           Next Hop        Path
2001:db8:a::/48   ::              i
2001:db8:b::/48   2001:db8:a::1   65537 i
2001:db8:c::/48   2001:db8:a::1   65537 65538 i
~~~~~~
{: #fig-rib-of-AS65536 title="Simplified RIB table of AS 65536"}

For BGP-UPDATE announced from AS 65538 to AS 65537, it fills the eContent of FC signed object as follows:

~~~~~~
FC_{65538, 65537}:
  version: 0
  orginatorASID: 65538
  prefix:
    afi: 2
    address: 2001:db8:c::
    prefixLength: 48
  ForwardingCommitment:
    id: HASH({65538}, 65537, 2001:db8:c::/48)
    signature: SIGNATURE1
~~~~~~

For BGP-UPDATE announced from AS 65538 to AS 65537, it fills the eContent of FC signed objects as follows:

~~~~~~
FC_{65538, 65537, 65536}:
  version: 0
  orginatorASID: 65538
  prefix:
    afi: 2
    address: 2001:db8:c::
    prefixLength: 48
  ForwardingCommitment:
    id: HASH({65538, 65537}, 65536, 2001:db8:c::/48)
    signature: SIGNATURE2

FC_{65537, 65536}:
  version: 0
  orginatorASID: 65537
  prefix:
    afi: 2
    address: 2001:db8:b::
    prefixLength: 48
  ForwardingCommitment:
    id: HASH({65537}, 65536, 2001:db8:b::/48)
    signature: SIGNATURE3
~~~~~~

These FC signed objects MUST store in the RPKI repository as soon as possible after the BGP announcement.

## FC Verification and AS_PATH Verification

When one AS receives one BGP-UPDATE, it MUST do as usual: filter the BGP route using its local policy, scratch the BGP route to its Route Information Base (RIB) table, generate Forwarding Information Base (FIB) table, and send it out as per the routing policy.

When synchronizing the FC signed objects from the RPKI repository, the BGP speaker MUST find FCs according to the ForwardingCommitment-id: HASH(current_AS_PATH, nextHopAS, prefix) in its RIB. If it finds one matched, the BGP speaker verifies the ForwardingCommitment-signature in the RPKI repository.

It SHOULD verify all FCs along the AS_PATH for AS_PATH verification. For example, AS 65536 should verify FC_{65538, 65537} and FC_{65538, 65537, 65536} for prefix 2001:db8:c::/48 in {{fig-example}}. If all FCs checks passed, the AS_PATH would be a valid one.

# Security Considerations

## Route Aggregation

There is no mechanism in FC that could protect one prefix that has been aggregated along the route propagation.

Maybe a MinLength like MaxLength in ROA could help to mitigate the effects of route aggregation. This field means it permits who to aggregate and aggregate to which degree.


# IANA Considerations

## SMI Security for S/MIME CMS Content Type (1.2.840.113549.1.9.16.1)

 Please add the id-mod-rpki-fc-2023 to the SMI Security for S/MIME Module Identifier (1.2.840.113549.1.9.16.0) registry (https://www.iana.org/assignments/smi-numbers/smi-numbers.xml#security-smime-0) as follows:

~~~~~~
Decimal    Description                   Specification
---------------------------------------------------------------------
TBD        id-mod-rpki-fc-2023           [RFC-to-be]
~~~~~~

## SMI Security for S/MIME CMS Content Type registry

Please add the FC to the SMI Security for S/MIME CMS Content Type (1.2.840.113549.1.9.16.1) registry (https://www.iana.org/assignments/smi-numbers/smi-numbers.xml#security-smime-1) as follows:

~~~~~~
Decimal     Description                  Specification
---------------------------------------------------------------------
TBD         id-ct-FC                     [RFC-to-be]
~~~~~~

## RPKI Signed Object registry

Please add Forwarding Commitment to the RPKI Signed Object registry (https://www.iana.org/assignments/rpki/rpki.xhtml#signed-objects) as follows:

~~~~~~
Name        OID                          Specification
---------------------------------------------------------------------
Forwarding
Commitment  1.2.840.113549.1.9.16.1.TBD  [RFC-to-be]
~~~~~~

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
