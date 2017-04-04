<zip>
- Title: "Memo Field: Refund Address"
- Status: Draft
- Version: Draft-0
- Authors: Nathan Wilcox
- Depends: ZIP-XX Memo Field Encoding
</zip>

Overview
========

This ZIP specifies the encoding and usability prescriptions for a `refund address` tagged
memo subfield. This standard enables two way private transfers of funds initiated with an
initial Zcash shielded transfer which supports use cases such as full or partial refunds.

This depends on ZIP-XX Memo Subfield Encoding, except we anticipate an alternative where
multiple separate tagged subfields may appear in a single encrypted memo field plaintext.

Motivation
==========

In general and by design, in the Zcash protocol, a shielded transfer recipient does
not learn the sender's address. This design allows for a wide array of use cases,
such as anonymous donations. A participant of a system with strong privacy may always
elect to reveal private details of parties of their choosing, such as the address they
prefer for receiving refunds. However, there is currently no standard in the Zcash
ecosystem for sharing a preferred refund address. This proposal attempts to address
that need.

Encoding
========

The return address tagged memo subfield follows the ZIP-XX Memo Subfield Encoding specification and uses 0 as `type` designator value. The `data` byte sequence is XXX

Security and Usability Prescriptions
====================================

Terminology
-----------

`user`
  A person, organization, or entity who sends, receives, or examines Zcash
  transfers. This document discusses users with an implicit assumption that they have
  beliefs and interact with a user interface. Although the typical assumption is of
  a single human interacting with the user interface, it's important to anticipate
  entities composed of humans, automated smart contracts, various kinds of artificial
  intelligence, and other decision making apparatuses.

`wallet`
  A wallet is *any* software or service which controls Zcash funds on behalf of a `user`.

Sending Wallet User Interface
-----------------------------

Wallet interfaces **SHOULD** allow users to select on a per-transaction basis whether
to include or omit a return address. They **SHOULD** default to *including* a return
address. They **MUST** make the choice explicitly visible. They **SHOULD** provide
contextual help for this choice and explain that including the address allows the
recipient to associate the transfer with the return address.

Wallet interfaces **MUST NOT** implicitly include a refund address in any transfer
*unless* this is clearly indicated to the user.

Rationale
~~~~~~~~~

We expect it to be common for users to believe Zcash offers strong privacy without an
in-depth understanding of the relevant technical details. Therefore it is dangerous
to include a return address implicitly, since that would lead to many users mistakenly
believing their behavior has stronger privacy guarantees than it does.

However, there's a similarly compelling problem with *excluding* a refund address
implicitly by default; a relatively new user may expect a refund from a service or
vendor which is otherwise happy to oblige only for the user to discover they failed
to configure an obscure setting.

We believe the best balance is to introduce a consistent requirement on all conforming
Zcash wallets.

Recipient Logic
---------------

Recipients **MUST NOT** rely on the belief that the sender controls the refund
address. There is *no provision* in this proposal for ensuring the sender controls
the refund address. For example, an application which relied on a refund address as
an identifier of the sender would be trivially vulnerable to impersonation attacks.

Instead, all implementations must treat this subfield as indicating the *sender's
preference* of a *return address*. User interfaces should reinforce this
understanding. For example, a fund-raising application may take user deposits along
with their pledges to donate by some criterion such as how many miles race participants
run. The application will refund the portion of a deposit which remains unallocated
after the pledge obligations are fulfilled. A sender may select as their refund address
the donation address of another of their favorite charities. In doing so, they would
be practically donating any unpledged portion of their deposit to the other charity.
