---
layout: problem
title:  "Post-subsidy Security"
tags: consenus, mining, incentives, PoW
status: "open"
maintainer: VzxPLnHqr
issue: 29
---

As the block subsidy drops, the network security budget may be inadequate if a robust fee market fails to develop. Two forms of the problem are overall low fees, and adequate-but-bursty fees. In both cases the network may fail to garner enough work to properly secure itself. 

## Impact

<!-- - Try not to repeat the description too much -->
<!-- - Make it clear what the impact on the big picture of Bitcoin's evolution would be -->

Ultimately a reduced security, if sustained for long enough, manifests as reduced confidence in the future of the network.
Reduced confidence could trigger a further reduction in security, and a downward spiral ensues. If the 
network cannot offer an attractive enough reward to those contributing to its security (miners), then this threat, while far
off right now (2022), could become an existential threat. 


## Potential Directions

<!-- - The main use of listing hand-wavy directions is useful to further explore the problem. -->

First we need to clearly define what a "security budget" for bitcoin is in such a way that nodes can 
measure it endogenously in an incentive compatible way (e.g. without reference/reliance on something 
outside the protocol like USD/tx). If nodes cannot measure it, it is exceedingly difficult to design
a fix.


## Proposed Solutions

Some possible (currently quite informal) definitions for security budget and associated solutions: 

1. Sztorc claims this can be solved in [1] via merged-mining and proposes a per-block security budget 
   `SB` as `SB = P*Q` where `P` is the average transaction price paid in sats/vbyte, and `Q` is the 
   number of bytes sold, the block size, in bytes/block. 
   
2. A different, but perhaps complementary, measure of network security would be to somehow spread
   the "net" accumulated work down to the per-satoshi level in the UTXO set. With this definition, 
   when a block is mined, the contributed work for that block (a simple function of the network difficulty) 
   is attributed to all sats in the current utxo set with older sats getting more than younger sats, presumably on an exponential
   scale to account for block reorg risk. When utxos are spent, their accumulated work is lost. The measure
   of network security then in this model is the total work attributed to the _current_ utxo set. Among other
   things we want this to be greater than zero.
   
 The above are just example definitions and informal possible directions. Yet, if there is to be a formally
 defined notion of a security budget for the network, then we also should be asking what the appropriate "target"
 for that budget is. We probably seek some goldilocks scenario here where the amount of security garnered by
 the network is "not too much," definitely "not too little," but "just right" (in expectation of course).


## Related Research

<!-- A very liberal list of related research. Try to include at least a  half-sentence about what it is or why it's related -->

1. Sztorc blogs about security budget here [https://www.truthcoin.info/blog/security-budget/](https://www.truthcoin.info/blog/security-budget/) and here [https://www.truthcoin.info/blog/security-budget-ii-mm/](https://www.truthcoin.info/blog/security-budget-ii-mm/)


## Commentary

<!-- This is where you can post relevant informal and opinionated comments from various sources on the problem. -->
<!-- Also you or anyone else can add conjecture to this section (after review). -->
<!-- In general, this is not a comments section (use the issue for that). -->




## Related Problems

1. Needs its own problem page - What to do in the event of a catastrophic break in sha256? Is there a soft-fork strategy available or only hard fork? 
