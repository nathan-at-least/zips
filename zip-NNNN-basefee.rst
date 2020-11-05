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

This proposal is directly and largely inspired by Ethereum's EIP-1559 (FIXME REF) and an early Zcash proposal (FIXME REF) by Vitalik Buterin. It differs from EIP-1559 due to differences in Zcash's privacy requirements and issuance constraints.

Motivation
==========

FIXME

Specification
=============

FIXME

Rationale
=========

FIXME

Security and Privacy Considerations
===================================

FIXME

Deployment
==========

FIXME

Reference Implementation
========================

FIXME

References
==========
