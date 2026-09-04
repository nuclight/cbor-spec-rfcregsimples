---
title: "A CBOR Simple Values Range for Packing and Templating"
abbrev: "CBOR Simple Values Range"
category: std
updates: 8949

docname: draft-goncharov-rfcregsimples-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "Concise Binary Object Representation Maintenance and Extensions"
keyword:
 - CBOR
 - simple values
 - compression
 - templating
 - packed CBOR
venue:
  group: "cbor"
  type: "Working Group"
  mail: "cbor@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/cbor/"
  github: "nuclight/cbor-spec-rfcregsimples"
  latest: "https://nuclight.github.io/cbor-spec-rfcregsimples/draft-goncharov-rfcregsimples.html"

author:
 -
    fullname: "Vadim Goncharov"
    organization: Consultant
    email: "vadimnuclight@gmail.com"

normative:
  RFC8949:
  IANA.cbor-simple-values:
    title: "Concise Binary Object Representation (CBOR) Simple Values"
    target: "https://www.iana.org/assignments/cbor-simple-values"
    author:
      - org: "IANA"
    date: false

informative:
  RFC1951:
  RFC4251:
  RFC4419:
  RFC4432:
  RFC5656:
  I-D.ietf-cbor-packed:
  MyLED:
    title: "MyLED Thing Description (WoT plugfest test data)"
    target: "https://github.com/w3c/wot-thing-description/raw/db8abb3655afc7f149db7976ba4e79149619f537/test-bed/data/plugfest/2017-05-osaka/MyLED_f.jsonld"
    author:
      - org: "W3C Web of Things Interest Group"
    date: 2017
  CBAR:
    title: "CBOR & generic BLOB by-Atoms Reducing (CBAR), Work in Progress"
    target: "https://github.com/nuclight/musctp/blob/main/cbar.txt"
    author:
      - name: "Vadim Goncharov"
        ins: "V. Goncharov"
    date: false

...

--- abstract

The Concise Binary Object Representation (CBOR, RFC 8949, STD 94) is a
data format whose design goals include the possibility of extremely
small code size, fairly small message size, and extensibility without
the need for version negotiation.

This document registers a range of sixteen CBOR simple values (0 to
15) that can be shared by different specifications using them for CBOR
transformations, such as compression or templating, in a
non-conflicting way.
This allows current and future specifications to
reuse the smallest (single-byte) simple values range while defining
their own ways to use them for achieving their goals.

This document updates RFC 8949.

--- middle

# Introduction

CBOR does not provide any forms of data compression.
While traditional data compression techniques such as DEFLATE
{{RFC1951}} can work well for CBOR encoded data items, their
disadvantage is that the recipient needs to decompress the compressed
form to make use of the data.

This document describes a set of CBOR simple values which can be used
by different specifications utilizing them for CBOR transformations,
such as compression or templating, in a non-conflicting way.
This allows for future standards to reuse the smallest values range
while inventing better ways to use them for achieving their goals.

# Terminology and Conventions

{::boilerplate bcp14-tagged}

<!-- TODO: any other terminology needed here? -->

# Motivation

In a world of constrained devices (IoT) such devices often have very
little memory, while these devices are also constrained in
e.g. packet size so that compression is very desirable.
Traditional generic compression algorithms like DEFLATE {{RFC1951}}
perform well but require memory both for decompressed data and
internal state of the decompressor itself.
Many constrained implementations would want to trade off between
compression ratio and required memory - tolerating worse compression
by lowering memory usage, ideally to zero bytes (that is, using
compressed data in-place as-is).
In the ongoing IETF efforts the "MyLED" example in JSON {{MyLED}}
takes 1210 bytes converted to CBOR which may be further reduced to
about 350 bytes using DEFLATE, but this requires more than 1.5 Kbytes
of memory during decompression - in contrast, work in progress
specifications like {{I-D.ietf-cbor-packed}} and {{CBAR}} are able to
reduce this example to about 500 bytes, however accessible in-place,
without need for a decompression buffer at all.

