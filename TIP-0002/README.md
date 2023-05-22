<pre>
TIP: 2
Title: Topl Address Encoding
Author: Edmundo López Bóbeda <e.lopez@topl.me>
Status: Accepted
Type: Standard
Created: 2023-04-26
License: CC-BY-4.0
</pre>

## Abstract

This TIP proposes an address serialization format for the TOPL blockchain

## Motivation

Implementors need to understand the format of the addresses and how to encode and decode them to call different services in the node.

## Specification

### Introduction

In the Topl ecosystem, a lock address is a data structure that stores the evidence of a Quivr proposition. Besides the evidence, the lock address data structure stores the network and ledger identifier. This extra information is useful for improving usability and tooling. We bundle together the aforementioned information and then encode into a human readable string. The end user commonly perceives this string as “the address”.

### Binary Format

We encode the information in the address in an array of bytes in the following form:



```
  4 bytes       4 bytes      32 bytes
<-----------> <---------> <-------------> 
┌────────────┬───────────┬───────────────┐
│ network id │ ledger id │ evidence      │
└────────────┴───────────┴───────────────┘
```



We reserved the following values for the standard Topl Networks in the first 4 bytes as the network identifier.

| **Network Name**      | **Byte Values** | **Address prefix** |
| --------------------- | --------------- | ------------------ |
| Main Network          | `0x8A11054C`    | `mtet`             |
| Valhalla Test Network | `0xA5BF4108`    | `vtet`             |
| Private Network       | `0x934B1900`    | `ptet`             |

When encoded, the addresses will feature the prefix in the column labeled **Address prefix**.

The next four bytes are the ledger identifier. We reserve the following value for the main ledger.

| **Ledger Name** | **Byte Values** | **Remarks**                                                  |
| --------------- | --------------- | ------------------------------------------------------------ |
| Main Ledger     | `0xE7B07A00`    | This number has the nice property of being encoded to “main” when encoding in the main net and thus forcing the prefix to be `mtetmain` in main net addresses. However, it does not work that way for the other networks prefixes. |

### Encoding

We do the encoding of the data structure of the lock address using the Base58Check algorithm that is used for Bitcoin and other crypto currencies. The Base58Check algorithm takes as input the binary representation and returns an array of bytes.

1. First compute the double sha256 hash of the payload. We have `doubleHash = sha256(sha256(payload))`.
2. Take the first 4 bytes of the double sha256. We have `checksum = doubleHash.take(4)`.
3. Concatenate the payload to the checksum and encode in [Base58](https://tools.ietf.org/html/draft-msporny-base58-02). We have `encoded = toBase58(payload.concat(checksum))`.

### Examples

The following are valid addresses for the different networks.

#### Example 1

Assuming the payload is a sequence of 32 bytes where all bytes are 0 (`payload = 0x0000000000000000000000000000000000000000000000000000000000000000`), the following are valid encoded addresses for Main, Valhalla and Private network respectively.

- `mtetmain1y1Rqvj9PiHrsoF4VRHKscLPArgdWe44ogoiKoxwfevERNVgxLLh`
- `vtetDGydU3EhwSbcRVFiuHmyP37Y57BwpmmutR7ZPYdD8BYssHEj3FRhr2Y8`
- `ptetP7jshHTuV9bmPmtVLm6PtUzBMZ8iYRvAxvbGTJ5VgiEPHqCCnZ8MLLdi`

#### Example 2

Assuming the payload is a sequence of 32 bytes where all bytes are 1 (`payload = 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF`), the following are valid encoded addresses for Main, Valhalla and Private network respectively.

- `mtetmain1y3Nb6xbRZiY6w4eCKrwsZeywmoFEHkugUSnS47dZeaEos36pZwb`
- `vtetDGydU3Gegcq4TLgQ8RbZ5whA54WYbgtXc4pQGLGHERhZmGtjRjwruMj7`
- `ptetP7jshHVrEKqDRdKAZtuybPZoMWTKKM2ngaJ7L5iZnxP5BprDB3hGJEFr`

#### Example 4

Assuming the payload is the following sequence of bytes `payload = 0xfcd1d7d5495063990f02e7106afe9217707781adda1dc8b966c68546e92de62e`, the following are valid encoded addresses for Main, Valhalla and Private network respectively.

- `mtetmain1y3MBsEzQyC3zNwELhxy5wcDs3a42T1Av2MPzp8BrQga8s5Vj17X`
- `vtetDGydU3GdHP7TSk9v1sU9EKoBHSTnWxfLQE4fVtAtoBi84314kk3LwRSh`
- `ptetP7jshHVpq67cR2ngTLnZjmfpZtQZEcobUjYNZddBMiPdUaxYW3k6M7Ho`

## Backwards Compatibility

The new addresses are not compatible with the Dion network. Indeed, Dion network addresses did not include a ledger identifier.

## Copyright

We license this work under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).
