---
layout: problem
title:  "Removing cross-layer links"
tags: lightning privacy
status: open
maintainer: LLFourn
issue: 3
---

When two lightning nodes wish to inform their peers about changes in the capacity between them they broadcast a [`channel_announcement`] or [`channel_update`] gossip message.
These messages both point to an on-chain UTXO through their `short_channel_id` field along with the nodes involved (and signatures from all keys).
This creates a *cross-layer link* between the nodes in the layer-2 network and the outputs on the layer-1 blockchain.
These cross-layer links are a privacy leak.

Why was the `short_channel_id` cross-layer link introduced in the first place?
The surprising answer is that it is a convenient protection against peer-to-peer message spam.
Note carefully that routing payments only requires some estimate of the capacity between nodes not which on-chain outputs funded the intermediate hops.
The `short_channel_id` is a resource constraint on the attacker since creating valid outputs requires confirming layer-1 transactions.
Without it a attacker can consume computational and network resources of honest nodes by spamming `channel_announcement` messages (which honest nodes are obliged to rebroadcast).

In 2021, Romiti et al. published *[Cross-Layer Deanonymization Methods in the Lightning Protocol]* where they use this cross-layer link to develop heuristics to:

- Determine which node funded a lightning channel and therefore which node owns the change output.
- Assign cooperative channel close outputs to the parties involved.

Being able to associate a lightning node with on-chain outputs hurts both the privacy of the remaining on-chain funds and the user's future lightning payments.
Knowledge about the on-chain entity can be attached to a particular lightning node.
Knowledge about the lightning node, like IP address and payment patterns etc can be attached to the on-chain entity.

## Impact

Lightning is often presented as a privacy enhancing technology.
Unfortunately, cross-layer links make several common lightning usage patterns actually harmful to user privacy.
Without these links users could interact with the lightning network without taking precautions to avoid associating their on-chain funds with their lightning node.
Another potential benefit of removing these links is that lightning channels would not be as bound to the Bitcoin blockchain and may even exist on [sidechains].

## Potential Directions

### Use layer-2 funds to prevent spam

Instead of using layer-1 transactions to prevent spam, perhaps existing layer-2 funds could be used to prevent spam.
For example if a small lightning payment had to be made to process channel update messages then this may pay for the resources used by peers on the network.

## Proposed Solutions

*There are no proposed solutions for this problem*

## Related Research

1. Romiti et al. inspired this problem with *[Cross-Layer Deanonymization Methods in the Lightning Protocol]*.
2. Kappos et al. apply similar ideas against *private channels* in *[An Empirical Analysis of Privacy in the Lightning Network]*. This shows how private channels are less private if the funding coins can be associated with public channel coins.

## Commentary

<!-- This is where you can post relevant informal and opinionated comments from various sources on the problem. -->
<!-- Also you or anyone else can add conjecture to this section (after review). -->
<!-- In general, this is not a comments section (use the issue for that). -->

*There has been no commentary on this problem so far*

## Related Problems

*There is currently no related problems*

[Cross-Layer Deanonymization Methods in the Lightning Protocol]: https://arxiv.org/pdf/2007.00764.pdf
[An Empirical Analysis of Privacy in the Lightning Network]: https://arxiv.org/pdf/2003.12470.pdf
[`channel_announcement`]: https://github.com/lightningnetwork/lightning-rfc/blob/master/07-routing-gossip.md#the-channel_announcement-message
[`channel_update`]: https://github.com/lightningnetwork/lightning-rfc/blob/master/07-routing-gossip.md#the-channel_update-message
[sidechains]: https://bitcoinops.org/en/topics/sidechains/
