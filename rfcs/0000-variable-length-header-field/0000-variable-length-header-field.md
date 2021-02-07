---
Number: "0000"
Category: Consensus (Hard Fork)
Status: Draft
Author: Ian Yang
Organization: Nervos Foundation
Created: 2021-02-07
---

# Add a variable length field in the block header

## Abstract

This document proposes adding an optional variable length field to the block header.

## Motivation

Currently, the block header is a fixed length structure. Each header consists of 208 bytes.

Many extensions require adding new fields into the block. For example, PoA for testnet requires a 64 bytes signature, and flyclient also needs to add a 64 bytes hash.

There's no enough reserved bits in the header for these extensions. There's a workaround to store these data in the cellbase transaction, but this solution has a big overhead for clients which wants to quickly verify the data using PoW only. If the data are stored in the cellbase transaction, the client has to download the cellbase transaction and the merkle tree proof of the cellbase transaction, which can be larger than the block header itself.

This document proposes a solution to add a variable length field to the block header. How the new field is interpreted is beyond the scope of this document and must be defined and deployed via a future soft fork.

## Design

The block header is encoded as a molecule struct, which consists of fixed length fields. The header binary is just the concatenation of all the fields in sequence.

There are many different ways to add the variable length field to the block header. I have listed some options below for discussion.

### Appending the Field At the End

The block header size is at least 208 bytes. The first 208 bytes are encoded the same as the current header. The remaining bytes are the variable length field.

```
+-----------------------+-----------+
|                       |           |
|    208-bytes header   | New Field |
|                       |           |
+-----------------------+-----------+
```

### Using Molecule Table in New Block Headers

This solution uses a different molecule schema for the new block headers. If the block header size is 208 bytes, it's encoded using the old schema, otherwise it uses the new one. The new schema converts `RawHeader` into a molecule table and adds a variable length bytes field at the end of `RawHeader`.

```
old one:
        208 bytes
+-----+-----------------+
|     |                 |
|Nonce| RawHeader Stuct |
|     |                 |
+-----+-----------------+

new one:

+-----+-------------------------------+
|     |                               |
|Nonce| RawHeader Table               |
|     |                               |
+-----+-------------------------------+
```

### Appending a Hash At the End

Instead of adding the new field directly at the end of the header, this solution adds a 32 bytes hash at the end of the header which is the hash of the new variable length field. The header is still a fixed length struct but is 32 bytes larger. If client does not need the extra field, it only has the 32 bytes overhead. Otherwise it has to download both the header and the extra field and verify that the hash matches.

```
+-----------------------+--+
|                       |  |
|    208-bytes header   | +----+
|                       |  |   |
+-----------------------+--+   |
                               | Hash of
                               |
                               v
                         +-----+-----+
                         |           |
                         | New Field |
                         |           |
                         +-----------+

```

----

TODO

## Specification
## Security
## Test Vectors
## Rationale
## Deployment
## Backward compatibility
## Acknowledgments