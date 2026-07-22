---
title: "Global Opaque Block for IOAM Pre-allocated Trace Option"
abbrev: "IOAM GOB"
category: exp

docname: draft-mayer-ioam-gob-latest
submissiontype: independent
ipr: trust200902
area: AREA
workgroup: WG Working Group
keyword:
  - IOAM
  - IPv6
  - In-band telemetry
  - Metadata
venue:
  group: EIP
  type: SIG
  mail: eip@cnit.it
  arch: http://postino.cnit.it/cgi-bin/mailman/private/eip/
  github: eip-home/ioam-gob
  latest: https://eip-home.github.io/ioam-gob/draft-mayer-ioam-gob.html

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
  - name: "Andrea Mayer"
    ins: "A. Mayer"
    organization: "Univ. of Rome Tor Vergata / CNIT / Common Net"
    email: "andrea.mayer@uniroma2.it"
  - name: "Stefano Salsano"
    ins: "S. Salsano"
    organization: "Univ. of Rome Tor Vergata / CNIT"
    email: "stefano.salsano@uniroma2.it"
  - name: "Justin Iurman"
    ins: "J. Iurman"
    organization: "University of Liege"
    email: "justin.iurman@uliege.be"
  - name: "Diego R. Lopez"
    ins: "D. R. Lopez"
    organization: "Telefonica"
    email: "diego.r.lopez@telefonica.com"

normative:
  RFC8200:
  RFC9197:
  RFC9486:
  RFC2119:
  RFC8174:
  IANA-ioam-types:
    title: "IOAM Data Field Types"
    author:
      - name: "IANA"
    format:
      HTML: "https://www.iana.org/assignments/ioam/ioam.xhtml#data-field-types"
  IANA-ipv6-parameters:
    title: "Internet Protocol Version 6 (IPv6) Parameters"
    author:
      - name: "IANA"
    format:
      HTML: "https://www.iana.org/assignments/ipv6-parameters/ipv6-parameters.xhtml"

informative:
  draft-eip-arch:
    title: "Extensible In-band Processing (EIP) Architecture and Framework"
    author:
      - name: "Stefano Salsano"
        ins: "S. Salsano"
      - name: "Hesham ElBakoury"
        ins: "H. ElBakoury"
      - name: "Diego R. Lopez"
        ins: "D. R. Lopez"
    date: 2026
    refcontent: "Internet-Draft draft-eip-arch"
    format:
      HTML: "https://datatracker.ietf.org/doc/draft-eip-arch/"
  salsano25cscn:
    title: "Integrating Extensible In-Band Processing (EIP) into the IOAM Framework: A Unified Approach to In-Packet Telemetry and Metadata"
    author:
      - name: "Stefano Salsano"
      - name: "Andrea Mayer"
      - name: "Giulio Sidoretti"
      - name: "Pierpaolo Loreti"
      - name: "Lorenzo Bracciale"
      - name: "Hesham ElBakoury"
      - name: "Diego R. Lopez"
    date: 2025
    refcontent: "IEEE CSCN 2025 Conference"
  mayer26noms:
    title: "Integrating the Global Opaque Block into the IOAM Framework: A Unified Approach to In-Packet Telemetry and Metadata"
    author:
      - name: "Andrea Mayer"
      - name: "Stefano Salsano"
      - name: "Giulio Sidoretti"
      - name: "Pierpaolo Loreti"
      - name: "Lorenzo Bracciale"
      - name: "Hesham ElBakoury"
      - name: "Diego R. Lopez"
    date: 2026
    refcontent: "IEEE/IFIP NOMS 2026"

--- abstract

Extensible metadata carried within the IOAM Pre-allocated Trace Option
(PTO) can support packet-level, flow-level, or path-level processing
beyond per-node trace data. This document defines the Global Opaque
Block (GOB), an extension to the IOAM PTO that introduces a single
pre-allocated global metadata region placed at the tail of the option,
after the pre-allocated node data space. Since the PTO node data list
is populated from the tail of the node data space toward its head,
under the control of the RemainingLen field, legacy IOAM transit nodes
can never reach the GOB region: the extension is transparent to such
nodes by construction, and only the decapsulating node or collector
needs to be GOB-aware. The GOB carries an explicit length and schema
identifier in a trailer word, preserves the pre-allocated PTO
processing model, and can be used to transport Extensible In-band
Processing (EIP) Information Elements or other structured metadata
formats.

