::

  ZIP: NNN
  Title: Zcash BASEFEE Transaction Fee Mechanism
  Owners: Nathan Wilcox <nathan+zip@electriccoin.co>
  Credits: Vitalik Buterin
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

The term "Sapling" in this document is to be interpreted as described in ZIP 205
[#zip-0205]_.

The term "Sprout value pool balance" in this document is to be interpreted as described
in ZIP 209 [#zip-0209]_.


Abstract
========

This proposal changes the transaction fee mechanism in Zcash to improve usability and privacy, simplify wallets, and to potentially address economic incentives related to hosting multiple assets on the Zcash blockchain and the future low-issuance era. The fee mechanism is altered from senders paying miners in a first price auction to senders paying a consensus-derived fee, called the `BASEFEE` into a long term issuance fund, which slowly pays out to miners.

This proposal is directly and largely inspired by Ethereum's EIP-1559 (FIXME REF) and an early Zcash proposal (FIXME REF) by Vitalik Buterin. It differs from EIP-1559 to account for Zcash differences in design and culture, particularly to accommodate the UTXO model, with consideration of privacy issus, and with issuance constraints in mind.

Motivation
==========

FIXME

Specification
=============

We first describe the "steady state" consensus rules informally, which assumes the consensus changes from this proposal are already in effect. We present activation considerations later in this document.

Constant Parameters
-------------------

The consensus rules depend on these constant protocol parameters, described here:

Block Size Cap
--------------

``BLOCK_SIZE_CAP = 2,000,000 [bytes]``
``BLOCK_SIZE_CAP = TARGET_BLOCK_SIZE * BLOCK_SIZE_ELASTICITY_FACTOR``

The size of any given block cannot be larget than ``BLOCK_SIZE_CAP`` in this proposal. The value selected matches the pre-proposal limit ensuring any reasoning about denial-of-service attacks or performance issues based on this limit remains valid. This parameter is a product of two other parameters for a (long term) target size, and the ratio between them.

Target Block Size
-----------------

``TARGET_BLOCK_SIZE = 1,000,000 [bytes]``
``TARGET_BLOCK_SIZE = BLOCK_SIZE_CAP / BLOCK_SIZE_ELASTICITY_FACTOR``

The target blocksize represents the target network transaction rate capacity over long time periods. Zcash blocks up to the date of this writing have never yet exceeded this size, so we reason that introducing this limit now will not impact current day usage, although it may impact future planning of any participants.

Because over longer time periods the average size of blocks will not grow much beyond this parameter, this proposal is effectively halving the transaction capacity of the protocol compared to the pre-proposal capacity. We consider this an acceptable drawback given the other benefits along with other scalability efforts outside of this proposal scope.

Elasticity Factor
-----------------

``BLOCK_SIZE_ELASTICITY_FACTOR = 2``

This represents the ratio of the hard Block Size Cap to the average long run capacity of the Target Block Size. A larger ratio with a fixed Block Size Cap reduces the average transaction capacity. A ratio below 2 reduces the short term transaction throughput during a spike above typical usage. We select 2 here because this is the same factor as is used by Ethereum in EIP-1559 (FIXME: investigate Filecoin) so by the time this ZIP reaches consensus there will be more real world data about this ratio.

- ``FEE_ADJUSTMENT_INTERVAL = 8 [blocks]`` 
- ``PRIORITIZATION_TIP_FACTOR = 1.2``
- ``PAYOUT_DIVISOR = 606625``

``TARGET_BLOCK_SIZE``
~~~~~~~~~~~~~~~~~~~~~

As long as blocks are below this target size, the transaction fees will continually fall.


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
