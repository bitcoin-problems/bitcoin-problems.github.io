---
layout: problem
title:  "Channel jamming"
tags: lightning denial-of-service
status: "open"
maintainer: s-tikhomirov
issue: 30
---

Jamming is a denial-of-service attack on Lightning channels.
The attacker initiates payments (jams) that consume the resources of routing nodes and then fail.
Jamming decreases the profitability and reliability of routing nodes.
The attack is cheap as failed payments are free.
Preventing jamming is difficult due to Lightning's privacy-preserving features.

In _controlled_ jamming, the attacker controls both the sender and the receiver of jams.
The receiver can keep jams in-flight for multiple hours by not revealing the payment secret.
In _uncontrolled_ jamming, the attacker sends unsolicited "fake" payments to arbitrary targets.
Uncontrolled jams fail within seconds, but they too consume the resources of routing nodes.
The attacker may lock up the resources of routing nodes by sending many uncontrolled jams.
Jamming may also be classified into _capacity-based_ and _slot-based_.
Capacity-based jamming is based on the fact that a payment of `X` coins locks `X` coins in-flight in every channel along the route.
By sending a high-value jam, the attacker depletes the capacity of victim channels.
Slot-based jamming exploits another limited resource, namely, payment slots.
Each channel has up to 483 slots for concurrent in-flight payments.
The attacker initiates many jams that occupy all available payment slots.
Jams have lower value compared to capacity-based jamming, making slot-based jamming capital-efficient.

## Impact

Preventing jamming would improve Lightning reliability and facilitate its widespread adoption.
If jamming is not efficiently prevented, routing nodes could request identification from their peers, diminishing Lightning's privacy guarantees.


## Potential Directions

Preventing jamming is non-trivial for two main reasons.
First, Lightning payments are onion-routed.
Intermediary nodes only know the immediate previous and next node in the route, but not the ultimate sender or receiver.
This means that routing nodes cannot easily rate-limit or ban jammers.
Second, failed payment attempts are free.
The attacker only bears the cost of temporary capital lock-up (which is low for slot-based jamming).

Proposed anti-jamming countermeasures include reputation systems and fees for failed payments.
Both approaches have benefits and drawbacks.
Reputation schemes weaken privacy guarantees, whereas extra fees introduce new game-theoretical and UX challenges.
A practical solution could combine the two approaches.


## Proposed Solutions

### Node configuration

Restricting payment forwarding may deter jamming to some extent.
Node configuration doesn't require changes in the Lightning protocol.
Configuration changes may include:

1. Decrease payment timeouts. The attacker would have to send more jams per unit time to keep a channel jammed (for controlled jamming). The drawback is a shorter time window to claim in-flight payments after channel closure.
2. Increase the minimal payment amount. The attacker would need more capital for slot-based jamming. The drawback is the loss of potential revenue from forwarding honest low-value payments.
3. Divide payment slots into groups for different amount buckets. A routing node would forward small payments through a subset of slots, always keeping some slots open for larger payments. This increases the cost of controlled slot-based jamming. The drawback, again, is lost revenue: some honest payments may be rejected if their slot bucket is full, even if other slots are available.
4. Limit the number of in-flight payments from less trustworthy peers. This may hinder attacks coming from an immediate peer but may also affect honest peers that are unknowingly forwarding jams.

Configuration changes discourage simplistic attacks but don't solve the issue fundamentally.


### Upfront fees

Currently, LN fees are paid only if the payment succeeds.
Upfront fees, in contrast, are paid to initiate a payment.
Upfront fees may be refundable or non-refundable.
Non-refundable upfront fees impose a cost on failed payment attempts.

In the simplest **forward upfront fee** scheme, each node pays a fee to the next node to forward a payment.
Fee amounts should decrease along the route to provide motivation for routing nodes.
If fees are equal on all hops, the attacker who controls the sender and the receiver would pay zero fee.
The drawback of forward upfront fees is that routing nodes are motivated to deliberately fail payments and keep the fee.