--- middle

# Introduction

The IOAM Pre-allocated Trace Option (PTO) defined in {{RFC9197}} and
{{RFC9486}} provides a structured way to carry in-band operational and
telemetry information along a packet path. In the PTO model, the
encapsulating node pre-allocates space for node data that can be filled
by IOAM-capable nodes along the path.

This document defines an extension to the IOAM PTO called the Global
Opaque Block (GOB). The GOB is a global metadata region that appears
once in the PTO, at the tail of the option, after the pre-allocated
node data space. The GOB is identified by a presence bit in the PTO
header and carries its own length and schema fields in the last
4-octet word of the option, referred to as the GOB Trailer.

The GOB enables the carriage of structured global metadata within the
IOAM PTO, while preserving the existing PTO processing model for
per-node trace data. In particular, the GOB can carry Extensible
In-band Processing (EIP) Information Elements or other schema-driven
metadata formats, without modifying the semantics of the node data list.

The placement of the GOB at the tail of the option makes it transparent
to legacy IOAM transit nodes. In the PTO, the node data list is
populated from the tail of the pre-allocated node data space toward its
head, at offsets derived from the RemainingLen field (Section 4.4.1 of
{{RFC9197}}). A compliant legacy node therefore never writes beyond the
node data space and cannot reach the GOB region that follows it. Only
the decapsulating node or the collector needs to be GOB-aware. This
property is analyzed in detail in {{backward-compatibility}}.

The GOB is pre-allocated by the encapsulating node. Transit nodes MAY
read or update the GOB payload, subject to the bounds established at
encapsulation time, but they MUST NOT alter the overall GOB size or the
layout of the enclosing PTO option.

This document specifies the GOB format, length and alignment rules, and
processing behavior for encapsulating, transit, and egress or collector
nodes.

An implementation and experimental evaluation of the proposed GOB approach
within the Linux IOAM framework are described in [mayer26noms].

# Conventions and Definitions

{::boilerplate bcp14-tagged}

The following terms are used in this document:

GOB:
: Global Opaque Block. A structured global metadata region inserted into
  the IOAM PTO.

GOB Trailer:
: The 4-octet trailer of the GOB, carried as the last word of the IOAM
  option and containing the GOB Len and Schema fields.

GOB Payload:
: The payload that immediately precedes the GOB Trailer. Its size is
  determined by the GOB Len field.

Node Data Space:
: The pre-allocated region that follows the PTO fixed header and hosts
  the Node Data List. Its size is set at encapsulation time via the
  RemainingLen field.

PTO:
: The IOAM Pre-allocated Trace Option defined in {{RFC9197}} and
  {{RFC9486}}.

Node Data List:
: The sequence of pre-allocated per-node data slots defined by the PTO
  format.

Ingress Node:
: The encapsulating node that inserts the IOAM PTO and, when present,
  allocates and initializes the GOB.

Transit Node:
: An IOAM-capable node along the path that may process the PTO and may
  read or update the GOB payload.

Egress or Collector Node:
: A node that removes, terminates, exports, or otherwise interprets the
  IOAM information carried in the packet.

Schema:
: A 24-bit identifier that specifies how the GOB Payload is to be
  interpreted.

# Design Overview

The GOB extends the IOAM PTO with a global metadata area that is
logically distinct from the per-node node data list. It is intended for
metadata that applies to the packet, flow, or path as a whole, rather
than to a specific transit node.

The GOB appears at most once in a PTO option. When present, it is
placed at the tail of the option, after the pre-allocated node data
space. The GOB consists of a variable-length payload followed by a
fixed 4-octet trailer, which is the last word of the IOAM option and
carries the GOB length and schema identifier. The GOB is located from
the end of the option, using the IOAM option length; any padding used
to align the enclosing Hop-by-Hop Options header lies outside the IOAM
option and does not affect GOB location.

