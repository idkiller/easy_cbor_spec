# CBOR specification

The Concise Binary Object Representation (CBOR) is a data format
whose design goals include the possibility of extremely small code
size, fairly small message size, and extensibility without the need
for version negotiation.  These design goals make it different from
earlier binary serializations such as ASN.1 and MessagePack.


## Overview

type name                                     | initial byte (bin)    | initial byte (hex)
----------------------------------------------|-----------------------|-------------------
fix positive integer (0-23)                   | 000_00000 - 000_10111 | 0x00 - 0x17
positive 8-bit integer                        | 000_11000             | 0x18
positive 16-bit integer                       | 000_11001             | 0x19
positive 32-bit integer                       | 000_11010             | 0x1a
positive 64-bit integer                       | 000_11011             | 0x1b
fix negative integer (-1 - -24)               | 001_00000 - 001_10111 | 0x20 - 0x37
negative 8-bit integer                        | 001_11000             | 0x38
negative 16-bit integer                       | 001_11001             | 0x39
negative 32-bit integer                       | 001_11010             | 0x3a
negative 64-bit integer                       | 001_11011             | 0x3b
fix length byte string (0-23)                 | 010_00000 - 010_10111 | 0x40 - 0x57
8-bit length byte string                      | 010_11000             | 0x58
16-bit length byte string                     | 010_11001             | 0x59
32-bit length byte string                     | 010_11010             | 0x5a
64-bit length byte string                     | 010_11011             | 0x5b
indefinite length byte string                 | 010_11111             | 0x5f
fix length unicode string (0-23)              | 011_00000 - 011_10111 | 0x60 - 0x77
8-bit length unicode string                   | 011_11000             | 0x78
16-bit length unicode string                  | 011_11001             | 0x79
32-bit length unicode string                  | 011_11010             | 0x7a
64-bit length unicode string                  | 011_11011             | 0x7b
indefinite length unicode string              | 011_11111             | 0x7f
fix length array (0-23)                       | 100_00000 - 100_10111 | 0x80 - 0x97
8-bit length array                            | 100_11000             | 0x98
16-bit length array                           | 100_11001             | 0x99
32-bit length array                           | 100_11010             | 0x9a
64-bit length array                           | 100_11011             | 0x9b
indefinite length array                       | 100_11111             | 0x9f
fix length map (0-23)                         | 101_00000 - 101_10111 | 0xa0 - 0xb7
8-bit length map                              | 101_11000             | 0xb8
16-bit length map                             | 101_11001             | 0xb9
32-bit length map                             | 101_11010             | 0xba
64-bit length map                             | 101_11011             | 0xbb
indefinite length array                       | 101_11111             | 0xbf
tag type (n length tags)                      | 110_00000 - 110_11011 | 0xc0 - 0xdb
simple value : unassigned                     | 111_00000 - 111_10011 | 0xe0 - 0xf3
simple value : False                          | 111_10100             | 0xf4
simple value : True                           | 111_10101             | 0xf5
simple value : Null                           | 111_10110             | 0xf6
simple value : undefined value                | 111_10111             | 0xf7
simple value : reserved for future            | 111_11000             | 0xf8
IEEE 754 half-precision float (16 bits)       | 111_11001             | 0xf9
IEEE 754 single-precision float (32 bits)     | 111_11010             | 0xfa
IEEE 754 double-precision float (64 bits)     | 111_11011             | 0xfb
unassigned                                    | 111_11100 - 111_11110 | 0xfc - 0xfe
"break" stop code for indefinite-length items | 111_11111             | 0xff

Four CBOR items (array, maps, byte strings, text strings) can be encoded with an indefinite length using additional information value 31.

### tag type

Initial byte of the tag follow the rules for positive integer. (major 0)

 tag                         | data item    |semantics
-----------------------------|--------------|--------------------------
 0                           | utf-8 string |standard date/time string
 1                           | multiple     |epoch-based date/time
 2                           | byte string  |positive big number
 3                           | byte string  |negative big number
 4                           | array        |decimal fraction
 5                           | array        |big float
 6 - 20                      | unassigned   |unassigned
 21                          | multiple     |base64url encoding
 22                          | multiple     |base64 encoding
 23                          | multiple     |base16 encoding
 24                          | byte string  |Encoded CBOR data item
 25 - 31                     | unassigned   |unassigned
 32                          | utf-8 string |URI
 33                          | utf-8 string |base64url
 34                          | utf-8 string |base64
 35                          | utf-8 string |regular expression
 36                          | utf-8 string |MIME message
 37 - 55798                  | unassigned   |unassigned
 55799                       | multiple     |Self-describe CBOR
 55800 - 18446744073709551615| unassigned   |unassigned



## Notation in diagrams

    one byte:
    +--------+
    |        |
    +--------+

    a variable number of bytes:
    +========+
    |        |
    +========+

    variable number of objects stored in Cbor format:
    +~~~~~~~~~~~~~~~~~+
    |                 |
    +~~~~~~~~~~~~~~~~~+

`X`, `Y`, `Z` and `A` are the symbols that will be replaced by an actual bit.

## Initial byte

    +----------+
    | xxxyyyyy |
    +----------+

    * xxx   = 3-bit major type
    * yyyyy = 5-bit additional information
    
    Additional information
    0  ~ 23 : unsigned integer
         24 : 1 byte unsigned integer following byte
         25 : 2 byte unsigned integer following byte
         26 : 3 byte unsigned integer following byte
         27 : 4 byte unsigned integer following byte
    28 ~ 30 : reserved for future
         31 : indefinite-length 

### Major Type