In the **reverse upfront fee** scheme, nodes pay to the previous node (that is, for _receiving_ a forwarding request, instead of _sending_ it).
Fee amounts for reverse fees should increase along the route.
The fee should be zero during a grace period: no fees are paid if a payment resolves quickly.
Without a grace period, an uncontrolled jamming attack would force nodes to pay fees with no cost for the attacker.
It is an open question how long the grace period should be and when exactly it should start.

The **bidirectional upfront fee** scheme combines forward and reverse upfront fees.
Forward fees are non-refundable, whereas reverse fees are only refundable within the grace period.

**Nested incremental routing** addresses the incentive compatibility concern with upfront fees (routing nodes deliberately failing payments).
The sender builds a route iteratively, unlike the current protocol, where the route is fully determined before the payment is initiated.
To send a payment to Dave, Alice first communicates with Bob and offers him an upfront fee to forward a payment to Charlie.
Bob then offers an upfront fee to Charlie.
If Charlie cannot forward the payment, Bob tries forwarding it through his other peer Carol, and so on.
At any point, the maximum fee value that a routing node can steal is one-hop worth of upfront fees.
In contrast, if the route is fully determined by the sender, the upfront fee is proportional to the route length, providing a stronger incentive for routing nodes to misbehave (especially those close to the sender).
The drawback of nested incremental routing is high communication cost.
Multiple round trips increase payment latency and facilitate timing-based privacy attack.

UX is a key challenge for upfront fee schemes.
Users may not be prepared to pay for failed payment attempts.
(This may be less of an issue if most attempts succeed.)
Professional routing nodes may also choose to voluntarily refund upfront fees if a payment fails.

Another important consideration is whether fee amounts should be proportional to the time it took a payment to resolve.
Constant amounts are simpler but don't reflect the fact that the longer a payment is held, the more resources it consumes.
Time-dependent fee amounts require a trusted high-precision time source (timestamps in Bitcoin blocks are not fine-grained enough).


### Reputation

**Direct peer reputation** is a scheme by which nodes assign reputation scores to their direct peers.
Bob decreases Alice's reputation if a payment she asked him to forward doesn't resolve quickly.
The fundamental drawback of direct peer reputation is that it punishes honest routing nodes who unknowingly forward jams.
The attacker, in turn, can deny responsibility by claiming to be an honest routing node.

The **provable blaming** scheme aims to properly assign blame for stuck payments.
If a node delays payment resolution, the previous node in the route should close their shared channel.
Consider a route: Alice - Bob - Charlie - Dave.
Suppose that Alice and Dave are cooperating to jam the channel between Bob and Charlie.
Alice initiates the payment, and Dave doesn't reveal the payment secret.
After a grace period, Charlie is supposed to close the channel to Dave.
If Charlie doesn't prove the closure to Bob, Bob should blame Charlie for the delay and close the channel to him.
If Bob doesn't prove the closure to Alice, Alice should blame Bob and close the channel to him.
Each node either punishes the presumed attacker or gets punished by its upstream peer.
The drawback of provable blaming is that closing channels threatens the overall network connectivity.
Channel closure also requires paying on-chain fees.


### Payment credits

Another approach is to require nodes to commit to some scarce resource in exchange for **payment credits**, which are then spent to forward payments.

One such resource is ownership of a Bitcoin unspent transaction output (UTXO), as described in the **stake certificates** proposal (an instance of a **fidelity bond**).
A routing node assigns credits to the sender based on the value of a UTXO it owns.
UTXO ownership may be proven in zero-knowledge to preserve privacy.
The routing node would only know that the sender controls _some_ UTXO from the current UTXO set that has the value in a certain range.

Many design and implementation questions remain open:

- Which ZK scheme should be used?
- What is the optimal credits-to-UTXO-value function?
- When should stake certificate expire?

Another way to assign payment credits is **proof-of-work** (PoW).
To forward a payment, the sender provides a proof of performed computation to a routing node.
The work should be specific to the payment.
Implementing a PoW-based scheme also has open questions:

