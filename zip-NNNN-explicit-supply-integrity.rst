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

Each `supply value` can be calculated solely inductively given the `supply values` of the parent block along with the current block's state. We define the values in their calculation specifications below, although consensus nodes may use a different equivalent calculation method.

We anticipate most implementations will use the inductive definitions and cache these values with blocks. This approach should introduce very little runtime overhead: ``K * 8`` bytes of space and ``???`` additions per block. However, this approach may introduce a risk of caching or persistence bugs.

Machine Representation
----------------------

Every value tracked by consensus rules may take any integer value between 0 and ``MAX_MONEY`` inclusive. All consensus nodes MUST ensure any possible value is representable in their runtime.

Consensus nodes SHOULD prevent runtime value representations that are not possible supply values where possible. For example, no supply variable can ever be validly negative, or greater than ``MAX_MONEY`` so consensus nodes MAY use unsigned types, refinement types, or other techniques to prevent runtime errors.

Arithmetic Operations
---------------------

The only operations necessary on supply values are addition, equality testing, and serialization for cryptographic hashing. Addition and equality testing must match common unsigned 64 bit semantics available on most machines and programming languages today.

Note that addition of any two supply values MUST be guaranteed by consensus nodes to never overflow. Consensus nodes can guarantee this by ensuring every supply value is known to be in the valid value range (thus ``≤ MAX_MONEY``) prior to addition, and then immediatley after addition verify that the sum is also in the same valid range. Consensus nodes must not add more than two candidate values between these range checks to completely mitigate the possibility of overflow bugs or vulnerabilities.

These values must be calculated as part of block validation, so an overflow failure MUST lead to a block rejection similar to any other block-scoped consensus violation.

Calculated Values
-----------------

The calculated values map to a subset of the terms defined in `Terminology`_ with identical names. Wherever a subscript ``…[n]`` appears the value is specific to a specific block at height ``n``. The block at height ``n-1`` is the direct parent of the block at height ``n``.

.. code::

   ISSUANCE[n] = FIXME # is already defined in `FIXME: REFERENCE`

   TOTAL_BLOCK_REWARD[n] = ISSUANCE[n] + FEES[n]

   UNCLAIMED_REWARDS[n] = TOTAL_BLOCK_REWARD[n] - CLAIMED_REWARDS[n]

   ISSUED_SUPPLY[n] = ISSUED_SUPPLY[n-1] + ISSUANCE[n]

   KNOWN_INACCESSIBLE_SUPPLY[n] = KNOWN_INACCESSIBLE_SUPPLY[n-1] + UNCLAIMED_REWARDS[n]




Block Header Commitments
------------------------

Constant Parameters
-------------------

Rationale
=========

Security and Privacy Considerations
===================================

Deployment
==========

Reference Implementation
========================

References
==========
