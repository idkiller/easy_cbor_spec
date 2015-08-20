# CBOR specification

The Concise Binary Object Representation (CBOR) is a data format
whose design goals include the possibility of extremely small code
size, fairly small message size, and extensibility without the need
for version negotiation.  These design goals make it different from
earlier binary serializations such as ASN.1 and MessagePack.



## Overview

type name                                     | initial byte (bin)    | initial byte (hex)
----------------------------------------------|-----------------------|-------------------
fix positive integer (0 ~ 23)                 | 000_00000 ~ 000_10111 | 0x00 ~ 0x17
positive 8-bit integer                        | 000_11000             | 0x18
positive 16-bit integer                       | 000_11001             | 0x19
positive 32-bit integer                       | 000_11010             | 0x1a
positive 64-bit integer                       | 000_11011             | 0x1b
fix negative integer (~24 ~ ~1)               | 001_00000 ~ 001_10111 | 0x20 ~ 0x37
negative 8-bit integer                        | 001_11000             | 0x38
negative 16-bit integer                       | 001_11001             | 0x39
negative 32-bit integer                       | 001_11010             | 0x3a
negative 64-bit integer                       | 001_11011             | 0x3b
fix length byte string (0 ~ 23)               | 010_00000 ~ 010_10111 | 0x40 ~ 0x57
8-bit length byte string                      | 010_11000             | 0x58
16-bit length byte string                     | 010_11001             | 0x59
32-bit length byte string                     | 010_11010             | 0x5a
64-bit length byte string                     | 010_11011             | 0x5b
indefinite length byte string                 | 010_11111             | 0x5f
fix length unicode string (0 ~ 23)            | 011_00000 ~ 011_10111 | 0x60 ~ 0x77
8-bit length unicode string                   | 011_11000             | 0x78
16-bit length unicode string                  | 011_11001             | 0x79
32-bit length unicode string                  | 011_11010             | 0x7a
64-bit length unicode string                  | 011_11011             | 0x7b
indefinite length unicode string              | 011_11111             | 0x7f
fix length array (0 ~ 23)                     | 100_00000 ~ 100_10111 | 0x80 ~ 0x97
8-bit length array                            | 100_11000             | 0x98
16-bit length array                           | 100_11001             | 0x99
32-bit length array                           | 100_11010             | 0x9a
64-bit length array                           | 100_11011             | 0x9b
indefinite length array                       | 100_11111             | 0x9f
fix length map (0 ~ 23)                       | 101_00000 ~ 101_10111 | 0xa0 ~ 0xb7
8-bit length map                              | 101_11000             | 0xb8
16-bit length map                             | 101_11001             | 0xb9
32-bit length map                             | 101_11010             | 0xba
64-bit length map                             | 101_11011             | 0xbb
indefinite length array                       | 101_11111             | 0xbf
tag type (n length tags)                      | 110_00000 ~ 110_11011 | 0xc0 ~ 0xdb
simple value : unassigned                     | 111_00000 ~ 111_10011 | 0xe0 ~ 0xf3
simple value : False                          | 111_10100             | 0xf4
simple value : True                           | 111_10101             | 0xf5
simple value : Null                           | 111_10110             | 0xf6
simple value : undefined value                | 111_10111             | 0xf7
simple value : reserved for future            | 111_11000             | 0xf8
IEEE 754 half-precision float (16 bits)       | 111_11001             | 0xf9
IEEE 754 single-precision float (32 bits)     | 111_11010             | 0xfa
IEEE 754 double-precision float (64 bits)     | 111_11011             | 0xfb
unassigned                                    | 111_11100 ~ 111_11110 | 0xfc ~ 0xfe
"break" stop code for indefinite-length items | 111_11111             | 0xff



### Tag type

Initial byte of the tag follow the rules for positive integer. (major 0)

 tag                         | data item    |semantics