The GOB preserves the pre-allocated nature of the PTO model. The ingress
node allocates the GOB space at encapsulation time and determines its
size, its schema, and the overall PTO layout. GOB-aware transit nodes
MAY modify the contents of the GOB payload, but MUST NOT modify the GOB
Trailer. Legacy transit nodes require no modification: they process the
node data space according to {{RFC9197}} and never access the GOB.

The GOB is intended to support structured metadata formats. One relevant
use case is the carriage of Extensible In-band Processing (EIP)
Information Elements, but the mechanism is not restricted to EIP and may
be used with other schema-driven payload formats.

The addition of the GOB does not change the semantics of the per-node
node data list. The GOB complements the PTO by adding a global opaque
area while leaving the existing per-node trace structure intact.


# GOB Format

## PTO Layout with GOB

When the Global Opaque Block (GOB) is present, the IOAM
Pre-allocated Trace Option (PTO) is extended by appending a single GOB
at the tail of the option, after the pre-allocated Node Data Space.

The resulting layout is:

1. PTO fixed header
2. Node Data Space (hosting the Node Data List)
3. GOB Payload
4. GOB Trailer
5. Tail padding, if required to satisfy alignment of the enclosing
   Hop-by-Hop Options header; such padding is outside the IOAM option

The GOB is indicated by the G bit in the PTO fixed header. If the G bit
is clear, no GOB is present and PTO processing follows the base PTO
format.

The GOB appears at most once in a PTO option.

## G Bit and Ver Field {#g-bit-ver}

The presence of the GOB is signaled in the last octet of the second
word of the PTO fixed header, defined as Reserved in {{RFC9197}}. When
the GOB extension is in use, this octet is structured as 5 reserved
bits, followed by a 2-bit Ver field and by the G bit in the least
significant position.

G bit:
: When set to 1, a GOB is present at the tail of the option. When set
  to 0, no GOB is present and the option follows the base PTO format.

Ver:
: A 2-bit field carrying the version of the GOB extension. This
  document defines version 0. An encapsulating node inserting a GOB
  MUST set Ver to 0. A GOB-aware node that receives a PTO with the G
  bit set and an unsupported Ver value MUST NOT parse or modify the
  GOB.

The remaining 5 bits are reserved: they MUST be set to zero by the
encapsulating node and ignored on receipt.

## GOB Trailer Fields

The GOB Trailer is 4 octets long and contains the following fields:

GOB Len:
: An 8-bit field that specifies the size of the GOB Payload in units of
  4 octets.

Schema:
: A 24-bit field that identifies the format of the GOB Payload. The
  value 0xFFFFFF indicates that no schema is associated with the GOB;
  in this case, the GOB Payload MUST NOT be interpreted. This mirrors
  the corresponding rule for the Opaque State Snapshot Schema ID in
  {{RFC9197}}. The structure of the Schema value space is defined in
  {{schema-semantics}}.

The total size of the GOB is therefore:

~~~
GOB_PLEN = GOB_Len * 4
GOB_SIZE = 4 + GOB_PLEN
~~~

where the 4 additional octets correspond to the GOB Trailer itself. The
GOB Payload consists of the `GOB_PLEN` octets that immediately precede
the GOB Trailer.

The Schema field allows the GOB Payload to be interpreted according to
a structured format. One important use case is the carriage of
Extensible In-band Processing (EIP) Information Elements, but the GOB
is not limited to EIP and may carry other schema-driven metadata
formats.

## Byte-Level Layout

The following figure shows the byte-level layout of the IOAM PTO with
the GOB present.