Sixteen simple values were intended to be used for compression
purposes in the original CBOR proposal, but didn't get into the
standard at the time being due to the complexity of the task.
Since then, more than one proposal for compression techniques has
appeared.
While the usual way for Internet specifications is to acquire
different codepoints (or ranges of them) from IANA, this does not work
well in the compression world given the specifics of CBOR encoding -
that is, different numbers are encoded in CBOR occupying a different
number of bytes.
Compression efficiency, however, is very sensitive to the length of
codes used - the shorter, the better.
Thus, registration of a short codes range for one standard exclusively
would put other standards - especially future standards - into an
unequal and unfair position if there are no more short codes left for
them.
However, it is known that future inventions tend to be more effective
in usage of the same resources but are impossible to predict at the
time of a prior standard being issued (otherwise enhancements would be
already incorporated into it).
As (at the time of writing) there are only 20 single-byte CBOR simple
values available for registration, such a situation can happen with
just the first standard accepted.

Therefore, this document registers the first sixteen CBOR simple
values (0 to 15) in such a way that they could be used by different
compression methods (different specifications), both existing and not
yet invented.
The way for a decoder to distinguish them - that is, to interpret
according to one specification or another - is done via a CBOR Tag
(acting as a "namespace") or by external means such as messages' media
type.

This range registration is similar in spirit to {{RFC4251}} allocating
a range of message numbers e.g. 30 to 49 to be key exchange method
specific, stating that numbers can be reused for different
authentication methods - and examples of specifications using such a
range are {{RFC4419}}, {{RFC4432}} or {{RFC5656}}.

# Specification

This document registers CBOR values simple(0) to simple(15) for needs
like packing or templating CBOR documents.

Following the spirit of {{RFC4251}}, this specification could just
declare this range to be packing/templating method specific (so the
text could end right here), but this would not be very useful to CBOR
parser implementers in terms of what could be expected from these
values or what kind of changes specific packing/templating methods may
require in their parsers.
So in the text below we take a slightly different approach.

This specification tries, speaking in object-oriented terms, to be the
"Abstract Base Class" for other specifications to "inherit", in the
sense of describing meaning as broadly as possible, but concrete
specifications do not need to implement every imaginable feature and
MAY choose to define only a subset.

Application protocols using CBOR simple values in the 0..15 range, or
generic packing/templating specifications, should include a reference
to this document about using the range and then describe the precise
usage of these simple values.

The semantics of these simple values 0 to 15 is that they are
substitutions for one or several other CBOR items.
That is, when a decoder sees values simple(0) to simple(15), it treats
them as some mapping function (0 to 15 may be viewed as the argument
here) which takes an argument and, depending on the current decoder
state, returns CBOR item(s) to substitute instead of the simple value,
as if the simple value never occurred in the stream, but the returned
CBOR item(s) occurred instead.

Such a mapping function may be very complex, taking into account even
surrounding CBOR items context to decide.
Or (especially expected for short-term solutions) an "inheriting"
specification may choose it to be rather simple, for example,
simple(0)..simple(15) could be references (indexes) to some table
(kept by the decoding process), or part of such a table, so that
instead of a simple value an integral number of CBOR items is
substituted from a table entry.
Note that these replacement items do not necessarily form a complete
well-formed CBOR item if viewed separately as an entry in the table.
For example, a sequence

~~~
[..., 1, 2, simple(2), "a", "b", "c", ...]
~~~

could expand to

~~~
[..., 1, 2, ["foo", "bar", "a"], "b", "c", ...]
~~~

if the entry for simple(2) at this moment contained a definite-length
array start item (0x83) and two string items "foo" and "bar".

(Informally speaking, this could be described as "cut", in a text
editor, a whole number of tokens (counting each string, brace, etc. as
one token) in CBOR diagnostic notation and then "paste" it instead of
each simple(N) occurrence).

Please note that "surrounding CBOR items context" (if the
specification goes the complex way) may include e.g. the absolute or
relative position of the simple value.
In the (imaginary non-normative) example:

