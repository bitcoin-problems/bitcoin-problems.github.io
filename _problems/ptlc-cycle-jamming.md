---
layout: problem
title:  "PTLC Cycle Jamming"
date:   2021-02-19 14:51:50 +1100
tags: lightning crypto
status: "open"
maintainer: LLFourn
---


In [PTLC] based lightning channels the lock may be randomized at each hop by the sender so that two malicious nodes separated by at least one honest node along the payment path cannot use the lock itself to link the incoming PTLCs in their two channels.
Unfortunetly, this seems to make payments a little bit too unlinkable -- it allows a malicious sender to create route that cycles multiple times through honest nodes with the goal of reducing their channel capacity.

![cycle attack](/assets/cycle-attack.svg)
*Two attacker 😈 controlled nodes attempt to lock as much capacity from **A** to **B** as possible by having cycles in the payment path. The PTLC radomization at **C** and **D** makes each incoming PTLC unlinkable from the last from the perspective of the honest nodes. The attacker triples the effectiveness of the attack against a single hop compared to an ordinary jamming attack*

Typical jamming attacks allow an attacker to lock up capacity along a path but any individual node will only have an amount locked roughly equal to that of the attacker.
This attack allows the attacker to magnify the attack a particular hop.
Note that in the above example it is not possible to just cycle back between **A** and **B** because that is easy to spot; each cycle needs three nodes in it.

## Impact

Solving this problem would allow randomized PTLCs to be included in lightning without the fear of enabling effective jamming attacks.
It is currently not clear whether the privacy gain of randomized PTLCs would be worth allowing this attack.

## Potentinal Directions

### Proof of Well Formedness of Payment Path

It may be possible to make a proof about 

## Proposed Solutions

*There are no proposed solutions for this problem* 


## Conjecture

<a href="https://twitter.com/LLFourn"> @LLFourn</a> We are yet to see channel jamming attacks in the wild on lightning.
A reason for this might be because despite jamming attacks being easy they are also expensive.
Having to lock 0.1 BTC to remove 0.1 BTC capacity from a single hop is not very effective unless there are many hops you want to attack at once.
I suspect economic attackers would want to target very particular hops -- this attack helps them do that.

## Related Research

1. PTLCs and their application to lightning was introduced by Poesltra in [*Lightning in Scriptless Scripts*](https://lists.launchpad.net/mimblewimble/msg00086.html) on the mimbewimble mailing list.
2. The idea was later formalized by Malavolta et al. in [*Anonymous Multi-Hop Locks for Blockchain Scalability and Interoperability*](https://eprint.iacr.org/2018/472.pdf).

## Related Problems

*There are currently no open problems related to this one*

[PTLC]: https://bitcoinops.org/en/topics/ptlc/