~~~
Offset      0                   1                   2                   3
            0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     40    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |   Next Header |  Hdr Ext Len  |   PadN Opt    | PadN OptLen   |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |    Opt Type   |  IOAM Opt Len |   Reserved    |   IOAM Type   |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |  Namespace-ID  (16b)          |NodeLen  | Flags |RemainingLen |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |         IOAM Trace-Type (24b)                 |Reserved |Ver|G|
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                                                               |
           ~                          Node Data List                       ~
           |                                                               |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                                                               |
           ~                           GOB Payload                         ~
           |                                                               |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           | GOB Len (8b)  | Schema (24b)                                  |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                           Padding                             |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: title="IOAM PTO with Global Opaque Block (GOB) at the tail"}

More explicitly, the internal structure is:

~~~
Offset      0                   1                   2                   3
            0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     40    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |   Next Header |  Hdr Ext Len  |   PadN Opt    | PadN OptLen   |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |    Opt Type   |  IOAM Opt Len |   Reserved    |   IOAM Type   |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+<-+
           |  Namespace-ID  (16b)          |NodeLen  | Flags |RemainingLen | |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ (A)
           |         IOAM Trace-Type (24b)                 |Reserved |Ver|G| |
     X     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+<-+
           |                                                               | |
           |                       Node Data List [0]                      | |
           |                                                               | |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ D
           |                                                               | a
           |                       Node Data List [1]                      | t
           |                                                               | a
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           ~                              ...                              ~ S
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ p
           |                                                               | a
           |                      Node Data List [n-1]                     | c
           |                                                               | e
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ |
           |                                                               | |
           |                       Node Data List [n]                      | |
           |                                                               | |
     Y     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+<-+
           |                                                               | G
           ~                           GOB Payload                         ~ O
           |                                                               | B
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+<-+
  GOB_TRL  | GOB Len (8b)  | Schema (24b)                                  |
     Y'    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |   PadN Opt    | PadN OptLen   |  Padding 16 bits              |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 ALG(Y',8) \_________________________8-byte aligned ________________________/
   = Y''
~~~
{: title="Detailed PTO layout with Node Data Space and tail GOB"}

The Node Data Space begins immediately after the PTO fixed header, at
the same position as in the base PTO format: this is the write anchor
used by all nodes, including legacy ones, to derive the node data
recording offset from RemainingLen. The GOB Payload immediately follows
the Node Data Space. The GOB Trailer is the last 4-octet word of the
IOAM option. Any final tail padding is added only to satisfy alignment
requirements of the enclosing Hop-by-Hop Options header and lies
outside the IOAM option.


# Length and Alignment Rules

This section defines the length relations used when the GOB is present
in the IOAM PTO.

## GOB Length

The GOB Payload length is determined by the GOB Len field:

~~~
GOB_PLEN = GOB_Len * 4
~~~

The total GOB size, including the fixed 4-octet GOB Trailer, is:

~~~
GOB_SIZE = 4 + GOB_PLEN
~~~

The GOB Len field is set by the ingress node at encapsulation time.

Transit nodes MAY modify the content of the GOB Payload, but MUST NOT
modify the GOB Trailer and therefore MUST NOT alter the total GOB
size.

## Internal PTO Offsets {#internal-pto-offsets}

Let:

* `X` be the offset of the start of the Node Data Space, immediately
  following the PTO fixed header;
* `Y` be the offset immediately following the Node Data Space;
* `Y'` be the offset immediately following the GOB Trailer, that is,
  the end of the IOAM option;
* `Y''` be the IOAM option length rounded up to the next multiple of
  8 octets;
* `RemainingLen_init` be the initial value of the RemainingLen field,
  set by the ingress node at encapsulation time.

Then:

