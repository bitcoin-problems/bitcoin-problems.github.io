---
layout: problem
title:  "PTLC Cycle Jamming"
tags: lightning
status: "open"
maintainer: LLFourn
issue: 1
---

In *Point Time Locked Contract* ([PTLC]) based [Lightning] channels the lock is randomized at each hop so that two malicious nodes separated by at least one honest node in a payment path cannot use the lock to determine whether the PTLCs are part of the same payment.
Unfortunately, this seems to make PTLCs a little bit too unlinkable -- it allows a malicious sender to create a single payment that cycles multiple times through a target pair of honest nodes significantly reducing the capacity between them for a small cost to the attacker.
As in an ordinary jamming attack the malicious receiver then refuses to unlock the payment leaving the funds locked along the path until timeout.
The honest nodes cannot detect the attack for the same reason that it preserves privacy: each incoming PTLC cannot be linked to any previous one.

{% include figure.html image="cycle-attack.svg" name="PTLC cycle jamming attack" caption="
Two attacker 😈 controlled nodes drain capacity from **A** to **B** by having cycles in the payment path. The PTLC randomization at each honest node makes each incoming PTLC unlinkable from the last from the perspective of the honest nodes. The attacker quadruples the effectiveness of an ordinary jamming attack against a single hop.
"
%}

Typical jamming attacks allow an attacker to lock up capacity along a path but any individual node will only have an amount locked roughly equal to that of the attacker.
This enhanced attack allows the attacker to magnify the attack a particular hop (as shown in the figure above).
Note that it is not possible to just cycle back between **A** and **B** because that is easy to spot; each cycle needs at least three honest nodes in it.

This attack is easily preventable with [HTLC] based payments where the lock is the same SHA256 image at each hop.
Despite this, most implementations did not enforce this (see the work of Pérez-Solà et al. in [Related Research](#related-research)).

## Impact

Solving this problem would allow randomized PTLCs to be included in lightning without the fear of enabling more effective jamming attacks.

## Potential Directions

### Prove payment path is cycle-free

An obvious direction is to somehow prove that the payment path does not have any cycles in it (without revealing anything else about it).
For example, the payer could create a cryptographic randomizable commitment to all the nodes in the path which is passed on by each node to the next (after randomizing it).
Then each encrypted payload for each node could contain a proof that it contains no cycles.

## Proposed Solutions

*There are no proposed solutions for this problem*

## Commentary


> <a href="https://twitter.com/LLFourn"> @LLFourn</a>
> Having to lock 0.1 BTC to remove 0.1 BTC capacity from a single hop is not very effective unless there are many hops you want to attack at once.
> I suspect economic attackers would want to target particular hops or particular nodes. This attack helps them do that.

## Related Research

1. PTLCs and their application to lightning was introduced by Poelstra in [*Lightning in Scriptless Scripts*](https://lists.launchpad.net/mimblewimble/msg00086.html) on the mimbewimble mailing list.
2. The idea was later formalized by Malavolta et al. in [*Anonymous Multi-Hop Locks for Blockchain Scalability and Interoperability*](https://eprint.iacr.org/2018/472.pdf).
3. A detailed description of the randomized PTLC protocol is described in [*Multi-Hop Locks from Scriptless Scripts*](https://github.com/ElementsProject/scriptless-scripts/blob/master/md/multi-hop-locks.md).
4. Pérez-Solà et al. analyze the effectiveness of cycle jamming attacks in against the existing HTLC based network in *[LockDown: Balance Availability Attack against Lightning Network Channels]*.
   They focus on the collateral the attacker needs to commit to either block incoming or outgoing payments.
   They show that it is practical to enhance the naive attack by 3-10 times using cycles in the payment path against existing implementations.
   Their main recommendation is to deny cycles in the payment path.
   It is not clear whether the strength of the attacks comes from using cycles between two nodes (which can easily be prevented with PTLCs) or cycles between 3 or more (which can't).

## Related Problems

*There are currently no open problems related to this one*

[PTLC]: https://bitcoinops.org/en/topics/ptlc/
[HTLC]: https://bitcoinops.org/en/topics/htlc/
[Lightning]: https://en.wikipedia.org/wiki/Lightning_Network
[LockDown: Balance Availability Attack against Lightning Network Channels]: https://eprint.iacr.org/2019/1149.pdf