- How to set the difficulty? A low difficulty wouldn't sufficiently deter jammers, while a high difficulty would deter honest users.
- Which function should be used for PoW? A SHA-256-based PoW scheme would provide a strong advantage to already deployed Bitcoin mining farms. Alternative "ASIC-resistant" hash functions have been designed to discourage hardware specialization.


## Related Research

- [Channel jamming attacks](https://bitcoinops.org/en/topics/channel-jamming-attacks/) - a Bitcoin Optech topic with links to relevant newsletter issues and other sources.
- [Spamming the Lightning Network](https://github.com/t-bast/lightning-docs/blob/master/spam-prevention.md) - an overview of jamming and upfront fee schemes.
- [Preventing Channel Jamming](https://blog.bitmex.com/preventing-channel-jamming/) - a summary and comparison of proposed countermeasures to jamming.
- An introductory blog post on [Stake Certificates](https://thelab31.xyz/blog/stake-certificates).

### Lightning-dev mailing list threads

- [PoW as anti-jamming measure](https://lists.linuxfoundation.org/pipermail/lightning-dev/2019-November/002275.html)
	- [On the equivalence of PoW vs monetary fees](https://lists.linuxfoundation.org/pipermail/lightning-dev/2019-November/002346.html)
- Discussions on upfront fees:
	- [Reverse upfront fees](https://lists.linuxfoundation.org/pipermail/lightning-dev/2020-February/002547.html)
	- [On the role of trust in fee schemes](https://lists.linuxfoundation.org/pipermail/lightning-dev/2020-October/002826.html)
	- [Bidirectional upfront fees](https://lists.linuxfoundation.org/pipermail/lightning-dev/2020-October/002862.html)
- [Nested incremental routing](https://lists.linuxfoundation.org/pipermail/lightning-dev/2020-October/002811.html)
- [Provable blaming](https://lists.linuxfoundation.org/pipermail/lightning-dev/2015-August/000135.html)


### Research papers

- Cristina Pérez-Solà et al. in *[LockDown: Balance Availability Attack against Lightning Network Channels]* focus on the capacity-based jamming attack and quantify its efficiency in terms of the ratio of the victim's locked capital to the attacker's capital.
- Rohrer et al. in *[Discharged Payment Channels: Quantifying the Lightning Network's Resilience to Topology-Based Attacks]* analyze attacks on the LN including jamming ("channel griefing"). The authors focus on the attacker's strategies, namely, which nodes or channels should be prioritized as targets, given a network graph.
- Mizrahi and Zohar in *[Congestion Attacks in Payment Channel Networks]* describe controlled slot-based jamming. The authors consider three potential goals for the attacker: blocking channels with most funds, separating as many pairs of nodes as possible, and cutting a victim node from the rest of the network. The paper also compares default parameters for the three prevalent LN implementations (LND, c-lightning, and Eclair) and estimates their respective shares in the public network. As countermeasures, the authors suggest shorter timelocks, shorter routes, peer reputation, and preventing looped payments.


## Commentary

<!-- This is where you can post relevant informal and opinionated comments from various sources on the problem. -->
<!-- Also you or anyone else can add conjecture to this section (after review). -->
<!-- In general, this is not a comments section (use the issue for that). -->

*There has been no commentary on this problem so far*


## Related Problems

[PTLC Cycle Jamming] is a special case of jamming for a new proposed type of LN channels (PTLC-based).
With PTLC-based channels, hops within one route are unlinkable, which makes jamming attacks more efficient.

[LockDown: Balance Availability Attack against Lightning Network Channels]: https://eprint.iacr.org/2019/1149
[Discharged Payment Channels: Quantifying the Lightning Network's Resilience to Topology-Based Attacks]: https://arxiv.org/abs/1904.10253
[Congestion Attacks in Payment Channel Networks]: https://arxiv.org/abs/2002.06564
[PTLC Cycle Jamming]: {% link _problems/ptlc-cycle-jamming.md %}
