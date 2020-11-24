::

  ZIP: NNNN
  Title: Explicit Supply Integrity
  Owners: Nathan Wilcox <nathan+zip@electriccoin.co>
  Status: ???
  Category: Consensus
  Created: 2020-11-05
  License: MIT


Terminology
===========

The key words `MUST`, `SHOULD`, and `OPTIONAL` in this document are to be interpreted
as described in RFC 2119. [#RFC2119]_

The term `network upgrade` in this document is to be interpreted as described in ZIP 200
[#zip-0200]_.

All of the following terms with the exception of `eventual supply` have unique specific values associated with each block. Wherever a specific block is not specified in context, the best known current chain tip is the implied block.

The term `issuance` refers to the creation of new units of ``ZEC``, increasing the total current ``ZEC`` supply.

The term `issued supply` refers to the sum of ``ZEC`` that has been issued up to a particular block. Where a specific block is not explicitly defined in context, it is assumed to be the current best known chain tip.

The term `eventual supply` refers to the issued ``ZEC`` supply after issuance reaches 0 ``ZEC`` per block, which by consensus rules and social consensus shall never be greater than ``MAX_MONEY`` `FIXME REF LINK`.

The term `accessible supply` refers to all ``ZEC`` for which anyone retains the capability of transferring that ``ZEC``. This value is unknown for at least two reasons: keys may be lost or funds may be transferred to irretrievable addresses. The accessible supply is bounded above by the `known maximal accessible supply`.

The term `inaccessible supply` is the complement to accessible supply, and the value is generally unknown for the same reasons described there. However, the value is partially known by a lower bound called the `known inaccessible supply`.

The term `known inaccessible supply` is ``ZEC`` which can be determined *solely by consensus rules* to be permanently inaccessible. The only known case of this occurs when block reward recipients do not claim the fully available rewards. The consensus rule constraint in this definition rules out subjective cases such as claims of lost keys, as well as objective cases that are too cumbersome to compute for consensus nodes, such as tranfers to proven irretrievable addresses *unless* those transfer mechanisms are explicitly defined in future protocol changes.

The term `maximal accessible supply` is the known upper bound of the `accessible supply`, and complementary to the `known inaccessible supply`.

The term `shielded pool supply` or simply `shielded pool` refers to the balance of one of the shielded pools, either the Sprout shielded pool, the Sapling shielded pool, or any newly introduced shielded pools. These are explicitly tracked consensus values of a particular block.

The term `transparent pool supply` or simply `transparent pool` refers to the sum of all ZEC in all UTXOs of a particular block.

The term `supply invariant` is an equation which must hold for each block to adhere to consensus rules.

Abstract
========

This proposal defines explicit terms for all portions of the ``ZEC`` supply and introduces an explicit `supply invariant` equation. It then introduces a protocol change requiring nodes to explicitly calculate and commit to the terms of the `supply invariant` equation.

Motivation
==========

The Zcash consensus rules have always safeguarded invariants about the supply of ZEC which is one of the most fundamental properties underpinning the value of ``ZEC`` and Zcash at large. While it is quite feasible in practice to tally blockchain data and verify that supply invariants hold, we see in practice that there are enough nuances and edge cases that independent attempts arrive at different sums. By requiring all consensus nodes to explicitly tally key high level terms and commit to them, we can improve accuracy and clarity of supply calculations, and in turn this can increase confidence in current or potential users and usage.

Additionally, if there are any undiscovered current or future bugs in maintaining supply integrity, an additional check and consensus-level commitment to block headers may help prevent certain kinds of implementation bugs from causing chain forks or supply integrity failures.

Specification
=============

In addition to the `Terminology`_ defined above, this proposal introduces a consensus changes requiring each block to commit to a caculation of these values, as well as a requirement that consensus nodes make these values available to their users.

Consensus State
---------------

Of the terms defined above in `terminology` the following terms have explicitly associated consensus state called `supply constants` or `supply values`. All of the named `supply values` are calculated from specific block context (including all of that blocks history), so each block (with historical context) maps deterministically to a set of specific `supply values`.

Each `supply value` can be calculated solely inductively given the `supply values` of the parent block along with the current block's state. We define the values using this inductive method below, although consensus nodes may use a different equivalent calculation method. A subset of `supply values` can be calculated solely from a single block's content.

We anticipate most implementations will use the inductive definitions and cache these values with blocks. This approach should introduce very little runtime overhead: ``K * 8`` bytes of space and ``???`` additions per block. However, this approach may introduce a risk of caching or persistence bugs.

Machine Representation
----------------------

All values can take a subset of integer values, representing a number of `zatoshi` units of account.

All supply values are either supply `balance` values or supply `delta` values. Supply balance values may take any value between 0 and ``MAX_MONEY`` inclusive, whereas supply delta values may take any value between value between ``- MAX_MONEY`` and ``MAX_MONEY`` inclusive.

.. FIXME:: Pick a better name than `balance`. What word am I reaching for here that complements delta and is a "natural number measurement" (ie: non-negative)?

In this specification, identifiers with a ``D_`` prefix have delta types and may take on negative values. All other identifiers refer to balance values and must be non-negative.

All consensus nodes MUST ensure any possible value is representable in their runtime, and must distinguish between the potentially negative delta types versus all other types.

Consensus nodes SHOULD prevent runtime value representations that are not possible supply values where possible. For example, no supply value, except for signed deltas, can ever be validly negative. Another exmaple is that no value can be greater than ``MAX_MONEY``.

Consensus nodes SHOULD use unsigned types for non-signed deltas, refinement types, or other techniques to prevent runtime errors. Consensus nodes should use proper checked arithmetic operations and MUST prevent overflow for any supply representation, as described next.

Arithmetic Operations
---------------------

The only operations necessary on supply values are addition, equality testing, and serialization for cryptographic hashing. Addition and equality testing must match common unsigned 64 bit semantics available on most machines and programming languages today.

Note that addition of any two supply values MUST be guaranteed by consensus nodes to never overflow. Consensus nodes can guarantee this by ensuring every supply value is known to be in the valid value range (thus ``≤ MAX_MONEY`` and for signed delta values ``≥ - MAX_MONEY``) prior to addition, and then immediatley after addition verify that the sum is also in the same valid range. Consensus nodes must not add more than two candidate values between these range checks to completely mitigate the possibility of overflow bugs or vulnerabilities.

These values must be calculated as part of block validation, so an overflow failure MUST lead to a block rejection similar to any other block-scoped consensus violation.

Potential Counterfeit Detection
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An important case of overflow is a negative intermediate result of a shielded pool supply value. This may be due to a live counterfeit exploit that is otherwise undetectable.

Therefore, implementations MUST not only reject candidate blocks, but also they MUST persistently store the invalid block (ideally in the network transmitted unserialized form), peer connection info, and other diagnostic information the implementors deem relevant for this case. Implementations MUST ensure this diagnostic information _excludes_ any sensitive secrets _except_ the peer IP address or other peer identification information. It explicitly MUST exclude the any cryptographic secrets the node has access to, as well as any wallet-specific information, even including addresses or other so-called "public" data.

Additionally, they MUST alert the user to the potential severity of this issue and encourage those users to share the underflow diagnostic information publicly.

.. TODO:: Introduce a ZIP for a unified RPC interface for user-facing alerts that handles both end-user apps as well as hosted production environment use cases (ie: exchanges, mobile wallet providers, etc…). This alert mechanism should use that.

.. TODO:: Introduce a ZIP for transmitting the underflow diagnostic specimen or a relevant subset of it through the p2p layer. (This may be difficult due to lack of authenticated attestation and/or Sybil resistance at that layer.

Finally, it's important to note this case may occur to to other bugs that do not stem from a counterfeit vulnerability, so designers, implementors, and users should be aware of the potential for false positives.

Calculated Values
-----------------

The calculated values map to a subset of the terms defined in `Terminology`_ with identical names. Wherever a subscript ``…[n]`` appears the value is specific to a specific block at height ``n``. The block at height ``n-1`` is the direct parent of the block at height ``n``.

Top-Level Aggregates
~~~~~~~~~~~~~~~~~~~~

We first define constants:

.. code::

   ZATOSHI_PER_ZEC = 100_000_000

   MAX_MONEY = 21_000_000 * ZATOSHI_PER_ZEC

   EVENTUAL_SUPPLY = FIXME-calculate-this. Is it different from BTC?

   assert( EVENTUAL_SUPPLY <= MAX_MONEY )

Next we define values that are derived solely from a single block's contents:

.. code::

   ISSUANCE[n] = FIXME # is already defined in `FIXME: REFERENCE`

   TOTAL_BLOCK_REWARD[n] = ISSUANCE[n] + FEES[n]

   UNCLAIMED_REWARDS[n] = TOTAL_BLOCK_REWARD[n] - CLAIMED_REWARDS[n]

Finally we define accumulating values which can be computed solely from the previous category and the values of the preceeding block, and which are defined as 0 for height 0:

.. code::

   ISSUED_SUPPLY[0] = 0

   ISSUED_SUPPLY[n] = ISSUED_SUPPLY[n-1] + ISSUANCE[n]

   KNOWN_INACCESSIBLE_SUPPLY[0] = 0

   KNOWN_INACCESSIBLE_SUPPLY[n] = KNOWN_INACCESSIBLE_SUPPLY[n-1] + UNCLAIMED_REWARDS[n]

Pool Aggregates
~~~~~~~~~~~~~~~

To account for any number of current or future shielded pools, we generalize with the `_{pool}` subscript notation and assume `pool` is defined uniquely for each pool.

This document is written while only two pools exist, which can be uniquely labeled as `sprout` and `sapling`. The general formulae for all pools are:

.. code::

   D_SHIELDED_POOL_DELTA_{pool}[0] = 0

   D_SHIELDED_POOL_DELTA_{pool}[n] = <pool-specific-calculation>

   SHIELDED_POOL_SUPPLY_{pool}[n] = SHIELDED_POOL_SUPPLY_{pool}[n-1] + D_SHIELDED_POOL_DELTA_{pool}[n]

Please see the `Potential Counterfeit Detection`_ section specifically about the transitive supply sum calculation for `SHIELDED_POOL_SUPPLY_{pool}[n]`.

Pool-Specific Calculations
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. TODO:: Fill out this section.

.. TODO:: Define a _potential_ standard for all future pools to use the Sapling-style delta? Is that too constraining on future designs?

Block Header Commitments
------------------------

.. TODO:: Fill out this section.

Value Reporting
---------------

.. TODO:: Define a standard format for reporting these values to users. Maybe a JSON blob? Then claim all implementations SHOULD be able to be queried for this over their programmatic UI.

Rationale
=========

.. rationale:: Ensure there is 0 misunderstanding or disagreement about these aggregate values so everyone sees the same data.

.. rationale:: belt-and-suspenders for _some_ kinds of supply tracking bugs.

.. rationale:: Let's make it really for any tools like block explorers to report the same info with the same methodology.

Security and Privacy Considerations
===================================

Deployment
==========

Reference Implementation
========================

References
==========
