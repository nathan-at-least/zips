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

This proposal introduces a new chain-wide fund called the "Zcash Posterity Fund" which aggregates deposits from any number of sources and produces regular small payouts from its total balance to the same recipients as new issuance as part of block rewards. These payouts are constrained by the consensus rules proposed here _and_ by establishing a new socially based precedent that the rate of payouts will not be increased in the future, thus accumulating the funds for posterity. This proposal itself does not specify how funds are deposited into the Posterity Fund, although it is accompanied by a companion proposal [FIXME REF] which defines a source of deposits as part of a new transaction fee mechanism. Unlike payouts, there is no social precedent constraining future kinds or amounts of deposits to the fund.

Motivation
==========

Smoothing out block rewards over time has both economic and security benefits:

Mitigate Outlier Block Rewards
------------------------------

If specific block rewards are much larger than others, this may perversely incentivize miner rollbacks to recapture past rewards. This risk grows as transaction fees (or other potential future reward mechanisms) grow in value relative to the steady issuance rewards, which may happen either if transaction fees grow enough, or enough time has passed for issuance rewards to diminish due to the issuance halving schedule.

Mitigate Payment-to-Self Vulnerabilities
----------------------------------------

For any consensus mechanism such as transaction fees that transfer from general users to miners, miners may pay themselves at no cost if they recoup those fees immediately. Even if those fees are recouped over a short enough time interval, there are risks for large miners or mining cartels to still benefit from "spoofing" transactions.

This prevents certain kinds of mechanism design possibilities in protocol design that may be beneficial, such as (ZIP NNN FIXME REF) which adjusts transaction fees and block size in tandem.

One solution is to destroy such fees so that miners cannot recoup their costs and no longer stand to benefit from "spoofing", which is an approach Ethereum takes in the design of EIP-1559. However, we argue that paying out those fees over a long enough time horizon has a similar practical effect based on the assumption that a mining cartel today cannot predict or control the long term future, and this also allows Zcash to remain closer to a fixed eventual supply goal.

Mining Continuity During Lulls
------------------------------

As issuance rewards decrease, if there are alternating periods of high activity followed by "lull" periods of low activity, rate limited payouts help maintain blockchain security during the lulls by savings from the high activity periods. Here we assume "activity" is anything that deposits funds into the Posterity Fund.

By contrast, if miner rewards correspond directly to activity on a short term, only miners with large savings reserves or access to credit will be able to weather the lulls. This leads makes mining less competitive and also reduces certainty about mining capacity for the whole ecosystem.

Ecosystem Benefits of Longer Time Horizons
------------------------------------------

Generally, the more reliable and long term the core functioning of the Zcash blockchain is, the more users can make long term plans that rely on Zcash safely. The more participants who do make long terms plans reinforces the planning of other potential participants, so this can lead to a virtuous adoption cycle which is sustainable, rather than due to short term trends.

Specification
=============


Consensus State
---------------

Consensus nodes are required to track new per-block state:

- `SLOW_FUND_BAL : u64 [zatoshi]`

The state is a single 64 bit integer representing the Posterity Fund balance in units of ``zatoshi``, the smallest tracked unit of ZEC. It is safe and potentially convenient to consider all blocks prior to the activation height to have a value of 0 in this field.

.. design_rationale:: 

Block Header Commitments
------------------------

Valid block headers are required to commit to the Posterity Fund balance.

.. FIXME:: Look up the arbitrary block state commitment mechanics and plug in appropriately.

Payout Mechanics
----------------

Payouts

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
