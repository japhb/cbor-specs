# Captures

This document specifies a tag for representing captures in Concise Binary
Object Representation (CBOR) [1].

    Tag:        25441
    Data Item:  Array containing at most one array followed by at most one map
    Semantics:  Capture [3]
    Contact:    Geoffrey Broadwell <gjb@sonic.net>
    Reference:  https://github.com/japhb/cbor-specs/blob/main/capture.md


## Introduction

In computer languages that can call procedures using both positional and named
arguments (such as Raku and Python), it is useful to be able to represent the
complete set of arguments for a call, separate from performing the call.  This
may be used for diagnostic/debugging purposes, metaprogramming applications, or
to support remote procedure call infrastructure.


## Semantics

An argument capture consists of two components: a positional (array-like) part
and an associative (map-like) part.  Simple positional arguments are collected
into the positional part, and named arguments into the associative part.  Thus
a CBOR Capture tag is applied to an array of at most two elements, with zero or
one array followed by zero or one map, both of which can contain values of any
type.

Preferred serialization omits the array and/or map children if they would be
empty, though the top-level Capture-tagged array cannot be omitted.  Note that
preferred serialization MUST be used by encoders wishing to satisfy the CBOR
core deterministic encoding requirements.

While many languages will use string keys for the named argument map, this is
not required.  For example, some calling conventions follow any positional
arguments with a list of option settings, with each option identified with an
integer or UUID key.  If a protocol wishes to declare that the map has only
string keys, it MAY apply CBOR tag 275 to the map.  Conversely if the protocol
wishes to declare that keys will specifically *not* always be strings, it MAY
instead apply CBOR tag 259 to the map.


## Examples

In the Raku programming language [2], the following declaration specifies a
function with two required positional arguments, one optional positional
argument (which has a default), and one optional named argument acting as a
Boolean option flag:

```
sub vector($x, $y, $z = 0, Bool :$normalize) { ... }
```

All of these are thus valid argument captures for that function:

```
\(1, 3)                  # Only required arguments (defaulting the others)
\(6, 9, -4)              # All positional arguments specified
\(0, 2, :normalize)      # Required positional arguments, normalize flag on
\(1, 2, 3, :!normalize)  # All arguments specified, normalize flag explicitly off
```

These will be encoded as follows; note that the map is skipped for the first
two cases as it would be empty:

```
             # \(1, 3)
d9 63 61     # tag (25441)
   81        # array (1)
      82     # array (2)
         01  # unsigned (1)
         03  # unsigned (3)

             # \(6, 9, -4)
d9 63 61     # tag (25441)
   81        # array (1)
      83     # array (3)
         06  # unsigned (6)
         09  # unsigned (9)
         23  # negative (-4)

             # \(0, 2, :normalize)
d9 63 61     # tag (25441)
   82        # array (2)
      82     # array (2)
         00  # unsigned (0)
         02  # unsigned (2)
      a1     # map (1)
         69 6e 6f 72 6d 61 6c 69 7a 65  # string (9) "normalize"
         f5  # sval (true)

             # \(1, 2, 3, :!normalize)
d9 63 61     # tag (25441)
   82        # array (2)
      83     # array (3)
         01  # unsigned (1)
         02  # unsigned (2)
         03  # unsigned (3)
      a1     # map (1)
         69 6e 6f 72 6d 61 6c 69 7a 65  # string (9) "normalize"
         f4  # sval (false)
```

Of course, it is possible to have a function with only named arguments,
some of which are required:

```
sub release(Str :$name!, Int :$year!, Str :$note) { ... }
```

In this case, the positional argument array would be empty, and thus omitted
under preferred serialization, and the named argument map would contain all of
the serialized information:

```
                                  # \(name => "Diwali", year => 2018)
d9 63 61                          # tag (25441)
   81                             # array (1)
      a2                          # map (2)
         64 6e 61 6d 65           # string (4) "name"
         66 44 69 77 61 6c 69     # string (6) "Diwali"
         64 79 65 61 72           # string (4) "year"
         19 07 e2                 # unsigned (2018)
```

If desired, the same map could be explicitly tagged to only have string keys:

```
                                  # \(name => "Diwali", year => 2018)
d9 63 61                          # tag (25441)
   81                             # array (1)
      d9 01 13                    # tag (275)
         a2                       # map (2)
            64 6e 61 6d 65        # string (4) "name"
            66 44 69 77 61 6c 69  # string (6) "Diwali"
            64 79 65 61 72        # string (4) "year"
            19 07 e2              # unsigned (2018)
```

Finally, a completely empty capture has the following four-byte representation:


```
                                  # \()
d9 63 61                          # tag (25441)
   80                             # array (0)
```


## Rationale

As with CBOR tag 40 (Multi-dimensional Array) [4], this tag binds together two
lower-level data structures to create a semantically more complex structure,
and thus uses the same basic layout: a tagged array containing two
substructures, in this case an array and a map.  Unlike tag 40, because the two
substructures for this tag are different major types, it is possible to
unambiguously omit one or both to gain coding efficiency when they would
otherwise be empty.

The tag number (25441) was chosen as ASCII "ca", mnemonic for "capture".


## Implementations

* [Raku] [CBOR::Simple](https://github.com/japhb/CBOR-Simple) (reference implementation)


## References

[1] Bormann, C. and P. Hoffman, "Concise Binary Object Representation (CBOR)",
    STD 94, RFC 8949, December 2020, <https://www.rfc-editor.org/info/rfc8949>

[2] The Raku Programming Language, <https://raku.org/>

[3] Raku Capture type documentation, <https://docs.raku.org/type/Capture>

[4] Bormann, C., "Concise Binary Object Representation (CBOR) Tags for Typed Arrays",
    RFC 8746, February 2020,
    <https://www.rfc-editor.org/rfc/rfc8746.html#name-multi-dimensional-array>


## Author

Geoffrey Broadwell <gjb@sonic.net>