-----------------------------|--------------|--------------------------
 0                           | utf-8 string |standard date/time string
 1                           | multiple     |epoch-based date/time
 2                           | byte string  |positive big number
 3                           | byte string  |negative big number
 4                           | array        |decimal fraction
 5                           | array        |big float
 6 ~ 20                      | unassigned   |unassigned
 21                          | multiple     |base64url encoding
 22                          | multiple     |base64 encoding
 23                          | multiple     |base16 encoding
 24                          | byte string  |Encoded CBOR data item
 25 ~ 31                     | unassigned   |unassigned
 32                          | utf-8 string |URI
 33                          | utf-8 string |base64url
 34                          | utf-8 string |base64
 35                          | utf-8 string |regular expression
 36                          | utf-8 string |MIME message
 37 ~ 55798                  | unassigned   |unassigned
 55799                       | multiple     |Self-describe CBOR
 55800 ~ 18446744073709551615| unassigned   |unassigned



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

All multi-byte values are encoded in network byte order
(that is, most significant byte first, also known as "big-endian").



## Type system

The initial byte of each data item contains both information about the major
type (the high-order 3 bits) and additional information (the low-order 5 bits).



### Initial byte

    +----------+
    | XXXYYYYY |
    +----------+

    * XXX   = 3-bit major type
    * YYYYY = 5-bit additional information



#### Major Type

    0 : positive integer.
    1 : negative integer.
    2 : a byte string.
    3 : a unicode string.
    4 : an array of data items.
    5 : a map of pairs of data items.
    6 : optional semantic tagging of other major types.
    7 : floating point numbers and simple data types that need no content 
        and "break" stop code.
    


#### Additional information

    0  ~ 23 : unsigned integer
         24 : 1 byte unsigned integer following byte
         25 : 2 byte unsigned integer following byte
         26 : 3 byte unsigned integer following byte
         27 : 4 byte unsigned integer following byte
    28 ~ 30 : reserved for future
         31 : indefinite-length 



#### Indefinite Lengths for Some Major Types

Four CBOR items (array, maps, byte strings, text strings) can be encoded 
with an indefinite length using additional information value 31.



## integer format family

    positive fix integer from 0 to 23 ( XXXXX <= 10111)
    +--------+
    |000XXXXX|
    +--------+

    negative fix integer from -24 to -1 ( XXXXX <= 10111)
    +--------+
    |001XXXXX|
    +--------+

    positive 8-bit unsigned integer
    +------+--------+
    | 0x18 |ZZZZZZZZ|
    +------+--------+

    positive 16-bit unsigned integer
    +------+--------+--------+
    | 0x19 |ZZZZZZZZ|ZZZZZZZZ|
    +------+--------+--------+

    positive 32-bit unsigned integer
    +------+--------+--------+--------+--------+
    | 0x1a |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|
    +------+--------+--------+--------+--------+

    positive 64-bit unsigned integer
    +------+--------+--------+--------+--------+--------+--------+--------+--------+
    | 0x1a |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|
    +------+--------+--------+--------+--------+--------+--------+--------+--------+

    negative 8-bit integer
    +------+--------+
    | 0x38 |ZZZZZZZZ|
    +------+--------+

    negative 16-bit integer
    +------+--------+--------+
    | 0x39 |ZZZZZZZZ|ZZZZZZZZ|
    +------+--------+--------+

    negative 32-bit integer
    +------+--------+--------+--------+--------+
    | 0x3a |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|
    +------+--------+--------+--------+--------+

    negative 64-bit integer
    +------+--------+--------+--------+--------+--------+--------+--------+--------+
    | 0x3b |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|
    +------+--------+--------+--------+--------+--------+--------+--------+--------+



## float format family

    IEEE 754 half-precision float
    +------+--------+--------+
    | 0xf9 |ZZZZZZZZ|ZZZZZZZZ|
    +------+--------+--------+

    IEEE 754 single-precision float
    +------+--------+--------+--------+--------+
    | 0xfa |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|
    +------+--------+--------+--------+--------+

    IEEE 754 double-precision float
    +------+--------+--------+--------+--------+--------+--------+--------+--------+
    | 0xfb |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|
    +------+--------+--------+--------+--------+--------+--------+--------+--------+



 ## byte string family

    a byte array whose length is 0 through 23 bytes ( XXXXX <= 10111)
    +--------+======+
    |010XXXXX| data |
    +--------+======+

    a byte array whose length is 0 through 255 bytes
    +------+--------+======+
    | 0x58 |ZZZZZZZZ| data |
    +------+--------+======+

    a byte array whose length is 0 through 65535 bytes
    +------+--------+--------+======+
    | 0x59 |ZZZZZZZZ|ZZZZZZZZ| data |
    +------+--------+--------+======+

    a byte array whose length is 0 through 4294967295 bytes
    +------+--------+--------+--------+--------+======+
    | 0x5a |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ| data |
    +------+--------+--------+--------+--------+======+

    a byte array whose length is 0 through 18446744073709551615 bytes
    +------+--------+--------+--------+--------+--------+--------+--------+--------+======+
    | 0x5a |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ| data |
    +------+--------+--------+--------+--------+--------+--------+--------+--------+======+



