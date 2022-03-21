---
layout: problem
title:  "Channel jamming"
tags: lightning denial-of-service
status: "open"
maintainer: s-tikhomirov
issue: 30
---

Jamming is a denial-of-service attack on Lightning channels.
Normally, Lightning payments occur in two stages.
First, liquidity is locked up in hash time-locked contracts (HTLCs) along a payment route.
Second, the receiver either claims or fails the payment, resolving the HTLCs.
The attacker exploits this mechanism by deliberately initiating spam payments (jams).
Jams put excessive burden on routing nodes and temporarily prevent them from forwarding other payments.,
Jamming decreases the profitability of routing nodes and the reliability of the network as a whole.
Preventing jamming is difficult due to the anonymity-preserving features of LN protocol.

Jamming attacks may be classified into _capacity-based_ vs _slot-based_.
In capacity based jamming, the value of jams is proportional to the capacity of victim channels.
In slot-based jamming, low-value jams are used to deplete the limit of _payment slots_ in victim channels.
A channel can only have up to 483 concurrent in-flight payments.

Furthermore, jamming attacks can be _controlled_ and _uncontrolled_.
In controlled jamming, the attacker controls both the sender and the receiver of jams.
The receiver can keep a jam in-flight for a long time (up until the payment timeout).
In uncontrolled jamming, the attacker sends unsolicited payments to arbitrarily chosen receivers.
An honest receiver would fail the payments quickly.
However, the resources of routing nodes (capacity and a payment slot) would be occupied for a few seconds.
The attacker can send uncontrolled jams often enough to replicate the effect of one long-held controlled jam.

Jamming is essentially free as senders only pay if a payment succeeds.
The attacker only bears the cost of temporarily locking some capital (which is minimal for slot-based jamming).
Moreover, Lightning payments are onion-routed to preserve users' privacy.
Therefore, routing nodes cannot easily rate-limit or ban misbehaving nodes as the don't know where jams come from.


## Impact

<!-- - Try not to repeat the description too much -->
<!-- - Make it clear what the impact on the big picture of Bitcoin's evolution would be -->

Effective countermeasures against jamming are necessary for widespread Lightning adoption.
In its current state, Lightning is constantly threatened by cheap denial-of-service attacks.
If this danger persists, professional routing nodes would be incentivized to implement proprietary anti-jamming measures, which would likely go against Lightning's emphasis on privacy.


## Potential Directions

<!-- - The main use of listing hand-wavy directions is useful to further explore the problem. -->

There are two major approaches to potential anti-jamming countermeasures:
1. Require that nodes control or spend some scarce resource to initiate a payment (examples include upfront fees, proof-of-work, and fidelity bonds);
2. Allow nodes to send payments in proportion to some reputation units.

The former approach maintains anonymity but must address game-theoretical and UX challenges.
In particular, users may be hesitant to pay fees for failed payment attempts.
The latter approach may be more straightforward but may also harm users' privacy.
A practical solution could combine the two approaches.


## Proposed Solutions

### Node configuration

Configuring routing nodes to be more restrictive in payment forwarding is a straightforward countermeasure that doesn't require changes in the Lightning protocol.
Configuration changes may include:

1. Decrease HTLC timeouts. The attacker would have to send more jams per unit time to keep a channel blocked (for controlled jamming). The drawback is that the node would have a shorter time window to withdraw their in-flight HTLCs after channel closure.
2. Increase minimal amount for forwarded payments. The attacker would need more capital (especially relevant for slot-based jamming). The routing node, on the other hand, would lose potential revenue from honest low-value payments.
3. Divide payment slots into groups for different amount buckets. A routing node would forward small payments (which may be jams) through a subset of slots, but aim to always keep some slots open for larger payments (which may also be jams). This countermeasure increases the attack cost for controlled slot-based jamming.
4. Limit the number of in-flight payments from less trustworthy peers. This may hinder attacks coming from an immediate peer but may also affect honest peers unknowingly forwarding jams.

Configuration changes may discourage simplistic attacks but don't solve the issue fundamentally.


### Upfront fees

Upfront fees are fees paid to initiate a payment.
In contrast, fees in the current LN protocol are only paid if the payment succeeds.
Upfront fees may be refundable or non-refundable.
Non-refundable upfront fees impose a cost on failed payment attempts.

