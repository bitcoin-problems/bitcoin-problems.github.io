---
layout: problem
title:  "Post-subsidy Security"
tags: consenus, mining, incentives, PoW
status: "open"
maintainer: VzxPLnHqr
issue: 29
---

As the block subsidy drops, the network security budget may be inadequate if a robust fee market fails to develop. Two forms of the problem are overall low fees, and adequate-but-bursty fees. In both cases the network may fail to garner enough work to properly secure itself.

A goal of this research is to define one or more *soft forks* which either permanently
solve or, at minimum, stave off the problem, without sacrificing current community values.
This means, for example, a hard-fork which introduces inflation will not be considered here.

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

### Defining the Security Budget

Some possible (currently quite informal) definitions for security budget and associated solutions: 

1. Sztorc claims this can be solved in [1] via merged-mining and proposes a per-block security budget 
   `SB` as `SB = P*Q` where `P` is the average transaction price paid in sats/vbyte, and `Q` is the 
   number of bytes sold, the block size, in bytes/block. 
   
2. ...
   
 The above are just example definitions and informal possible directions. Yet, if there is to be a formally
 defined notion of a security budget for the network, then we also should be asking what the appropriate "target"
 for that budget is. We probably seek some goldilocks scenario here where the amount of security garnered by
 the network is "not too much," definitely "not too little," but "just right" (in expectation of course).

### Other Ideas

There are other ideas/directions which may be currently more intuitive, but less formal:

1. Pay-it-forward-fees. A soft-fork which requires miners to pay-forward *at least
   half* of the non-subsidy fees. These forwarded fees would be earned by future miners.
   For example, the soft-fork could requie that miners include an output at a specific
   index with a locking script of `<1 year> OP_CSV`. A more complex soft-fork could
   make the duration a function of the the age of the soft-fork itself.

2. Nothing is broken. Of course there is a reasonable chance that nothing is actually
   broken and needs fixing here. For example, even in a post-subsidy world where
   blocks are mostly empty, we can imagine layer-2,3,4... protocols requiring/utilizing
   recent block headers within their own protocol. Using Lightning as a more concrete
   example: if everybody is happy with their channels, there is plenty of liquidity,
   closings are infrequent, etc, then we can easily imagine miners themselves earning
   fees within channels.


## Related Research

<!-- A very liberal list of related research. Try to include at least a  half-sentence about what it is or why it's related -->

1. Sztorc blogs about security budget here [https://www.truthcoin.info/blog/security-budget/](https://www.truthcoin.info/blog/security-budget/) and here [https://www.truthcoin.info/blog/security-budget-ii-mm/](https://www.truthcoin.info/blog/security-budget-ii-mm/)


## Commentary

<!-- This is where you can post relevant informal and opinionated comments from various sources on the problem. -->
<!-- Also you or anyone else can add conjecture to this section (after review). -->
<!-- In general, this is not a comments section (use the issue for that). -->




## Related Problems

1. Needs its own problem page - What to do in the event of a catastrophic break in sha256? Is there a soft-fork strategy available or only hard fork? 