~~~
X   = (sizeof(IOAM_TLV) - 2) + sizeof(IOAM_PTO_HDR)
Y   = X + RemainingLen_init * 4
Y'  = Y + GOB_SIZE
Y'' = ALIGN(Y', 8)
~~~

In these expressions:

* `sizeof(IOAM_TLV) - 2` excludes the `Option Type` and `Option Length`
  octets already accounted for in the option layout;
* `sizeof(IOAM_PTO_HDR)` is the fixed PTO header size;
* `RemainingLen` retains its exact meaning from {{RFC9197}}: it counts
  only the pre-allocated Node Data Space and does not include the GOB.

The Node Data Space starts immediately after the PTO fixed header and
spans `RemainingLen_init * 4` octets. The GOB occupies the last
`GOB_SIZE` octets of the IOAM option.

Let `PTO_LEN` be the total length of the PTO option content, from the
beginning of the PTO fixed header to the end of the GOB Trailer, and
let `RemainingLen` be the current (received) value of the field. The
following validity check holds for a well-formed option at any point
along the path:

~~~
4 * (GOB_Len + 1) <= PTO_LEN - sizeof(IOAM_PTO_HDR)
                     - RemainingLen * 4
~~~

The check holds with equality at encapsulation time and is preserved
along the path, since RemainingLen can only decrease. A GOB-aware node
MUST treat the GOB as invalid if this check fails.

## IOAM Option Length and Hop-by-Hop Header Length

The IOAM Option Length field MUST be set to `Y''`, that is, to the total
internal length of the IOAM option after rounding up to the required
alignment.

The enclosing Hop-by-Hop Options header length is derived from the total
size of the header, including:

* the 2-octet `Next Header` and `Hdr Ext Len` fields;
* any PadN option used to align the start of the IOAM option;
* the IOAM option itself, including any final tail padding.


Let `HBH_LEN` denote the total Hop-by-Hop Options header size in octets.
Then:

~~~
HBH_LEN = ALIGN(4 + sizeof(IOAM_PTO_HDR) + RemainingLen_init * 4
                + GOB_SIZE, 8)
~~~

where the leading 4 octets account for:

* 2 octets for `Next Header` and `Hdr Ext Len`;
* 2 octets for the PadN option that aligns the IOAM option start.

The IPv6 `Hdr Ext Len` field is encoded as:

~~~
Hdr Ext Len = HBH_LEN / 8 - 1
~~~


## Alignment Semantics

As specified in {{RFC8200}}, the Hop-by-Hop Options header is encoded so
that its total length is a multiple of 8 octets, and the Hdr Ext Len
field carries that length in 8-octet units, not including the first
8 octets.

The final tail padding used to align the enclosing Hop-by-Hop Options
header to an 8-octet boundary is not part of the GOB and is not part of
the Node Data List.

More specifically:

* `Y` identifies the end of the Node Data Space;
* `Y'` identifies the end of the GOB and of the IOAM option;
* `Y''` identifies the aligned end, including padding that lies outside
  the IOAM option.

The difference between `Y'` and `Y''` is due only to final alignment
padding and does not carry GOB or node-data semantics. The GOB is
always located relative to the end of the IOAM option as given by the
IOAM option length, never relative to the end of the Hop-by-Hop Options
header.

## Preservation of PTO Semantics

The presence of the GOB does not alter the semantics of `RemainingLen`
or the pre-allocated nature of the PTO. In particular, `RemainingLen`
counts only the Node Data Space, exactly as in {{RFC9197}}, and the
Node Data Space starts at the same offset as in the base PTO format.

The ingress node determines, at encapsulation time:

* the size of the Node Data Space, via `RemainingLen_init`;
* the GOB size;
* the final aligned IOAM option length.

The following budget constraints apply:

* `RemainingLen_init <= 127`, due to the 7-bit RemainingLen field; note
  that the GOB does not consume this budget;
* the total PTO option content, `sizeof(IOAM_PTO_HDR) +
  RemainingLen_init * 4 + GOB_SIZE`, is bounded by the length limits of
  the enclosing encapsulation (approximately 253 octets for the IPv6
  Hop-by-Hop encapsulation of {{RFC9486}}).

Transit nodes operate only within the pre-allocated bounds of the Node
Data Space and, for GOB-aware nodes, of the GOB Payload, and MUST NOT
change the overall layout of the PTO option.

In this way, the pre-allocated model of the PTO is preserved while
extending the option with a structured global metadata region.


# Backward Compatibility {#backward-compatibility}

This section shows that the GOB is transparent to legacy IOAM transit
nodes, that is, nodes that implement {{RFC9197}} without GOB support.

## Structural Transparency to Legacy Transit Nodes

In the Pre-allocated Trace-Option, the node data list is populated from
the tail of the data space toward its head. Section 4.4.1 of
{{RFC9197}} specifies that RemainingLen "is used to derive the offset
in data space to record the node data element", and that "the recording
of the node data element would start from RemainingLen - NodeLen - size
of (opaque snapshot) in 4-octet units".

A compliant transit node therefore only writes within the interval:

~~~
[X + (RemainingLen - NodeLen - SoS) * 4, X + RemainingLen * 4)
~~~

where `X` is the start of the data space ({{internal-pto-offsets}}) and
`SoS` is the size of the Opaque State Snapshot, if present. Since
RemainingLen starts at `RemainingLen_init` and can only decrease, no
compliant node can ever write at or beyond `X + RemainingLen_init * 4`,
which is exactly where the GOB begins. The GOB is therefore
unreachable by legacy transit nodes, for any number of IOAM hops on
the path.

## Overflow Behavior

When a transit node finds that RemainingLen is smaller than the space
required for its node data element, it sets the Overflow bit (O-bit)
and does not record any data ({{RFC9197}}). With the GOB at the tail of
the option, this behavior is preserved exactly: overflow is signaled
precisely when the Node Data Space is exhausted, and the GOB is not
affected. The ingress node SHOULD set `RemainingLen_init = h * NodeLen`
for the expected number `h` of IOAM-capable hops; if the path contains
more IOAM hops than expected, the additional hops do not record data
and the O-bit is set, while the GOB remains intact.

The use of the variable-length Opaque State Snapshot (bit 22 of the
IOAM Trace-Type) consumes RemainingLen faster than the fixed NodeLen
accounting and thus affects the dimensioning of `RemainingLen_init`,
but does not affect the integrity of the GOB.

## GOB-Only Mode

An ingress node MAY set `RemainingLen_init = 0`, resulting in an option
that carries only the GOB and no node data. In this case, a legacy IOAM
transit node finds no room for its node data element and sets the O-bit
(or skips the option); the GOB is not affected.

## Legacy Decapsulating Nodes and Collectors

A legacy decapsulating node removes the whole IOAM option; the GOB is
lost but packet forwarding is unaffected. A legacy collector that
parses the raw option content and assumes that node data extends to
the end of the option would misinterpret the GOB as the oldest node
data elements. Within a GOB deployment domain, the decapsulating node
and the collector are GOB-aware by definition, so this situation only
arises from misconfiguration. Note that, in that case, the size of the
region interpreted as node data would generally not be an integer
multiple of `NodeLen * 4`, which provides a detection opportunity.

## Path MTU

Section 4.4.1 of {{RFC9197}} relates RemainingLen to the path MTU for
the node data space. An ingress node inserting a GOB MUST also account
for `GOB_SIZE` octets when assessing path MTU constraints.


# Processing Semantics

## Ingress Node Behavior

An ingress node that inserts a GOB into the PTO:

* MUST set the G bit to 1 and the Ver field to 0;
* MUST allocate the full GOB space (Payload and Trailer) at the tail
  of the option at encapsulation time;
* MUST set the GOB Len and Schema fields in the GOB Trailer;
* MUST set `RemainingLen_init` according to the size of the Node Data
  Space only, without including the GOB (e.g., `RemainingLen_init =
  h * NodeLen` for `h` expected IOAM hops);
* MUST determine the final PTO layout, including the Node Data Space,
  the GOB, and any required tail padding;
* MUST account for the GOB size when assessing path MTU constraints.

The ingress node MAY initialize the GOB Payload with structured
metadata, zero values, or other schema-defined initial content.

## Transit Node Behavior

Legacy IOAM transit nodes require no modification: they process the
Node Data Space according to the procedures of {{RFC9197}} and, as
shown in {{backward-compatibility}}, never access the GOB.

A GOB-aware transit node that processes a PTO containing a GOB:

* MUST verify the validity check of {{internal-pto-offsets}} before
  accessing the GOB;
* MAY read the GOB Payload;
* MAY update fields within the GOB Payload, subject to the schema in
  use;
* MUST NOT modify the GOB Trailer (GOB Len and Schema fields);
* MUST NOT modify PTO fixed-header fields, other than Flags and
  RemainingLen as allowed by {{RFC9197}};
* MUST NOT change the overall size or layout of the PTO option.

A transit node MUST confine any GOB update to the pre-allocated GOB
Payload bounds established by the ingress node.

## Egress or Collector Behavior

An egress or collector node that processes a PTO with the G bit set
locates the GOB Trailer as the last 4-octet word of the IOAM option,
and the GOB Payload as the `GOB_PLEN` octets that immediately precede
it. The node data recorded along the path occupies the region between
`X + RemainingLen * 4`, where RemainingLen is the received value, and
the start of the GOB Payload.

Such a node:

* MUST verify the validity check of {{internal-pto-offsets}}; if the
  check fails, the GOB MUST be considered invalid and MUST NOT be
  parsed;
* MAY parse the GOB Payload according to the Schema field;
* MAY export the GOB content as structured metadata or as raw bytes;
* MAY ignore the GOB Payload if the Schema value is unknown or not
  supported.

If the Schema value is unknown, the node MUST NOT infer semantics beyond
the presence and length of the GOB.

# Relation to PTO Semantics

The GOB preserves the pre-allocated semantics of the IOAM PTO.

The ingress node continues to determine the overall PTO allocation at
encapsulation time. This includes the size of the node data list and,
when present, the size of the GOB. Transit nodes operate only within the
pre-allocated space and do not change the overall layout of the PTO.

The Node Data List retains its existing semantics and processing model.
The GOB does not redefine the per-node trace data structure and does not
replace the node data list. Instead, it introduces a distinct global
metadata region placed after the node data space, at the tail of the
option.

In this sense, the pre-allocated nature of the PTO applies both to the
per-node trace space and to the GOB. The GOB extends the PTO with a
global opaque area while preserving the original PTO processing model.


# Schema Semantics and Extensibility {#schema-semantics}

The Schema field identifies the format of the GOB Payload. As for the
Opaque State Snapshot Schema ID of {{RFC9197}}, the Schema field is
interpreted within the context of the IOAM-Namespace indicated by the
Namespace-ID field of the enclosing PTO.

A Schema value may identify:

* a sequence of EIP Information Elements;
* another structured metadata format defined by a specification;
* a locally defined or domain-specific format.

This document defines the GOB container, but does not define the
internal syntax of individual schema-specific payloads. The semantics of
the GOB Payload therefore depend on the specification associated with
the Schema value in use.

Following the model used by {{RFC9197}} for the Namespace-ID, the
24-bit Schema value space is divided into two subranges:

* an operator-assigned range from 0x000000 to 0x7FFFFF, and
* an IANA-assigned range from 0x800000 to 0xFFFFFE.

The operator-assigned range is domain specific and managed by the
network operator: values in this range identify locally defined
schemas, whose meaning is established by configuration within the
deployment domain and within the context of the IOAM-Namespace in use.
No value in this range has a preassigned meaning.

The IANA-assigned range is intended for globally defined schemas
specified in future documents, allowing new and interoperable GOB
payload formats — for example, a registered schema identifying a
sequence of EIP Information Elements.

The Schema value 0xFFFFFF is reserved and indicates that no schema is
associated with the GOB, consistent with the Opaque State Snapshot rule
of {{RFC9197}}, where "the Schema ID MUST be set to 0xFFFFFF to mean no
schema". An encapsulating node that inserts a GOB without an associated
schema MUST set the Schema field to 0xFFFFFF. When the Schema field is
0xFFFFFF, nodes MUST NOT interpret the GOB Payload beyond its presence
and length. Future specifications that define allocation policies for
Schema values MUST preserve this reservation.

# Integration with EIP

One important use of the GOB is the carriage of Extensible In-band
Processing (EIP) Information Elements.

The GOB provides a natural container for EIP within the IOAM PTO because
it offers a single global metadata region with explicit length and
schema identification, distinct from the per-node trace data. This makes
it suitable for metadata that is flow-level, packet-level, or path-level
rather than node-local.

Earlier work explored the integration of EIP into the IOAM framework
through a different approach based on IOAM data fields
[salsano25cscn]. The GOB-based approach defined in this document
provides an alternative integration model based on a pre-allocated
global metadata region within the PTO.

The GOB is a general container for structured metadata. It can carry
EIP Information Elements, but it is not limited to EIP.

The GOB-based integration has also been described and experimentally
evaluated in [mayer26noms].


# Security Considerations

The GOB introduces a global metadata region that may be read or modified
by on-path nodes. Unauthorized access to, or modification of, the GOB
Payload can affect telemetry interpretation, service behavior, or policy
enforcement.

For this reason, deployments of the GOB SHOULD be limited to controlled
or trusted domains, consistent with the limited-domain assumptions often
applied to IOAM deployments.

The GOB Trailer is not protected by any checksum or authentication
mechanism. A misbehaving or malicious on-path node could corrupt the
GOB Trailer; in particular, an inflated GOB Len value could cause a
decapsulating node to interpret recorded node data as GOB Payload. The
validity check of {{internal-pto-offsets}} rejects GOB sizes that are
inconsistent with the option length and with the current RemainingLen
value, and MUST be applied by every GOB-aware node before parsing the
GOB. Deployments MAY additionally cross-check the received GOB Len and
Schema values against per-namespace configuration.

The security implications of the GOB Payload also depend on the Schema
in use. Some schemas may carry purely observational metadata, while
others may influence packet treatment or convey more sensitive
information. Schema-specific specifications SHOULD describe any
additional confidentiality, integrity, or authorization requirements.

# IANA Considerations

This document defines the Global Opaque Block (GOB) as an extension to
the IOAM Pre-allocated Trace Option.

As discussed in {{backward-compatibility}}, the GOB is transparent to
legacy IOAM transit nodes: it is placed outside the region that such
nodes can address, and the semantics of all existing PTO fields are
preserved.

In this light, this document proposes to reuse the existing codepoint
of the IOAM Pre-allocated Trace-Option, defining the G bit and the Ver
field within the Reserved octet of the PTO fixed header, in the
positions specified in {{g-bit-ver}}. Since {{RFC9197}} requires
transit nodes to ignore the Reserved field, this approach does not
affect legacy transit behavior. However, {{RFC9197}} also requires the
encapsulating node to set the Reserved field to zero: the reuse of the
PTO codepoint therefore requires a formal update of the definition of
the Reserved octet of the PTO, allocating its least significant bit as
the G bit and the two preceding bits as the Ver field, while the
remaining five bits stay reserved.

As a subordinate fallback, should the update of the PTO Reserved octet
definition not be acceptable, a new codepoint can be defined for a PTO
variant that supports the Global Opaque Block. In this case, the
presence or absence of the GOB would still be indicated by the G bit,
while the use of a distinct codepoint provides explicit protocol
separation from the current PTO format and avoids any redefinition of
the existing PTO Reserved octet. Note, however, that this fallback
weakens the main benefit of the tail placement: a legacy transit node
that does not recognize the new codepoint would skip the option
entirely, instead of processing the node data space as it does under
the codepoint-reuse approach.

In addition, {{schema-semantics}} divides the 24-bit Schema value space
into an operator-assigned range (0x000000 to 0x7FFFFF) and an
IANA-assigned range (0x800000 to 0xFFFFFE), with 0xFFFFFF reserved to
indicate that no schema is associated with the GOB. A future version of
this document is expected to request the creation of a "GOB Schema
Identifiers" registry covering the IANA-assigned range.

This version of the document does not request IANA actions. Future
versions may request updates to the relevant IOAM or IPv6 registries,
depending on which standardization approach is selected.

--- back

# Acknowledgments
{:numbered="false"}

TBD.
