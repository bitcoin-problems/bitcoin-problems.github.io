---
layout: problem
title:  "Simple Secure Tumbler"
tags: privacy
status: "open"
maintainer: LLFourn
issue: 2
---

Tumblers are services that offer users *[coin swaps]* in a privacy preserving way.
A coin swap protocol lets Alice make a payment to Bob, not by sending it directly but instead sending the coins to Carol who sends completely unrelated coins to Bob.
A coin swap protocol guarantees that no party has a way of stealing money -- either Bob gets Carol's coins and Carol gets Alice's or the coins are returned.
Blockchain observers should be unable to easily link the payment of one address to the other.

Note that Coin swaps only provide privacy if the person in the middle (Carol) keeps secret details of her involvement secret.
A Tumbler is a more sophisticated version of Carol where if she is executing multiple swaps concurrently she cannot tell which incoming coins are linked to which outgoing coins.
This should hold even if she behaves maliciously.
This effectively provides the same privacy enhancement as a *[chaumian coinjoin]* except with the following advantages:

1. The transactions involved may be indistinguishable from normal transactions (the inputs and at least not grouped together in a single transaction).
2. Tumblers can be used on *layer-2* networks like [Lightning] to help make payments more unlinkable. Additionally, They can even be used between on-chain and off-chain funds or even payments between chains (i.e. *atomic swaps*).

The practical problem with the Tumbler protocols they are either complicated and use exotic cryptographic primitives or their security is in question.
This problem is considered solved by protocol that uses the Bitcoin crypto stack (secp256k1, SHA256 etc) and has a security proof. 

## Impact

<!-- - Try not to repeat the description too much -->
<!-- - Make it clear what the impact on the big picture of Bitcoin's evolution would be -->
A secure Tumbler protocol that only uses the Bitcoin crypto stack would significantly improve Bitcoin's privacy story both on-chain and off-chain.

## Proposed Solutions


*There are no proposed solutions for this problem*


## Related Research

- [*TumbleBit: An Untrusted Bitcoin-Compatible Payment Hub*](https://eprint.iacr.org/2016/575.pdf) introduced the concept of a Tumbler and provided the first secure protocol. The lock de-linking is implemented with RSA encryption and a cut-and-choose protocol.
- [*A²L: A2L: Anonymous Atomic Locks for Scalability in Payment Channel Hubs*](https://eprint.iacr.org/2019/589.pdf) demonstrated a simpler protocol based where the lock de-linking is done through homomorphic Castangnos-Laguillaumie encryption of a Bitcoin secret key.
- [*Partially Blind Atomic Swap Using Adaptor Signatures*](https://github.com/ElementsProject/scriptless-scripts/blob/master/md/partially-blind-swap.md) presents what is effectively a Tumbler protocol that only uses the Bitcoin crypto stack. Unfortunately it relies on Schnorr Blind Signatures which are known to be insecure.
- [*Blind Schnorr Signatures and Signed ElGamal Encryptionin the Algebraic Group Model*](https://eprint.iacr.org/2019/877.pdf) formally demonstrates a modified blind signing algorithm that is not broken.

## Commentary

<!-- This is where you can post choice informal and opinionated comments from various sources on the problem. -->
<!-- Also you or anyone else can add conjecture to this section (after review). -->
<!-- In general, this is not a comments section (use the issue for that). -->

*There has been no commentary on this problem so far*

## Related Problems

*There is currently no related problems*

## Footnotes

*There are no footnotes for this article*


[Lightning]: https://en.wikipedia.org/wiki/Lightning_Network
[coin swaps]: https://gist.github.com/chris-belcher/9144bd57a91c194e332fb5ca371d0964
[submarine swaps]: https://wiki.ion.radar.tech/tech/research/submarine-swap
[chaumian coinjoin]: https://bitcoinops.org/en/topics/coinjoin/