## text string family

    a unicde array whose length is 0 through 23 bytes ( XXXXX <= 10111)
    +--------+======+
    |011XXXXX| data |
    +--------+======+

    a unicode array whose length is 0 through 255 bytes
    +------+--------+======+
    | 0x78 |ZZZZZZZZ| data |
    +------+--------+======+

    a unicode array whose length is 0 through 65535 bytes
    +------+--------+--------+======+
    | 0x79 |ZZZZZZZZ|ZZZZZZZZ| data |
    +------+--------+--------+======+

    a unicode array whose length is 0 through 4294967295 bytes
    +------+--------+--------+--------+--------+======+
    | 0x7a |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ| data |
    +------+--------+--------+--------+--------+======+

    a unicode array whose length is 0 through 18446744073709551615 bytes
    +------+--------+--------+--------+--------+--------+--------+--------+--------+======+
    | 0x7b |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ| data |
    +------+--------+--------+--------+--------+--------+--------+--------+--------+======+



## array format family

    an array whose length is 0 through 23 elements ( XXXXX <= 10111 )
    +--------+~~~~~~~~~~~+
    |100XXXXX| N objects |
    +--------+~~~~~~~~~~~+

    an array whose length is 0 through 255 elements
    +------+--------+~~~~~~~~~~~+
    | 0x98 |ZZZZZZZZ| N objects |
    +------+--------+~~~~~~~~~~~+

    an array whose length is 0 through 65535 elements
    +------+--------+--------+~~~~~~~~~~~+
    | 0x99 |ZZZZZZZZ|ZZZZZZZZ| N objects |
    +------+--------+--------+~~~~~~~~~~~+

    an array whose length is 0 through 4294967295 elements
    +------+--------+--------+--------+--------+~~~~~~~~~~~+
    | 0x9a |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ| N objects |
    +------+--------+--------+--------+--------+~~~~~~~~~~~+

    an array whose length is 0 through 18446744073709551615 elements
    +------+--------+--------+--------+--------+--------+--------+--------+--------+~~~~~~~~~~~+
    | 0x9b |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ| N objects |
    +------+--------+--------+--------+--------+--------+--------+--------+--------+~~~~~~~~~~~+

    an array whose length is indefinite-length
    +------+~~~~~~~~~~~+------+
    | 0x9f | N objects | 0xff |
    +------+~~~~~~~~~~~+------+

    * N is the size of an array

    represent the abstract definite-length array of multi depth
    [1, [2, 3], [4, 5]]
    +------+------+------+------+------+------+------+------+
    | 0x83 | 0x01 | 0x82 | 0x02 | 0x03 | 0x82 | 0x04 | 0x05 |
    +------+------+------+------+------+------+------+------+
    explain each byte
    +------+
    | 0x83 | -- array of length 3
    +------+------+
           | 0x01 | -- 1
           +------+------+------+
           | 0x82 | 0x02 | 0x03 | -- array of length 2 [2, 3]
           +------+------+------+
           | 0x82 | 0x04 | 0x05 | -- array of length 2 [4, 5]
           +------+------+------+

    represent the indefinite-length array of multi depth
    [1, [2, 3], [4, 5]]
    +------+------+------+------+------+------+------+------+------+------+
    | 0x9f | 0x01 | 0x82 | 0x02 | 0x03 | 0x9f | 0x04 | 0x05 | 0xff | 0xff |
    +------+------+------+------+------+------+------+------+------+------+
    explain each byte
    +------+
    | 0x9f | -- start indefinite-length array
    +------+------+
           | 0x01 | -- 1
           +------+------+------+
           | 0x82 | 0x02 | 0x03 | -- array of length 2 [2, 3]
           +------+------+------+
           | 0x9f | -- start indefinite-length array
           +------+-------+------+
                  |  0x04 | 0x05 | -- 4, 5
           +------+-------+------+
           | 0xff | -- break (inner array)
    +------+------+
    | 0xff | -- break (outer array)
    +------+



