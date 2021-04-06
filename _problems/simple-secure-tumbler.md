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
Coin swap protocols guarantee that no party has a way of stealing money -- either Bob gets Carol's coins and Carol gets Alice's or the coins are returned.
Blockchain observers should be unable to easily link the payment from Alice to Carol with the payment from Carol to Bob.

Coin swaps only provide privacy if the person in the middle (Carol) keeps her involvement secret.
A *tumbler* is a more sophisticated version of Carol where if she is executing multiple swaps concurrently she cannot tell which incoming coins are linked to which outgoing coins.
She is guaranteed that the incoming are coins equal to (or greater) than the outgoing coins but cannot link any particular incoming transaction to any outgoing transaction.
In other words users are not trusting the tumbler to maintain confidentiality they are only relying on there being multiple payments being made through the tumbler at the same time.
Of course a malicious tumbler could only respond to a single request during a period to try and link the one incoming payment to the single outgoing payment but this would at least be detectable (note coinjoins have the same problem).
Tumblers effectively provides the same privacy enhancement as a *[chaumian coinjoin]* except with the following advantages:

1. The transactions involved may be indistinguishable from normal transactions (the inputs and at least not grouped together in a single transaction like in a coinjoin).
2. Tumblers can be used on *layer-2* networks like [Lightning] to help make payments more unlinkable. Additionally, They can even be used between on-chain and off-chain funds or even payments between chains (i.e. *atomic swaps*).

The only disadvantage is that the throughput is limited by the coins the tumbler has available.

The practical problem with Tumbler protocols is that they are either complicated (*TumbleBit*) or use exotic cryptographic primitives (*A²L*) or their security has not been proved (*Partially Blind Atomic Swap*).
This problem is considered solved by the publication of a protocol that uses the Bitcoin crypto stack (secp256k1, SHA256 etc) and has a security proof.

## Impact

A secure Tumbler protocol that only uses the Bitcoin crypto stack has the potential to significantly improve Bitcoin's privacy story both on-chain and off-chain.


## Potential Directions

### Prove Nick's scheme secure

In *[Partially Blind Atomic Swap Using Adaptor Signatures]* Jonas Nick sketches a tumbler protocol which is effectively solves this problem.
It composes MuSig, adaptor signatures and Schnorr blind singatures together.
Proving this secure is not straightforward especially since Schnorr blind signatures are insecure on their own (as noted in the proposal itself).
Fortunetly, Fuchsbauer et al. present a concrete solution to this problem in *[Blind Schnorr Signatures and Signed ElGamal Encryption in the Algebraic Group Model]*.
Therefore proving this scheme (or a refinement of it) secure requires showing that the security of each individual component is preserved when composed in the way the proposal suggests.

## Proposed Solutions

*There are no proposed solutions for this problem*

## Related Research

- [*TumbleBit: An Untrusted Bitcoin-Compatible Payment Hub*](https://eprint.iacr.org/2016/575.pdf) introduced the concept of a Tumbler and provided the first secure protocol. The lock de-linking is implemented with RSA encryption and a cut-and-choose protocol.
- [*A²L: Anonymous Atomic Locks for Scalability in Payment Channel Hubs*](https://eprint.iacr.org/2019/589.pdf) demonstrated a simpler protocol based where the lock de-linking is done through homomorphic Castangnos-Laguillaumie encryption of a Bitcoin secret key.
- *[Partially Blind Atomic Swap Using Adaptor Signatures]* presents what is effectively a Tumbler protocol that only uses the Bitcoin crypto stack. Unfortunately it relies on Schnorr Blind Signatures which are known to be insecure.
- *[Blind Schnorr Signatures and Signed ElGamal Encryption in the Algebraic Group Model]* formally demonstrates a modified blind signing algorithm that is not broken.

## Commentary


<!-- This is where you can post choice informal and opinionated comments from various sources on the problem. -->
<!-- Also you or anyone else can add conjecture to this section (after review). -->
<!-- In general, this is not a comments section (use the issue for that). -->

*There has been no commentary on this problem so far*

## Related Problems

*There is currently no related problems*

[Lightning]: https://en.wikipedia.org/wiki/Lightning_Network
[coin swaps]: https://gist.github.com/chris-belcher/9144bd57a91c194e332fb5ca371d0964
[submarine swaps]: https://wiki.ion.radar.tech/tech/research/submarine-swap
[chaumian coinjoin]: https://bitcoinops.org/en/topics/coinjoin/
[Partially Blind Atomic Swap Using Adaptor Signatures]:https://github.com/ElementsProject/scriptless-scripts/blob/master/md/partially-blind-swap.md
[Blind Schnorr Signatures and Signed ElGamal Encryption in the Algebraic Group Model]: https://eprint.iacr.org/2019/877.pdf