~~~
10([simple(0), 16, "atom1",
   {
     "foo": simple(0),
     "bar": 10([simple(0), 16, "otheratom",
               {
                  "baz": simple(0),
                  "quux": 10(122(0))
               }
            ])
   }
])
~~~

the simple(0) at the very start of the array expands to
"function-name", but the same simple(0) as the map value expands to
(presumably) "atom1" under the "foo" key and to "otheratom" under the
"baz" key.
That is, if a packing method specification chooses the "index in
table" approach, there could be several such tables.

Even more, "surrounding context" usage is not prohibited from explicit
arguments and their "swallowing" after substitution, for example, in

~~~
[..., 1, 2, "arg1", "arg2", simple(2), 3, ...]
~~~

or

~~~
[..., 1, 2, simple(2), "arg1", "arg2", 3, ...]
~~~

simple(2) could be viewed as a function call telling to use 2
arguments from surrounding CBOR items, and then do something based on
"arg1" and "arg2", so probably the whole triple will be replaced by
the function return value, not just simple(N) itself:

~~~
[..., 1, 2, {"substituted": "sequence"}, ['frobnicate'], 3, ...]
~~~

However, such advanced usage is NOT RECOMMENDED for generic
specifications because it is hard to do it right with generic CBOR
parsers if such parsers are streaming / event-based.
(This may be not true for application-specific parsers in
application-specific packing/templating methods.)

Therefore, as usage of simple(0)..simple(15) can modify the structure
of a CBOR document, they SHOULD be treated as an error if used outside
of an area where a table (or part of such a table) is set up, e.g.
outside of a "namespace" tag or an area implied by a media type.
The exact mechanism to set up such tables (or modify them so that the
same simple value may expand to different content in different parts
of a CBOR stream) is left for definition by the application protocol,
or an application protocol may "inherit" it (implemented in a library)
from a generic specification in an appropriate IETF RFC document (such
as {{I-D.ietf-cbor-packed}}).

## Interoperability

Note that it is straightforward to have usage of simple values of
different compression/templating specifications in the non-overlapping
areas of a single CBOR document, because the primary method for
specifications to manifest themselves is via using some CBOR Tag(s) as
a "namespace" ("area"), like in this example:

~~~
{
  "foo": 55510([..., simple(1), /meaning by spec 1/ ...]),
  "bar": 55513([..., simple(1), /meaning by spec 2/ ...])
}
~~~

(the tag numbers here are fictitious, to be used just as an example).

Tag nesting, however, which could be used for things like providing
the output of one decompression phase (by one specification) as input
for another (by the same or a different specification) is a more
complicated case, details of which are left out from this document to
be defined by actual specifications.
It is noted here for specifications' authors that the possibility of
such multi-phase (nested) processing is the reason for a "SHOULD"
requirement above instead of a "MUST", because simple values (of the
0..15 range) produced as a result of one phase may be fed to a
different decoder.

Here, only some basic principle for tag nesting is defined: a packing
method tag MUST NOT be used as the outer one if it does not support
preserving non-own simple values after applying its transformations
(unpacking).
This is best illustrated by an example: suppose that for the imaginary
tags above, the specification of tag 55510 supports such preserving
(in the example, it will list integers corresponding to simple numbers
and values corresponding to them) by leaving them as-is, and the
specification of tag 55513 uses a simplistic implementation where
simple(N) is just an index N into a table.
Then, the following example is valid:

~~~
55510([10, "foo", 11, "bar", 12, "baz", /* outer tag's setup */
   /* outer tag's transformed data area */
   55513([                                    /* inner tag's */
      ["rgbValue", "rgbValueRed", "rgbValueGreen"], /* setup */
      [/* inner tag's transformed data area */
         simple(0), simple(1), simple(2), simple(10),
         simple(0), simple(1), simple(2), simple(11),
         simple(0), simple(1), simple(2), simple(12)
      ]
   ])
])
~~~

As tags are processed from outside to inside, the decoder of tag 55510
will make this fragment equivalent to:

~~~
55513([                                    /* inner tag's */
   ["rgbValue", "rgbValueRed", "rgbValueGreen"], /* setup */
   [/* inner tag's transformed data area */
      simple(0), simple(1), simple(2), "foo",
      simple(0), simple(1), simple(2), "bar",
      simple(0), simple(1), simple(2), "baz"
   ]
])
~~~

which the decoder of tag 55513 finally turns into unpacked form:

~~~
[
   "rgbValue", "rgbValueRed", "rgbValueGreen", "foo",
   "rgbValue", "rgbValueRed", "rgbValueGreen", "bar",
   "rgbValue", "rgbValueRed", "rgbValueGreen", "baz"
]
~~~

However, the opposite is not possible because the specification of tag
55513 does not make exceptions.
The following example is NOT valid:

~~~
55513([                                    /* outer tag's */
   ["rgbValue", "rgbValueRed", "rgbValueGreen"], /* setup */
   /* outer tag's transformed data area */
   55510([10, "foo", 11, "bar", 12, "baz", /* inner setup */
      [/* inner tag's transformed data area */
         simple(0), simple(1), simple(2), simple(10),
         simple(0), simple(1), simple(2), simple(11),
         simple(0), simple(1), simple(2), simple(12)
      ]
   ])
])
~~~

<!-- TODO 05.12.25: outside of namespace tag - for CBOR Sequences -->

## Validity Checking

Please note that the whole concept of CBOR document transformation, be
it packing or templating, means that, in the general case, it is not
safe to do validity checks even on known tags as described by Section
5.3.2 of {{RFC8949}}.
For example, Tag 32 means URI and requires a text string as its
content.
However, the goal of packing is to reduce redundant information, which
may mean replacing such a text string with a simple value, raising the
possibility of the following example:

~~~
55513([..., 32(simple(1)), ...])
~~~

which would be considered as an error by a validity-checking decoder.

Note that Section 5.4 of {{RFC8949}} does not cover this case, because
it is focused on evolution; however, with packing mechanisms this may
occur with not new but already known, stable tags.

Therefore, this document updates Section 5.4 of {{RFC8949}} dictating
that generic decoders performing validity checking MUST provide a way
to disable this validity checking.
Note that a mode, where the decoder knows the packing method and
postpones validity checking until after unpacking, is possible - but
is not sufficient, because newer specifications, yet unknown to such a
decoder, may appear in the future after its deployment.

# IANA Considerations

In the "CBOR Simple Values" registry {{IANA.cbor-simple-values}}, IANA
is requested to allocate the simple values defined in {{tab-simple-values}}.

| Value | Semantics                                                 | Reference  |
|-------|-----------------------------------------------------------|------------|
| 0..15 | Packing and templating: shared by multiple specifications | RFC XXXX   |
{: #tab-simple-values title="Simple Values"}

\[RFC EDITOR: Please replace "RFC XXXX" above with the RFC number
assigned to this document.\]

# Security Considerations

The security considerations of {{RFC8949}} apply.

If a specification chooses to implement generic non-well-formed
entries in a table (modifying CBOR structure as in the example above),
care must be taken to avoid implementation bugs potentially leading to
out-of-bounds accesses.

Specifications must decide what to do with the possibility of decoding
loops or infinite recursion, for example, if the table entry for
simple(M) includes another simple(N) value which then may be expanded
recursively.
Possible ways are either to not expand it at all, allowing for the
aforementioned multi-phase processing, or to provide counter-measures
similar to symlink processing in filesystems, or something else.

--- back

# Acknowledgments
{:numbered="false"}

TODO: acknowledge.

<!--
TBD for 04.12.25 meeting:
* 31(undefined)
* move tags from 240-255 to release space for abbreviations
* 'As the unpacking is deterministic ... CDE' is not fully true for all goals
* record tag - yet another form of tables?
* tags in simples?
* dns-cbor: subj "slides on DNS/packed for 2025-05-14 interim" still unresolved
-->