## map format family
    
    a map whose length is 0 through 23 elements ( XXXXX <= 10111 )
    +--------+~~~~~~~~~~~~~+
    |101XXXXX| N*2 objects |
    +--------+~~~~~~~~~~~~~+

    a map whose length is 0 through 255 elements
    +------+--------+~~~~~~~~~~~~~+
    | 0xb8 |ZZZZZZZZ| N*2 objects |
    +------+--------+~~~~~~~~~~~~~+

    a map whose length is 0 through 65535 elements
    +------+--------+--------+~~~~~~~~~~~~~+
    | 0xb9 |ZZZZZZZZ|ZZZZZZZZ| N*2 objects |
    +------+--------+--------+~~~~~~~~~~~~~+

    a map whose length is 0 through 4294967295 elements
    +------+--------+--------+--------+--------+~~~~~~~~~~~~~+
    | 0xba |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ| N*2 objects |
    +------+--------+--------+--------+--------+~~~~~~~~~~~~~+

    a map whose length is 0 through 18446744073709551615 elements
    +------+--------+--------+--------+--------+--------+--------+--------+--------+~~~~~~~~~~~~~+
    | 0xbb |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ| N*2 objects |
    +------+--------+--------+--------+--------+--------+--------+--------+--------+~~~~~~~~~~~~~+

    a map whose length is indefinite-length
    +------+~~~~~~~~~~~~~+------+
    | 0xbf | N*2 objects | 0xff |
    +------+~~~~~~~~~~~~~+------+

    * N is the size of a map

    represent the abstract definite-length map of multi depth
    {"Fun":1, "Amt":{"Foo":1, "Bar":2}, "Pius":{"tall":185, "weight":95}}
    +------+------+------+------+------+------+------+------+------+------+------+
    | 0xa3 | 0x63 | 0x46 | 0x75 | 0x6e | 0x01 | 0x63 | 0x41 | 0x6d | 0x74 | 0xa2 | ~
    +------+------+------+------+------+------+------+------+------+------+------+
    | 0x63 | 0x46 | 0x6f | 0x6f | 0x01 | 0x63 | 0x42 | 0x61 | 0x72 | 0x02 | 0x64 | ~
    +------+------+------+------+------+------+------+------+------+------+------+
    | 0x50 | 0x69 | 0x75 | 0x73 | 0xa2 | 0x64 | 0x74 | 0x61 | 0x6c | 0x6c | 0x18 | ~
    +------+------+------+------+------+------+------+------+------+------+------+
    | 0xb9 | 0x77 | 0x65 | 0x77 | 0x69 | 0x67 | 0x68 | 0x74 | 0x18 | 0x5f |
    +------+------+------+------+------+------+------+------+------+------+

    explain each byte
    +------+
    | 0xa3 | -- map of 3 elements
    +------+------+------+------+------+
           | 0x63 | 0x46 | 0x75 | 0x6e | -- "Fun"
           +------+------+------+------+
           | 0x01 | -- 1
           +------+------+------+------+
           | 0x63 | 0x41 | 0x6d | 0x74 | -- "Amt"
           +------+------+------+------+
           | 0xa2 | -- map of 2 elements
           +------+------+------+------+------+
                  | 0x63 | 0x46 | 0x6f | 0x6f | -- "Foo"
                  +------+------+------+------+
                  | 0x01 | -- 1
                  +------+------+------+------+
                  | 0x63 | 0x42 | 0x61 | 0x72 | -- "Bar"
                  +------+------+------+------+
                  | 0x02 | -- 2
           +------+------+------+------+------+
           | 0x64 | 0x50 | 0x69 | 0x75 | 0x73 | -- "Pius"
           +------+------+------+------+------+
           | 0xa2 | -- map of 2 elements
           +------+------+------+------+------+------+
                  | 0x64 | 0x74 | 0x61 | 0x6c | 0x6c | -- "Tall"
                  +------+------+------+------+------+
                  | 0x18 | 0xb9 | -- 185
                  +------+------+------+------+------+------+
                  | 0x65 | 0x77 | 0x69 | 0x67 | 0x68 | 0x74 | -- "Bar"
                  +------+------+------+------+------+------+
                  | 0x18 | 0x5f | -- 95
                  +------+------+


    represent the abstract indefinite-length map of multi depth
    {"Fun":1, "Amt":{"Foo":1, "Bar":2}, "Pius":{"tall":185, "weight":95}}
    +------+------+------+------+------+------+------+------+------+------+------+
    | 0xbf | 0x63 | 0x46 | 0x75 | 0x6e | 0x01 | 0x63 | 0x41 | 0x6d | 0x74 | 0xa2 | ~
    +------+------+------+------+------+------+------+------+------+------+------+
    | 0x63 | 0x46 | 0x6f | 0x6f | 0x01 | 0x63 | 0x42 | 0x61 | 0x72 | 0x02 | 0x64 | ~
    +------+------+------+------+------+------+------+------+------+------+------+
    | 0x50 | 0x69 | 0x75 | 0x73 | 0xbf | 0x64 | 0x74 | 0x61 | 0x6c | 0x6c | 0x18 | ~
    +------+------+------+------+------+------+------+------+------+------+------+
    | 0xb9 | 0x77 | 0x65 | 0x77 | 0x69 | 0x67 | 0x68 | 0x74 | 0x18 | 0x5f | 0xff | ~
    +------+------+------+------+------+------+------+------+------+------+------+
    | 0xff |
    +------+

    explain each byte
    +------+
    | 0xbf | -- map of indefinite-length
    +------+------+------+------+------+
           | 0x63 | 0x46 | 0x75 | 0x6e | -- "Fun"
           +------+------+------+------+
           | 0x01 | -- 1
           +------+------+------+------+
           | 0x63 | 0x41 | 0x6d | 0x74 | -- "Amt"
           +------+------+------+------+
           | 0xa2 | -- map of 2 elements
           +------+------+------+------+------+
                  | 0x63 | 0x46 | 0x6f | 0x6f | -- "Foo"
                  +------+------+------+------+
                  | 0x01 | -- 1
                  +------+------+------+------+
                  | 0x63 | 0x42 | 0x61 | 0x72 | -- "Bar"
                  +------+------+------+------+
                  | 0x02 | -- 2
           +------+------+------+------+------+
           | 0x64 | 0x50 | 0x69 | 0x75 | 0x73 | -- "Pius"
           +------+------+------+------+------+
           | 0xbf | -- map of indefinite-length
           +------+------+------+------+------+------+
                  | 0x64 | 0x74 | 0x61 | 0x6c | 0x6c | -- "Tall"
                  +------+------+------+------+------+
                  | 0x18 | 0xb9 | -- 185
                  +------+------+------+------+------+------+
                  | 0x65 | 0x77 | 0x69 | 0x67 | 0x68 | 0x74 | -- "Bar"
                  +------+------+------+------+------+------+
                  | 0x18 | 0x5f | -- 95
           +------+------+------+
           | 0xff | -- break (inner map)
    +------+------+
    | 0xff | -- break (outer map)
    +------+




#### Indefinite-Length Arrays and Maps

The end of the indefinite-length array or map is indicated by encoding a
"break" stop code in a place where the next data item would normally have been
included.


#### Indefinite-Length Byte Strings and Text Strings

Indefinite-length byte strings and text strings are actually a concatenation
of zero or more definite-length byte or text strings ("chunks") that are
together treated as one contiguous string.

The end of the series of chunks is indicated by encoding the "break" stop code
in a place where the next chunk in the series would occur.

The contents of the chunks are concatenated together, and the overall length of
the indefinite-length string will be the sum of the lengths of all of the chunks.