Consider a payment along a route: Alice - Bob - Charlie - Dave.
In the simplest **forward upfront fee** scheme, each node pays a fee to offer an HTLC to the next node (Alice to Bob, Bob to Charlie, Charlie to Dave).
The fee amounts should increase: otherwise, in controlled jamming, the attacker who controls Alice and Dave would pay zero fee.
The issue with forward upfront fees is incentive compatibility.
Intermediary nodes have an incentive to deliberately fail a payment and keep the upfront fee.

In the **reverse upfront fee** scheme, nodes pay for _receiving_ HTLCs rather than offering them (Bob to Alice, Charlie to Bob, Dave to Charlie).
Reverse fees may be justified as follows: Bob pays Alice for the opportunity to earn routing fees that she is offering to him.
Reverse upfront fees should be zero during some grace period (that is, no fees are paid if a payment resolves quickly).
Otherwise, an uncontrolled jamming attack would force nodes to pay fees with no cost for the attacker.

The **bidirectional upfront fee** scheme combines forward and reverse upfront fees.
Forward fees are non-refundable, whereas reverse fees are only refundable within the grace period.

**Nested incremental routing** discourages routing nodes from failing payments to keep the upfront fee.
As per this proposal, the sender builds the route iteratively.
To send a payment to Dave, Alice first communicates to Bob and offers him an upfront fee to forward a payment to Charlie.
Bob then offers an upfront fee to Charlie.
If Charlie cannot forward the payment, Bob can ask his other direct peer Carol, and so on.
At any point, the maximum value that a routing node can steal is one-hop worth of upfront fees.
This distinguishes this scheme from the current protocol where routing nodes are offered upfront fees proportional to length of the remaining rouge, and therefore have a stronger incentive to misbehave.
The drawback of nested incremental routing is high communication cost: multiple round trips would increase payment latency and facilitate timing-based privacy attack.

The key challenge for upfront fee schemes is UX.
Users may not be prepared to pay for failed payment attempts.
This concern may be less pressing if failed attempts are rare.
Professional routing nodes may also choose to voluntarily refund upfront fees for their users if a payment fails.

Another open question is whether fee amounts should be based on how long the HTLC is held.
Fundamentally, this should be the case, as in-flight payments consume liquidity (time-value of locked-up capital).
Moreover, it's unclear what the trusted time source should be.
Timestamps in Bitcoin blocks can be considered trustless.
However, more fine-grained time signal is needed for LN payments.


### Reputation

There have been multiple proposals based on node reputation.

One proposal is to assign reputation to direct peers (**direct peer reputation**).
Bob would decrease the reputation of his peer Alice if a payment she offered did not resolve quickly.
The fundamental drawback of this approach is that it may unduly decrease reputation of honest routing nodes.
Due to onion routing, Bob cannot know if Alice sent the payment or forwarded it.
The attacker, in turn, can deny responsibility by claiming that jams originated at another node.
Punishing direct peers for payments they forward may also motivate routing nodes to require identification from all direct peers.
This would endanger the permissionless nature of Lightning.

The **provable blaming** scheme aims to properly assign blame for stuck payments.
The node that delays payment resolution should have its channel closed by the prior peer on the route.
Consider a route: Alice - Bob - Charlie - Dave.
Suppose that Alice and Dave are cooperating to jam the channel between Bob and Charlie.
Alice initiates the payment, and Dave doesn't reveal the payment secret.
After a grace period, Charlie closes the channel to Dave.
If Charlie doesn't prove the closure to Bob, Bob blames Charlie and closes the channel to him.
If Bob doesn't do that, Alice blames Bob and closes the channel to him.
Thus, each node either cooperates in punishing the attacker or gets punished itself.
The drawback of provable blaming is that closing a channel may be too harsh a punishment.
First, it requires paying on-chain fees.
Second, if widely applied, it threatens the overall connectivity of the network.


### Payment tokens / credits

A similar approach may be as follows: each node is assigned some number of **reputation tokens** (or **payment credits**) that are consumed when payments are made.
The key question is who and how assigns such tokens.

One way to do that could be **fidelity bonds**.
Nodes are supposed to commit some scarce resource (bond) and receive the right to send LN payments in exchange.
An example of such resource is the ownership of an unspent transaction output (UTXO), as described in the **stake certificates** proposal.
As per this proposal, senders provide a proof that they control a UTXO.
A routing node assigns some amount of credits to the sender based on the value of the UTXO in question.
Zero-knowledge proofs may be used to preserve sender's privacy.
Ideally, the routing node only knows that the sender controls _some_ UTXO that belongs to the current UTXO set with the value in a certain interval (for example, between 0.5 and 1 BTC).

