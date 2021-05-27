---
layout: problem
title:  "Replacement rules and Fee Bumping"
tags: lightning
status: open
maintainer: LLFourn
---

Layer-2 protocols like [Lightning] treat the blockchain as a dispute resolution and settlement layer.
Only when their counterparty is unreachable or uncooperative does a participant have to unilaterally broadcast a transaction.
The protocol often requires a party to confirm a transaction within a certain time so the funds are distributed fairly.
So how can layer-2 protocol designers guarantee their users are able to confirm their transaction before the deadline?
Since miners prioritize transactions based on their feerate, the only effective way is to make sure participants can set the fee of time sensitive transactions and ideally even increase it repeatedly after the transaction has been broadcast if conditions change.

There are two approaches available in Bitcoin today.

1. *Replace-by-Fee* ([RBF]): The user simply signs a new transaction that is likely to replace the old one since it pays a higher fee.
2. *Child-Pays-For-Parent* ([CPFP]): The original transaction remains unchanged but a new transaction with a higher fee is signed which spends from the old one increasing the average feerate of the pair.

The goal of a researcher considering this problem should be to design a method for ensuring crucial layer-2 transactions are included in the chain in a timely fashion without all the complexity that the present state of affairs.
Let's look at the implications of the current methods for layer-2 protocol designers.

### Pre-signed Transactions

Often the transaction an honest party is racing against the clock to confirm is a *pre-signed* transaction.
It was signed when their counterparty was cooperative -- now the honest party needs to go on chain precisely because they are not cooperative.
If fees become persistently higher than the pre-signed transaction's fees the party may miss their deadline.
The obvious but suboptimal solution is to monitor fees closely and update any time-sensitive pre-signed transactions with the counterparty while they are online (this method is available via the [`update_fee`] message in lightning today).

It is much more desirable to allow the party to spontaneously increase the fee.
This can be done with [CPFP].
For example, in the [Lightning] protocol commitment transactions are designed with [anchor outputs] so they can be easily fee bumped through CPFP.

### Resource Usage of Replacement

Allowing a mempool transaction to be replaced by higher fee paying transactions is that this may open up nodes to *denial of service* (DoS) attacks.
Without replacement a malicious node's ability to get your node to process a transaction is limited by the number of UTXOs they own.
With replacement an attacker can repeatedly spam your node with transactions that replace previous ones even if they only own a single UTXO.

To prevent effective denial of service attacks, replacement transactions have to follow some rather strict rules as defined in [BIP125].
In particular a replacement transaction must pay a higher absolute fee and a higher feerate than the transactions they are evicting and it must not evict more than 100 transactions.

For layer-2 protocols this rules had the unintended consequence of introducing  "pinning" attacks (see [Related Research](#related-research)).
This is usually done by adding a lot of descendants to a transaction that conflicts with the one the honest party needs confirm.
As we have said an honest party needs to confirm a transactions in a certain time -- if it can't evict a maliciously crafted transaction then they are in a helpless position.

An instinctive exception to [BIP125]'s rules called the [CPFP carve out] was introduced to allow lightning designers to avoid pinning attacks with some extra design effort.

## Impact

The principles layer-2 protocol designers should use to enable transactions to confirm on time are an esoteric domain understood by only a handful of people.
Even those practiced in the occult [have made mistakes](https://lists.linuxfoundation.org/pipermail/lightning-dev/2020-September/002796.html) trying to design around them.
Having a simple set of principles would increase the confidence in the resulting protocols and allow a wider group to understand and build layer-2 systems.

## Potential Directions

1. **Reconsidering mempool and relay rules**: It is possible that mempool rules could be made less problematic for layer-2 protocol designers without harming the Bitcoin system.
   The main concern with this approach is that the rules that are optimal for layer-2 protocols may not be the rules that are optimal for miner profit or preventing DoS attacks.
   However, if tweaked relay rules were to allow layer-2 Bitcoin protocols to thrive then the increased value of the Bitcoin network could more than compensate miners for using a slightly suboptimal algorithm (as long as they don't introduce DoS attacks).
2. **Consensus changes**: It may be possible introduce new ways of increasing an already broadcasted transaction's fee through a consensus change (see [Proposed Solutions](#proposed-solutions)).

## Proposed Solutions

In *[A Replacement for RBF and CPFP: Non-Destructive TXID Dependencies for Fee Sponsoring]* Rubin proposes allowing a transaction to sponsor another.
The sponsor transaction can only be included in a block if the transaction it is sponsoring is in it too.
The sponsor transaction's fees then increase th overall feerate of the pair of transactions.
The advantage of this idea is that it is always available to any transaction without the protocol designer having to consider fee bumping as part of the design.

## Related Research

1. In *[RBF Pinning with Counterparties and Competing Interest]* Corallo introduces the idea of a pinning attack.
2. In *[Pinning : The Good, The Bad, The Ugly]* Riard describes the problem of pinning attacks and gives several examples relevant to Lightning.
3. The *[Mempool and mining]* article on the Bitcoin core development wiki Daftuar provides the rationale for why the mempool rules are the way they are.

<!-- This is where you can post relevant informal and opinionated comments from various sources on the problem. -->
<!-- Also you or anyone else can add conjecture to this section (after review). -->
<!-- In general, this is not a comments section (use the issue for that). -->

*There has been no commentary on this problem so far*


## Related Problems

*There is currently no related problems*

[BIP125]: https://github.com/bitcoin/bips/blob/master/bip-0125.mediawiki
[`update_fee`]: https://github.com/lightningnetwork/lightning-rfc/blob/master/02-peer-protocol.md#updating-fees-update_fee
[Pinning : The Good, The Bad, The Ugly]: https://lists.linuxfoundation.org/pipermail/lightning-dev/2020-June/002758.html
[RBF Pinning with Counterparties and Competing Interest]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-April/017757.html
[A Replacement for RBF and CPFP: Non-Destructive TXID Dependencies for Fee Sponsoring]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-September/018168.html
[Mempool and Mining]: https://github.com/bitcoin-core/bitcoin-devwiki/wiki/Mempool-and-mining
[CPFP carve out]: https://bitcoinops.org/en/topics/cpfp-carve-out/
[Lightning]: https://en.wikipedia.org/wiki/Lightning_Network
[RBF]: https://bitcoinops.org/en/topics/replace-by-fee/
[CPFP]: https://bitcoinops.org/en/topics/cpfp/
[anchor outputs]: https://bitcoinops.org/en/topics/anchor-outputs/
