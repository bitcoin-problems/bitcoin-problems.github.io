---
layout: problem
title:  "Channel balance probing"
tags: lightning privacy
status: "open"
maintainer: s-tikhomirov
issue: 24
---

Balances of Lightning channels are not revealed during normal protocol operation. Hence LN users may have a somewhat grounded perception that their balances are private. Indeed, as LN payments are not publicly broadcast (contrary to L1 Bitcoin transactions), Lightning may be seen as not only scalability solution but also a privacy enhancing technology.

In reality, however, channel balance privacy can be violated in the following scenario. The attacker sends a series of payments (probes) with randomly generated payment hashes. Each probe fails because of either insufficient balance at an intermediary hop, or because the final receiver doesn't know the respective preimage. By distinguishing between these two cases, the attacker performs binary search over all possible positions of the balance (between zero and channel capacity).

{% include figure.html image="probing-attack.svg" name="Channel balance probing attack" caption="The attacker makes a series of probes following the binary search pattern and improves their estimates (colored interval) of the position of the true balance (star) within the channel (outer interval)."
%}

There are multiple variations of the probing attack:
- the victim is the receiver vs the attacker controls the receiving node;
- establishing a direct channel to the victim vs sending probes along multi-hop paths;
- simple binary search vs more advanced techniques to account for parallel channels.

Besides threatening individual users' privacy, probing may be the basis for more advanced network-wide attacks. For instance, an attacker might take frequent balance snapshots of the LN graph and reveal full payment paths, as balances along each path are shifted by the same amount (modulo fees) at around the same time. While it may be challenging to take such snapshots frequently enough for the entire graph, the attacker may instead focus on the sub-graph between a minority of highly-connected nodes that play an important role in routing.

Preventing probing may have a negative effect on payment reliability. Lightning wallets may perform probing to populate their local network graph with balance estimations, which would allow them to increase the chance of payment success. 

One may argue that updating balance estimations for remote channels after an honest payment failure also constitutes probing. It probably makes sense though to distinguish between "malicious" probing (deliberately spending network resources on payments that by definition cannot succeed) vs probing as a side-effect of normal operation.

Ideally, it would be desirable to prevent deliberate probing while ensuring payment reliability (through multi-part payments, choosing paths with high success probability, etc).


## Impact

Preventing probing would enhance Lightning privacy and resolve the apparent contradiction between the perceived secrecy of balances and the fact that they can be easily revealed.


## Potential Directions

Probing is based on the following assumptions:
- erring nodes truthfully report error location and error type;
- channel balances don't shift very often;
- there is at most one channel between any pair of nodes (for "naive" probing algorithm);
- failed payment attempts are free.

