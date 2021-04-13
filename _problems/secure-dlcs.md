---
layout: problem
title:  "Secure DLCs"
tags: DLC
status: open
maintainer: LLFourn
issue: 8
---

In *[Discreet Log Contracts]* (DLC), Dryja presents a compelling design for Bitcoin based smart contracts that settle based on real world events.
Trusted third parties called *oracles* map the real world events to cryptographic conditions that are used to settle the bets.
Unfortunately, the original work did not contain security proofs and suffers from several [attacks]({{ page.url }}#attacks-on-dlcs).

Using the language from [DLC specification] the idea works like this:

1. An oracle *announces* an event by signing a message containing the details of the event (e.g. what possible outcomes there may be) and some cryptographic data (a nonce).
2. Users take these announcements (perhaps from several oracles) and derive *attestation points* for each possible outcome.
3. Two users who wish to make a bet jointly put funds into a 2-of-2 multi-signature output and pre-sign *Contract Execution Transactions* (CETs) spending from the output.
   Each CET is locked by an, or combination of, attestation points (In the original work through a key combination but in the [current specification][DLC specification] through *[adaptor signatures]*).
4. Once the event transpires the oracle observes the real world outcome and reveals the *attestation secret* for the attestation point corresponding to the outcome.
5. Users then use the attestation secret(s) to unlock the corresponding CET signature and broadcast it to the network.


One of the interesting ideas from the paper is that attestation points can be combined to create a conjunction.
By taking the point `A1` for event `A` and combining it with `B3` for event `B` you could create a new condition `A1 + B3` that represents the events having these respective outcomes.
Events `A` and `B` could even be from different oracles.
The combination of attestation points is one of the more challenging aspects of security analysis.

### Security Notions

To be secure a DLC protocol must at least satisfy these two properties:

1. Bet security: If the oracle or oracles are honest then the coins are distributed at the end of the protocol according to the agreement made between the two users.
2. Oracle accountability: Any user wronged by a malicious oracle must be able to hold them accountable. It must be impossible to hold an honest oracle to account for something they didn't do.

Note in the [original work][Discreet Log Contracts], a strong notion of oracle accountability is acheived such that a user who obtains attestations for two contradictory outcomes is able to extract the oracle's static public key from it.

### Attacks on DLCs

So far the following attacks have been demonstrated or conjectured to exist against the original work:

- **User-on-User rogue key attack**: A user can pick their public key to cancel out the oracle's attestation so that it doesn't need the oracle's attestation to claim the funds.[^1]
- **Oracle-on-Oracle rogue key attack**: An oracle can choose their announcement nonce for an event to cancel out an attestation point of another oracle such that if the two are events are combined then the malicious oracle will be able to produce the combined attestation by itself.[^2]
- **Oracle signature forgery**: A malicious party who has control over what the oracle attests to may get them to attest to a combination of outcomes such that they can forge a Schnorr signature by combining the attestations. They do this by solving a k-sum problem[^4] such that the sum of the Schnorr hashes equals their target value (note they can compute the Schnorr hashes ahead of time because the oracle includes the nonce in the announcement)[^3].
- **Fraud proof forgery by collision**: A malicious user may be find two sets of different attestation points for the same set of announcements that add up to the same point. Therefore, if the oracle honestly attests to one combination they will have implicitly attested to the other. The user may then present the attestation for the malicious combination as proof of fraud. Once again, this employs an algorithm for solving the k-sum problem[^3].

## Impact

The existing [DLC specification] effort has been impeded by having to consider security matters while
A scheme with security proofs would allow users and engineers to be confident in the construction of predication markets and financial contracts on Bitcoin.

## Potential Directions

<!-- - The main use of listing hand-wavy directions is useful to further explore the problem. -->
*There are no speculative directions towards solving this problem*

## Proposed Solutions

*There are no proposed solutions for this problem*

## Related Research

1. Dryja introduces the idea in *[Discreet Log Contracts]*.
2. Fournier demonstrates the User-on-User rogue key attack in *[How to Make a Prediction Market on Twitter with Bitcoin]*.
3. Fournier introduces a basic notions of security in *[Security of Discreet Log Contract Attestation Schemes]* and demonstrates the Oracle-on-Oracle rogue key attack.
4. Chalkias et al. demonstrate how to securely combine the `s` values from many Schnorr signatures into a single one in *[Non-interactive half-aggregation of EdDSA and variants of Schnorr signatures]*. The original DLC paper suggests to naively add `s` values together which leads to the collision problem -- this paper shows a secure way to do it.

## Commentary

*There has been no commentary on this problem so far*

## Related Problems

*There is currently no related problems*

## Footnotes

[^1]: First [pointed out on irc](https://freenode.irclog.whitequark.org/bitcoin-wizards/2017-06-06) soon after the original work was published. Also discussed in *[How To Make a Prediction Market on Twitter with Bitcoin]*.
[^2]: This attack is discussed in *[Security of Discreet Log Contract Attestation Schemes]*.
[^3]: This attack is yet to be published.
[^4]: k-sum known as the "generalized birthday problem". Minder and Sinclair give applicable algorithms for solving it in *[The extended k-tree algorithm]*.

[DLC specification]: https://github.com/discreetlogcontracts/dlcspecs
[Discreet Log Contracts]: https://adiabat.github.io/dlc.pdf
[How to Make a Prediction Market on Twitter with Bitcoin]: https://github.com/LLFourn/two-round-dlc/blob/master/main.pdf
[Security of Discreet Log Contract Attestation Schemes]: https://github.com/LLFourn/dlc-sec/blob/master/main.pdf
[adaptor signatures]: https://bitcoinops.org/en/topics/adaptor-signatures/
[The extended k-tree algorithm]: https://eprint.iacr.org/2016/312.pdf
[Non-interactive half-aggregation of EdDSA and variants of Schnorr signatures]: https://eprint.iacr.org/2021/350.pdf
