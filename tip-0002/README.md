<pre>
TIP: 2
Title: Hierarchical Deterministic Wallet (HD Wallet)
Author: Nicholas Edmonds 
Status: Draft
Type: Process
Created: 2022-07-20
License: CC-BY-4.0
</pre>

## Abstract

This TIP specifies the implementation of Topl's heirarchical deterministic wallet. Building on Bitcoin's BIP-0032 and improvements to allow the use of Ed25519 keys, this allows for a tree of keypairs to be generated from a given seed and a wallet structure to be added that is compliant for use with the Topl blockchain and other wallets or applications.

## Motivation

Based on the specific asymmetric key cryptography and architectural decisions made in creating the Topl blockchain protocol, an amalgamation of standards is needed to properly implement a hierarchical deterministic wallet that is compatible with the Topl blockchain. This document describes in detail the standards and implementation-level specifics required to implement an HD wallet that is fully compatible and interoperable with the Topl blockchain and ecosystem.

## Specification

### Steps
1. Entropy is a fixed-length byte array derived from a [BIP-0039] compliant mnemonic

2. The user provides an arbitrary-length passphrase, encoded as a UTF-8 string

3. The entropy and passphrase are passed as inputs to a key derivation function to produce a seed. The KDF used is PBKDF2-HMAC-SHA512 specified in [RFC 4231] with the number of iterations being 4096 and the derived key length being 96.
``

## Compatibility

## References

[BIP-0039]: Mnemonic code for generating deterministic keys

[BIP-0039]: https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki


[RFC 4231]: Test vectors for HMAC-512 key derivation function

[RFC 4231]: https://datatracker.ietf.org/doc/html/rfc4231

## Copyright

This work is licensed under a
[Creative Commons Attribution 4.0 International License][cc-by].

[cc-by]: https://creativecommons.org/licenses/by/4.0/