Potential preventive measures may aim at invalidating these assumptions (see [Proposed Solutions](#proposed-solutions)).


## Proposed Solutions

### Error handling

Probing is based on distinguishing between reasons of payment failure. A generic security advice is to hide any details on why an error occurred from users. However, for the LN specifically, balance errors are important for reliability: senders use them to improve their network maps. This trade-off between privacy and reliability should be carefully evaluated.

The most simplistic countermeasure would be to eliminate error messages altogether. This would harm reliability. Moreover, without receiving an error message, the sender doesn't know whether the payment is stuck or has failed, and therefore can't decide whether it's time to make another attempt.

A more realistic variation of this idea could be to merge the two error types (low balance / unknown hash) into one. This wouldn't be very helpful either, because a) the attacker can tell the two scenarios apart based solely on _where_ the error originated, regardless of its type; b) the attacker who also controls the receiving node doesn't depend on error messages at all.

Assuming the attacker does _not_ control the receiver, routing nodes may hinder probing by forging the error _location_ in error messages. For example, for a probe going via the route Alice-Bob-Charlie-David and failing at Charlie (who has low balance in the channel to David), Bob would replace Charlie's error message, pretending that _he_ lacks balance in the channel to Charlie. This would indeed confuse Alice (the prober), but such measure is not incentive-compatible. Bob essentially "takes the blame" for the failed payment, willfully decreasing reliability metrics (and future routing fees) for his own node.

While the drawbacks of these naive proposals likely outweigh their benefits, they may be iterated upon or mixed with other ideas to arrive at a satisfactory solution.


### Parallel channels

LN allows a pair of nodes to share multiple channels, which are called parallel. Payment forwarding in Lightning is _non-strict_: if Alice has multiple channel to Bob, she may choose any of them to forward a payment. This complicates probing: the attacker doesn't know which channel estimate to update.

Parallel channels may be used as an anti-probing countermeasure. Two modes may advertise just one channel but maintain multiple unannounced ("private") channels that actually do the forwarding. The balance of the public channel is thus kept secret. More advanced parallel channel configurations may be possible.

A pair of nodes sharing multiple channels may also split a payment in multiple parts within their hop (intra-hop split; this hasn't been implemented).


### Rebalancing and JIT-routing

Channel rebalancing means sending (potentially circular) payments to shift the distribution of funds in one's channels. In [Just-in-time (JIT) routing], nodes do rebalancing during payment forwarding (hence, just-in-time), as opposed to doing it preemptively. If asked to forward a payment that larger than any single channel balance, a JIT-routing node concentrates enough funds in one of its channels and then continues forwarding. From the prober's standpoint, rebalancing changes the properties of a hop while probing is happening, hence the attacker's prior estimates get invalidated. However, the attacker may still reveal the sum of the hop's balances. (What exactly can be revealed in this case depends on which channel directions are enabled - see Section 5.4 in *[Analysis and Probing of Parallel Channels in the Lightning Network]*.)

In general, probing assumes that the balance of the victim channel doesn't change while the attack is happening. In realistic scenarios, one probe takes a few seconds. Probing a 1-million satoshi channel with 1-thousand satoshi precision (10 binary search steps, i.e. probes) shouldn't take more than a minute. Rebalancing or other techniques that temporarily shift channel balances at least once a minute could therefore be considered a countermeasure against probing.


### Anti-jamming measures

Another assumption behind probing is that failed payment attempts are free (probes are by definition failed payments). This makes probing essentially free. A closely related line of research investigates ways to limit [Channel jamming attacks] in Lightning (see also: [Preventing Channel Jamming]). Anti-jamming measures include upfront fees, stake certificates, and reputation tokens.

Anti-jamming measures would also deter probing because probes are just another type of unwanted messages in the LN protocol alongside jams. However, the utility of anti-jamming measures against probing may be moderate: the probing attack requires much fewer probes than jamming attack requires jams. Probing a single channel only requires tens of probes (depending on channel capacity and desired precision), whereas jamming a channel requires hundreds of jams (assuming slot-based jamming).


### Trust-based solutions

To avoid getting probed and otherwise attacked, routing nodes may decide to only forward payments from trusted peers. This would be an undesirable development, as arguably the whole point of Lightning (and Bitcoin) is to be permissionless money. However, LN nodes have (relatively) persistent identities, and forwarding payments implies renting scarce resource (liquidity) to strangers. Therefore it's likely for trust-based solutions to emerge organically.


### Deliberate balance sharing

We may envision an additional protocol that would allow LN nodes to willingly share their balance (up to some precision) to help others route payments through them. Such a protocol may be seen as a "carrot" complementing "sticks" described above: nodes that want to give up privacy for more reliability may do so in a deliberate way, rather than simply opening themselves up to probing.

An additional line of research here could be using zero-knowledge proofs for privacy-preserving / verifiable balance sharing. One node would prove to the other that its balance lies in a certain range without revealing any more information.


## Related Research

1. Herrera-Joancomartí et al. in *[On the Difficulty of Hiding the Balance of Lightning Network Channels]* introduced the general idea of probing in the LN.
2. Van Dam in *[Improvements of the Balance Discovery Attack on Lightning Network Payment Channels]* suggested probing channels from both ends, which reduces the capital requirement for the attack.
3. Kappos et al in *[An Empirical Analysis of Privacy in the Lightning Network]* proposed controlling both the sender and the receiver of probes. This removes the dependency on error messages: the attacker always knows whether the probe has reached the final destination.
4. Tikhomirov et al. in *[Probing Channel Balances in the Lightning Network]* implemented multi-hop probing. Sending probes along multi-hop paths allows for a) accumulating balance information about multiple channels at once; b) amortizing on-chain fees for channel openings across many target hops.
5. Biryukov et al. in *[Analysis and Probing of Parallel Channels in the Lightning Network]* explored probing of parallel channels. If the target hop contains parallel channels, the prober, in the general case, doesn't know which of them forwarded the probe. This obstacle may be solved with jamming-enhanced probing, i.e., probing parallel channels one by one while blocking the remaining ones.
6. Pickhardt and Richter in *[Optimally Reliable & Cheap Payment Flows on the Lightning Network]* consider success probabilities when choosing payment paths and look at LN path selection as the min-cost flow problem. Their proposed round-based algorithm improves upon the state of the art in terms of payment reliability.


## Commentary

<!-- This is where you can post relevant informal and opinionated comments from various sources on the problem. -->
<!-- Also you or anyone else can add conjecture to this section (after review). -->
<!-- In general, this is not a comments section (use the issue for that). -->

*There has been no commentary on this problem so far*


## Related Problems

[Removing cross-layer links] could allow for eliminating the notion of a channel from the routing protocol. Nodes would advertise the maximal payment size they are willing to forward, and use parallel channels and rebalancing in the background to satisfy their claims.

<!-- TODO: link to channel jamming problem page when it's created. Solving jamming partially helps solve probing (see probing of parallel channels, as well as generic spam prevention). -->


[Channel jamming attacks]: https://bitcoinops.org/en/topics/channel-jamming-attacks/
[Preventing Channel Jamming]: https://blog.bitmex.com/preventing-channel-jamming/
[Just-in-time (JIT) routing]: https://bitcoinops.org/en/topics/jit-routing/
[Optimally Reliable & Cheap Payment Flows on the Lightning Network]: https://arxiv.org/abs/2107.05322
[On the Difficulty of Hiding the Balance of Lightning Network Channels]: https://eprint.iacr.org/2019/328
[Improvements of the Balance Discovery Attack on Lightning Network Payment Channels]: https://eprint.iacr.org/2019/1385
[An Empirical Analysis of Privacy in the Lightning Network]: https://arxiv.org/abs/2003.12470
[Probing Channel Balances in the Lightning Network]: https://arxiv.org/abs/2004.00333
[Analysis and Probing of Parallel Channels in the Lightning Network]: https://eprint.iacr.org/2021/384
