---
layout: problem
title:  "PTLC Cycle Jamming"
date:   2021-02-19 14:51:50 +1100
tags: lightning
status: "open"
maintainer: LLFourn
issue: 1
---

In [PTLC] based lightning channels the lock is randomized at each hop so that two malicious nodes separated by at least one honest node along the payment path cannot use the lock to link two incoming PTLCs together[^1].
Unfortunately, this seems to make PTLCs a little bit too unlinkable -- it allows a malicious sender to create route that cycles multiple times through a target pair of honest nodes reducing the capacity between them.
The honest nodes cannot detect the attack for the same reason that it preserves privacy: each incoming PTLC cannot be linked to any previous one.
This attack is easily preventable with [HTLC] based payments where the lock is the same SHA256 image at each hop.

![cycle attack](/assets/cycle-attack.svg)
*Two attacker 😈 controlled nodes attempt to lock as much capacity from **A** to **B** as possible by having cycles in the payment path. The PTLC randomization at each honest nodes makes each incoming PTLC unlinkable from the last from the perspective of the honest nodes. The attacker triples the effectiveness of the attack against a single hop compared to an ordinary jamming attack*

Typical jamming attacks allow an attacker to lock up capacity along a path but any individual node will only have an amount locked roughly equal to that of the attacker.
This attack allows the attacker to magnify the attack a particular hop.
Note that in the above example it is not possible to just cycle back between **A** and **B** because that is easy to spot; each cycle needs three nodes in it.


## Impact

Solving this problem would allow randomized PTLCs to be included in lightning without the fear of enabling more effective jamming attacks.
It is currently not clear whether the privacy gain of randomized PTLCs would be worth allowing this attack.

## Potential Directions

### Proof of Well Formedness of Payment Path

An obvious direction is to somehow prove that the payment path does not have any cycles in it (without revealing anything else about it).
The payer could create a cryptographic a randomizable commitment to all the nodes in the path which is passed on by each node to the next (after randomizing it).
Then each encrypted payload for each node could contain a proof that it contains no cycles or, perhaps more simply, a proof that each node is only in the path once.
Note there is no need to prove that there are no cycles if there are malicious parties in the cycle since this means the malicious parties also have to lock up more funds for each cycle -- this is no more effective than a typical jamming attack.

## Proposed Solutions

*There are no proposed solutions for this problem*

## Conjecture

<a href="https://twitter.com/LLFourn"> @LLFourn</a> We are yet to see channel jamming attacks in the wild on lightning.
A reason for this might be because despite jamming attacks being easy they are also expensive.
Having to lock 0.1 BTC to remove 0.1 BTC capacity from a single hop is not very effective unless there are many hops you want to attack at once.
I suspect economic attackers would want to target very particular hops -- this attack helps them do that.

## Related Research

1. PTLCs and their application to lightning was introduced by Poelstra in [*Lightning in Scriptless Scripts*](https://lists.launchpad.net/mimblewimble/msg00086.html) on the mimbewimble mailing list.
2. The idea was later formalized by Malavolta et al. in [*Anonymous Multi-Hop Locks for Blockchain Scalability and Interoperability*](https://eprint.iacr.org/2018/472.pdf).
3. The randomized PTLC protocol for lightning is described in details in [*Multi-Hop Locks from Scriptless Scripts*](https://github.com/ElementsProject/scriptless-scripts/blob/master/md/multi-hop-locks.md)

## Related Problems

*There are currently no open problems related to this one*

## Footnotes


[^1]: This does not mean that the malicious node cannot link the payments using heuristics like amount or time-lock but that is separate issue.

[PTLC]: https://bitcoinops.org/en/topics/ptlc/
[HTLC]: https://bitcoinops.org/en/topics/htlc/
