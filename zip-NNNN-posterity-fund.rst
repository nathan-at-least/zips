::

  ZIP: NNNN
  Title: The Posterity Fund
  Owners: Nathan Wilcox <nathan+zip@electriccoin.co>
  Status: ???
  Category: Consensus
  Created: 2020-11-05
  License: MIT


Terminology
===========

The key words "MUST", "SHOULD", and "OPTIONAL" in this document are to be interpreted
as described in RFC 2119. [#RFC2119]_

The term "network upgrade" in this document is to be interpreted as described in ZIP 200
[#zip-0200]_.

The term "issuance" refers to the creation of new units of ZEC, increasing the total current ZEC supply.

The term "issued ZEC supply" refers to the sum of ZEC that has been issued up to a particular block.

The term "current ZEC supply" or simply "supply" refers to the issued ZEC supply for the current best chain tip block.

The term "eventual ZEC supply" refers to the issued ZEC supply after issuance reaches 0 ZEC per block, which by consensus rules and social consensus shall never be greater than ``MAX_MONEY`` (FIXME LINK).

The term "Posterity Fund" refers to a new chain-wide fund specified in this proposal.

The term "fund" refers to any distinct ZEC balance which (is by definition) governed by consensus rules.

The term "Posterity Fund payout" or simply "payout" refers to transfers out of the Posterity Fund.

The term "Posterity Fund deposit" refers to transfers into the Posterity Fund.


Abstract
========

This proposal introduces a new chain-wide fund called the "Zcash Posterity Fund" which aggregates deposits from any number of sources and produces regular small payouts from its total balance to the same recipients as new issuance as part of block rewards. These payouts are constrained by the consensus rules proposed here _and_ by establishing a new policy precedent that the rate of payouts will not be increased in the future, thus accumulating the funds for posterity.

The Posterity Fund payouts become part of the per-block reward. Importantly, the payout recipients and proportion among those recipients is defined by this proposal to match that of newly issued ZEC distributions.

This proposal itself does not specify how funds are deposited into the Posterity Fund, although it is accompanied by a companion proposal [FIXME REF] which defines a source of deposits as part of a new transaction fee mechanism. Unlike payouts, there is no social precedent constraining future kinds or amounts of deposits to the fund.

This proposal not only introduces a consensus change, but also establishes a set of policies constraining future consensus changes:

#. The rate of payouts can never be increased by future changes, although it may be decreased by a legitimate consensus change process.
#. No ZEC may be removed from the fund except through the payout mechanism with the sole exception of a bug fix required to maintain the fundamental supply integrity through a legitimate consensus change process.
#. The set of recipients and proportion among them must be the same for newly issued ZEC and Posterity Fund payouts. Thus any change to the recipients or their proprtions must apply identically to both the Posterity Fund and newly issued ZEC.

Motivation
==========

The Zcash network operation and development is sustained by block rewards, a fundamental feature for its success. Block rewards are algorithmically guaranteed to follow a known schedule into the future due to the fundamental issuance policy Zcash inherits from Bitcoin. This clarity about the rate and future schedule of block rewards is a foundational component of the value proposition of Zcash, because it lets all participants understand and plan around the network's sustainability model. However, this algorithmic control and clarity about the future comes at the cost of newly issued ZEC which dilutes current holders to sustain the network.

The Zcash Posterity Fund supplements block rewards from newly issued ZEC with a payout mechanism designed to give similar clarity and algorithmic control benefits, but which uses other sources of funds instead of dilution via newly issued ZEC. This proposal does not itself specify a source of funds and can support any number and sources of funds. It is introduced with a companion proposal [FIXME REF] which provides an initial source of funds from transaction fee mechanics.

As the issuance rate of Zcash drops, there are known concerns about the sustainability of the network design shared by Bitcoin-like designs. By instituting a mechanism to augment and eventually replace that source of block rewards now, Zcash will be well positioned to maintain long-term sustainability. Because payouts from the Posterity Fund are rate limited by design, establishing deposits into this fund now will allow it to build up reserves prior to upcoming issuance reward halvings.

Ecosystem Benefits of Longer Time Horizons
------------------------------------------

Generally, the more reliable and long term the core functioning of the Zcash blockchain is, the more users can make long term plans that rely on Zcash safely. The more participants who do make long terms plans reinforces the planning of other potential participants, so this can lead to a virtuous adoption cycle which is sustainable, rather than due to short term trends.

Consensus Continuity During Lulls
---------------------------------

As issuance rewards decrease towards zero, if there are alternating periods of high activity followed by "lull" periods of low activity, rate limited payouts help maintain blockchain security during the lulls by savings from the high activity periods. Here we assume "activity" is anything that deposits funds into the Posterity Fund.

By contrast, if miner rewards correspond directly to activity on a short term, only miners with large savings reserves or access to credit will be able to weather the lulls. This leads makes mining less competitive and also reduces certainty about mining capacity for the whole ecosystem.

Mitigate Outlier Block Rewards
------------------------------

If specific block rewards are much larger than others, this may perversely incentivize miner rollbacks to recapture past rewards. This risk grows as transaction fees (or other potential future reward mechanisms) grow in value relative to the new supply issuance rewards (which tend towards zero). This may happen either if transaction fees grow enough, or enough time has passed for issuance rewards to diminish due to the issuance halving schedule.

Mitigate Payment-to-Self Vulnerabilities
----------------------------------------

For any consensus mechanism such as transaction fees that transfer from general users to miners, miners may pay themselves at no cost if they recoup those fees immediately. Even if those fees are recouped over a short enough time interval, there are risks for large miners or mining cartels to still benefit from "spoofing" transactions.

This prevents certain kinds of mechanism design possibilities in protocol design that may be beneficial, such as (ZIP NNN FIXME REF) which adjusts transaction fees and block size in tandem.

One solution is to destroy such fees so that miners cannot recoup their costs and no longer stand to benefit from "spoofing", which is an approach Ethereum takes in the design of EIP-1559. However, we argue that paying out those fees over a long enough time horizon has a similar practical effect based on the assumption that a mining cartel today cannot predict or control the long term future, and this also allows Zcash to remain closer to a fixed eventual supply goal. The Posterity Fund offers an alternative to *any* protocol, application, or use case that benefits from destroying or burning funds by, in-effect, directing those funds instead to future maintenance of the block chain.

Specification
=============

The Posterity Fund is a single global balance with an exponential rate of payouts. This provides the economic and security support described in the motivation section, while also importantly keeping the fund payouts extremely simple to describe and implement.

Consensus State
---------------

Consensus nodes are required to track new per-block state:

- `POSTERITY_FUND_BAL[H] : u64 [zatoshi]`

The state is a single 64 bit integer at any given block height, ``H``, representing the Posterity Fund balance at that height, ``H``. This state represents units of ``zatoshi``, the smallest tracked unit of ZEC. It is safe and potentially convenient to consider all blocks prior to the activation height to have a value of 0 in this field.

.. annotation:: Design Rationale

    A single integer per block is sufficient state to represent accumulation in the form of deposits, and payouts which are constrained to a simple exponential decay formula. The cost and complexity overhead for fully validing nodes is very modest.

Block Header Commitments
------------------------

Valid block headers are required to commit to the Posterity Fund balance.

.. FIXME:: Look up the arbitrary block state commitment mechanics and plug in appropriately.

.. annotation:: Design Rationale

    Requiring block-header-rooted commitments of global fund balances such as the Posterity Fund ensures that any consensus deviating bugs in accounting of this balance are immediately detected in the earliest impacted block. This is a "belt-and-suspenders" safety measure.

Payout Mechanics
----------------

Payouts occur each block and by the definition of this proposal, they are always distributed to the same recipients, in the same proportion, as newly issued ZEC defined in those rewards. When the rate of newly issued ZEC reaches zero, the recipients and proportion of distribution of these Posterity Fund remains as it was for the block with the last non-zero ZEC issuance reward.

Any algorithmic or codified changes to the recipients or proportions 

 As of NU4, newly issued ZEC in block rewards are distributed proportionally with 80% to the block miner, then the 20% remainder distributed as part of the ZIP-1014 Dev Fund to the Bootstrap Foundation, the Zcash Foundation, and the Zcash Open Major Grants organization.

.. annotation:: Design Rationale

    This ZIP is explicitly agnostic as to the recipients of block rewards so that acceptance or adoption of the Posterity Fund does not introduce or bundle reallocation decisions with the primary proposal.

Constant Parameters
-------------------

The consensus rules depend on these constant protocol parameters, described here:

- ``PAYOUT_DIVISOR = 606625``

Rationale
=========

FIXME

Security and Privacy Considerations
===================================

FIXME

TODO: Calculate the cost per time of filling blocks to the limit starting with 0 BASEFEE.

TODO: Review DOS benchmarks w/ zcashd and zebrad.

Deployment
==========

FIXME

Reference Implementation
========================

FIXME

References
==========