The expected benefit of stake certificates as compared to upfront fees is that it doesn't inflate the cost of honest payments.
In other words, honest users only have to commit so the scarce resource (UTXO ownership) that they control anyway, as opposed to paying extra fees with sats.

However, many design and implementation questions remain.
For instance, it's unclear which ZK-scheme to use; what the payment-credits-to-UTXO-value function should be; for how long stake certificate should be issues, and so on.

Payment tokens or credits may function similarly to stake certificates.
The key difference is initial distribution: it's unclear who should initially assign reputation tokens and based on which criteria.
Another open question is the consequences of a secondary market for reputation tokens.


### PoW

Proof-of-work, which was initially proposed as an anti-spam measure, could in principle be used against jamming.
A routing node would require the sender to present a PoW for a particular payment.
This scheme may be simpler than upfront fees discussed above, although there are several potential issues.

1. It's unclear how difficulty should be defined. A low difficulty won't deter jamming, while a high difficulty would burden honest users too much. Moreover, PoW-based anti-spam protection in Lightning may give rise to a market for PoW solutions, with dedicated computational facilities  computing PoW on request, potentially facilitating spam.
2. Bitcoin itself is already bootstrapped from PoW. Therefore, one may argue, introducing PoW at another layer of the Bitcoin stack is redundant. Theoretically, the same logic should be implementable with existing PoW-based tokens (sats). On the other hand, fees paid in sats may not be game-theoretically equivalent to PoW. Sats are valuable outside of a particular LN payment, while PoW solutions are not.
3. PoW may give "unfair" advantage to operators of specialized hardware (analogous to SHA-256 ASIC in Bitcoin mining). This effect may be weaker depending on the chosen PoW function. For instance, memory-hard hash functions are more "ASIC-resistant" than general-purpose hash functions like SHA-256. On the other hand, the economic argument suggests that ASICs for any computation will be developed if they are profitable.


## Related Research

- [Channel jamming attacks](https://bitcoinops.org/en/topics/channel-jamming-attacks/) - a Bitcoin Optech topic with links to relevant newsletter issues and other sources.
- [Spamming the Lightning Network](https://github.com/t-bast/lightning-docs/blob/master/spam-prevention.md) - an overview of jamming and upfront fee schemes.
- [Preventing Channel Jamming](https://blog.bitmex.com/preventing-channel-jamming/) - a summary and comparison of proposed countermeasures to jamming.
- [Stake Certificates](https://thelab31.xyz/blog/stake-certificates)

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

- Cristina Pérez-Solà et al. in *[LockDown: Balance Availability Attack against Lightning Network Channels]* focus on the capacity-based jamming attack and quantify its efficiency in terms of the ratio of the blocked capital to attacker's capital.
- Rohrer et al. in *[Discharged Payment Channels: Quantifying the Lightning Network's Resilience to Topology-Based Attacks]* analyze attacks on the LN including jamming ("channel griefing"). The authors focus mainly on the attacker's strategies, namely, which nodes or channels should be attacked with a given graph structure. 
- Mizrahi and Zohar in *[Congestion Attacks in Payment Channel Networks]* describe controlled slot-based jamming. The authors consider three potential goals for the attacker: blocking channels with most funds; separating as many pairs of nodes as possible; cutting a chosen victim node from the network. The paper also compares default parameters for the three prevalent LN implementations (LND, c-lightning, and Eclair) and estimates their respective shares in the public graph. As countermeasures, the authors suggest shorter HTLC timelocks, shorter routes, lower maximum concurrent HTLCs based on peer reputation, and preventing looped payments.


## Commentary

<!-- This is where you can post relevant informal and opinionated comments from various sources on the problem. -->
<!-- Also you or anyone else can add conjecture to this section (after review). -->
<!-- In general, this is not a comments section (use the issue for that). -->

*There has been no commentary on this problem so far*


## Related Problems

[PTLC Cycle Jamming] is a special case of jamming for a proposed new type of LN channels. Its strengthened privacy properties (unlinkability hops within one payment) may make jamming attacks more efficient.


[PTLC Cycle Jamming]: {% link _problems/ptlc-cycle-jamming.md %}
[Congestion Attacks in Payment Channel Networks]: https://arxiv.org/abs/2002.06564
[LockDown: Balance Availability Attack against Lightning Network Channels]: https://eprint.iacr.org/2019/1149
[Discharged Payment Channels: Quantifying the Lightning Network's Resilience to Topology-Based Attacks]: https://arxiv.org/abs/1904.10253
