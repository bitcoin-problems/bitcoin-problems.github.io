---
layout: problem
title:  "Secure Fee-Bumping for L2s"
tags: lightning
status: open
maintainer: ariard
issue: 16
---

Layer-2 protocols like [Lightning] treat the blockchain as a dispute resolution and settlement layer.
Only when their counterparty is unreachable or uncooperative does a participant have to unilaterally broadcast a transaction.
Protocols like [Lightning] require a participant to confirm a transaction within a certain time so the coins are distributed correctly.
So how can layer-2 protocol designers guarantee an opportunity to confirm time sensitive transactions to participants under reasonable assumptions?
Since miners prioritize transactions based on their feerate, the only effective way is to make sure participants can control the fee of these transactions unilaterally.
Ideally they should even be able to increase it after the transaction has been broadcast to the network should the original fee be insufficient.

There are two approaches available for fee bumping in Bitcoin today:

1. *Replace-by-Fee* ([RBF]): The user simply signs a new transaction that is likely to replace the old one since it pays a higher fee.
2. *Child-Pays-For-Parent* ([CPFP]): The original transaction remains unchanged but a new transaction with a higher fee is signed which spends from the old one increasing the average feerate of the pair.

The goal of a researcher considering this problem is to design a method for ensuring crucial layer-2 transactions are included in the chain in a timely fashion without all the complexity that comes with the present state of affairs.
Let's look at the implications of the current methods have on layer-2 protocols.

### Pre-signed Transactions

Often the transaction an honest party is racing against the clock to confirm is a *pre-signed* transaction.
It was signed when their counterparty was cooperative -- now they need to go on chain precisely because they are not cooperative.
If fees become persistently higher than the pre-signed transaction's fees the party may miss their deadline.
The obvious but suboptimal solution is to monitor fees closely and update any time-sensitive pre-signed transactions with the counterparty while they are online (this method is available via the [`update_fee`] message in lightning today).

It is much more desirable to allow the party to spontaneously increase the fee at will.
This can be done with [CPFP].
For example, in the [Lightning] protocol commitment transactions have [anchor outputs] so they can be easily fee bumped through CPFP.
Although this works it is rather complex to design and implement.
Furthermore, users have to keep unattached fee-bumping UTXOs around in their wallet in case their need to do a fee-bump via CPFP.

### Resource Usage of Replacement

Allowing a mempool transaction to be replaced by higher fee paying transaction may open up Bitcoin network nodes to *denial of service* (DoS) attacks.
Without replacement, a malicious party's ability to get your node to process a transaction is limited by the number of UTXOs they own.
With replacement an attacker can repeatedly spam your node with transactions that replace previous ones even if they only own a single UTXO forcing you to spend time verifying them.

To prevent effective denial of service attacks, replacement transactions have to follow some rather strict rules as defined in [BIP125].
In particular a replacement transaction must pay a higher absolute fee (rule 3) and a higher feerate (rule 4) than the transaction it is evicting.
In addition, it must not evict more than 100 transactions (rule 5).

For layer-2 protocols these rules had the unintended consequence of creating "pinning" attacks (see [Related Research](#related-research)).
For example, a malicious party could add 100 descendants to a transaction that conflicts with the one the honest party needs confirm.
Even if the honest party pays a large fee it still won't evict the malicious party's transaction since that would evict more than 100 transactions which breaks rule 5.
Rule 3 may be abused as well by creating a very low feerate (so miners won't include it) but high absolute fee (so it won't be evicted) transaction that conflicts with the honest party's transaction.

The Bitcoin core developers instinctively introduced an exception to BIP125's 5th rule called the *[CPFP carve out]* to allow lightning designers to mitigate pinning attacks to some extent.
Unfortunately, even with the carve out rule it will not be an effective solution until the Bitcoin network supports [package relay].

## Impact

The principles layer-2 protocol designers use to work around the [BIP125] rules an esoteric domain understood by only a handful of people.
Even those practiced in the occult [have made mistakes](https://lists.linuxfoundation.org/pipermail/lightning-dev/2020-September/002796.html) trying to design around them.
Having a simple set of principles would increase the confidence in the resulting protocols and allow a wider group to understand and build layer-2 systems.

## Potential Directions

1. **Reconsidering mempool and relay rules**: It is possible that mempool rules could be made less problematic for layer-2 protocol designers without harming the Bitcoin system.
2. **Consensus changes**: It may be possible introduce new ways of increasing an already broadcasted transaction's fee through a consensus change (see [Proposed Solutions](#proposed-solutions)).

## Proposed Solutions

In *[A Replacement for RBF and CPFP: Non-Destructive TXID Dependencies for Fee Sponsoring]* Rubin proposes allowing a transaction to sponsor another.
The sponsor transaction can only be included in a block if the transaction it is sponsoring is in it too.
The sponsor transaction's fees then increase the overall feerate of the pair of transactions.
The advantage of this idea is that it is always available to any transaction without the protocol designer having to consider fee bumping.

## Related Research

1. In *[CPFP Carve-Out for Fee-Prediction Issues in Contracting Applications (eg Lightning)]* Corallo introduces the idea of the [CPFP carve out] rule to help Lightning.
2. In *[RBF Pinning with Counterparties and Competing Interest]* Corallo describes a novel pinning attack against Lightning.
3. In *[Pinning : The Good, The Bad, The Ugly]* Riard further describes the problem of pinning attacks and gives several examples relevant to Lightning.
4. The *[Mempool and mining]* article on the Bitcoin core development wiki Daftuar provides the rationale for why the mempool rules are the way they are.
5. In *[A Stroll through Fee-Bumping Techniques : Input-Based vs Child-Pay-For-Parent]* Riard evaluates evaluates CPFP vs "Input based" fee bumping techniques (mutating the transaction to have additional inputs).

## Related Problems

*There is currently no related problems*

## Commentary

- [@ariard](https://github.com/ariard) provides some [useful commentary](https://github.com/bitcoin-problems/bitcoin-problems.github.io/pull/13#discussion_r641696641) on security models in the original pull request for this text.

[BIP125]: https://github.com/bitcoin/bips/blob/master/bip-0125.mediawiki
[`update_fee`]: https://github.com/lightningnetwork/lightning-rfc/blob/master/02-peer-protocol.md#updating-fees-update_fee
[Pinning : The Good, The Bad, The Ugly]: https://lists.linuxfoundation.org/pipermail/lightning-dev/2020-June/002758.html
[RBF Pinning with Counterparties and Competing Interest]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-April/017757.html
[A Replacement for RBF and CPFP: Non-Destructive TXID Dependencies for Fee Sponsoring]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-September/018168.html
[CPFP Carve-Out for Fee-Prediction Issues in Contracting Applications (eg Lightning)]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-November/016518.html
[Mempool and Mining]: https://github.com/bitcoin-core/bitcoin-devwiki/wiki/Mempool-and-mining
[CPFP carve out]: https://bitcoinops.org/en/topics/cpfp-carve-out/
[Lightning]: https://en.wikipedia.org/wiki/Lightning_Network
[RBF]: https://bitcoinops.org/en/topics/replace-by-fee/
[CPFP]: https://bitcoinops.org/en/topics/cpfp/
[anchor outputs]: https://bitcoinops.org/en/topics/anchor-outputs/
[package relay]: https://bitcoinops.org/en/topics/package-relay/
[A Stroll through Fee-Bumping Techniques : Input-Based vs Child-Pay-For-Parent]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-May/019031.html
