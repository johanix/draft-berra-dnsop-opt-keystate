---
title: "State Signalling Via DNS EDNS(0) OPT"
abbrev: State Signalling Via DNS EDNS(0) OPT
docname: draft-berra-dnsop-transaction-state-00
date: {DATE}
category: std

ipr: trust200902
area: Internet
workgroup: DNSOP Working Group
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: E. Bergström
    name: Erik Bergström
    organization: The Swedish Internet Foundation
    country: Sweden
    email: erik.bergstrom@internetstiftelsen.se
 -
    ins: L. Fernandez
    name: Leon Fernandez
    organization: The Swedish Internet Foundation
    country: Sweden
    email: leon.fernandez@internetstiftelsen.se
 -
    ins: J. Stenstam
    name: Johan Stenstam
    organization: The Swedish Internet Foundation
    country: Sweden
    email: johan.stenstam@internetstiftelsen.se

normative:

informative:

--- abstract

This document introduces the Transaction State (TS) EDNS(0) option code
to enable the exchange of state information between DNS entities via
the DNS protocol. The TS option allows DNS clients and servers to include
transaction-specific state data in both queries and responses, overcoming
limitations of existing mechanisms like Extended DNS Errors (EDE) which
are constrained to error responses and human-readable text. By utilizing
the TS option, DNS entities can communicate operational states essential
for coordination in scenarios such as synchronization of SIG(0) key
states between child and parent server, and state synchronization between
signers in distributed multi-signer DNSSEC configurations.
This mechanism enhances the efficiency and reliability of DNS operations
requiring mutual state awareness between parties.

This document proposes such a mechanism.

TO BE REMOVED: This document is being collaborated on in Github at:
[https://github.com/johanix/draft-berra-dnsop-opt-transaction-state](https://github.com/johanix/draft-berra-dnsop-transaction-state-00).
The most recent working version of the document, open issues, etc, should all be
available there.  The authors (gratefully) accept pull requests.

--- middle

# Introduction

Yada, yada, yada.

Knowledge of DNS NOTIFY {{!RFC1996}} and DNS Dynamic Updates
{{!RFC2136}} and {{!RFC3007}} is assumed. DNS SIG(0) transaction
signatures are documented in {{!RFC2931}}. In addition this
Internet-Draft borrows heavily from the thoughts and problem statement
from the Internet-Draft on Generalized DNS Notifications (work in progress).

## Requirements Notation

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**",
"**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**",
"**NOT RECOMMENDED**", "**MAY**", and "**OPTIONAL**" in this document
are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

# Terminology 

SIG(0) 
: An assymmetric signing algorithm that allows the recipient to only 
  need to know the public key to verify a signature created by the 
  senders private key. 


# Use Cases

There are two specific use cases where the proposed new OPT code will
solve current problems. In both cases private EDE info codes were
initially used, but it was quickly realized that while EDE provides
an excellent model for the type of communication needed, EDE was too
limited in scope and another mechanism is needed.

The proposed mechanism re-uses the same format as EDE, while
explicitly removing some of the scope limitation.

## Synchronization of SIG(0) Key State Between Child and Parent
 
In draft-johani-dnsop-delegation-mgmt-via-ddns the child operator
signs DNS UPDATE messages sent to the parent receiver using a SIG(0)
private key that the parent receiver must have the public key
for. While there is a mechanism for both uploading and rolling the
public SIG(0) key there is still a risk of the child operator and the
parent receiver getting out of sync.

This will be noticed by the child if a DNS UPDATE is refused, and the
parent receiver does have the opportunity and abiulity to provide more
details of the refusal via an EDE opcode {{!RFC8914}}, but at that point
it is too late to immediately get back in sync again and as a result
the needed delegation synchronization that triggered the DNS UPDATE
will be delayed.

Using the proposed OPT TransactionState the child gains the ability to
inform the parent about its own state and in return become aware of
the parent's state independently of a new DNS UPDATE (i.e. the OPT
exchange may be sent via a normal DNS QUERY + response.

## Synchronization of State Between Signers in Distributed Multi-Signer
 
The DNSSEC multi-signer architecture {{!RFC8901}} defines a set of
processes for managing multiple DNSSEC signers, each with its own set
of DNSSEC keys. However, the signers need to communicate to exchange
DNSKEY records with each other (as each signer must sign the DNSKEY
RRset including also other signer's DNSKEYs using its KSK).

Multi-signer logic was initially managed via a monolithic, central
"controller" that queried the individual signers for data, computed
the changes needed and inserted the changes into each signer. This
model is changing to a distributed model, where there is one
controller next to each signer (i.e. under the control of the same
entity, to avoid the need for an external third party that has access
to modify contents of a zone at the signer).

However, with a distributed multi-signer model, the individual
controllers need to be able to synchronize their state as they step
through the different "processes" defined in {{!RFC8901}}. There are
two parties to each exchange and they need the ability to both express
the sender's state and, in return, receive the receivers state.

Again, this can not be achieved by using EDE.

# Comparision to Extended DNS Errors {{!RFC8914}} 

EDE (Extended DNS Errors) specify a mechanism by which a receiver of a 
DNS message gains the ability to provide more information about the 
reason for a negative response. EDE data travels in an OPT record in 
the response and consist of an EDE code and, optionally, an EDE "Extra 
Text". It is possible to return EDE data with all types of DNS 
messages, including QUERY, NOTIFY and UPDATE. 

However, there are three limitations to EDE that make it insufficent 
for communicating state between two parties: 

1. An EDE must only be present in a response, not in the originating message. 

2. An EDE must only be used to augment an error response. It should not 
   be part of a successful response. 

3. An EDE must contain an EDE info code (16 bits) and may contain "Extra 
   Text". However this extra text is intended for human consumption, 
   not automated parsing. To communicate state between two parties 
   this requirement is too strict. 

These limitations are not a problem, as EDE serves a different purpose. But it is 
clear that for use cases like the above examples, EDE is not the right 
mechanism and another mechanism is needed. Hence the present proposal. 

...

# KeyState EDNS0 Option Format 

This document uses an Extended Mechanism for DNS (EDNS0) {{!RFC6891}}
option to include Key State information in DNS messages. The option is 
structured as follows: 

                                             1   1   1   1   1   1 
     0   1   2   3   4   5   6   7   8   9   0   1   2   3   4   5 
   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
0: |                            OPTION-CODE                        |
   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
2: |                           OPTION-LENGTH                       |
   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
4: |                               KEY-ID                          |
   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
8: | SCOPE |        UNUSED         |           KEY-STATE           |
   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+ 
10:/ EXTRA-TEXT                                                    /
   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+


Field definition details: 

OPTION-CODE: 
    2 octets / 16 bits (defined in {{!RFC6891}}) contains the value NN
    for KeyState.

OPTION-LENGTH: 
    2 octets / 16 bits (defined in {{!RFC6891}}) contains the length of 
    the payload (everything after OPTION-LENGTH) in octets and should 
    be 4 plus the length of the EXTRA-TEXT field (which may be zero 
    octets long). 

KEY-ID:
    16 bits. The KeyID of the SIG(0) key that the KeyState inquiry is
	referring to. Note that while KeyIds are not guaranteed to be
    unique, it is the child that generates the initial SIG(0) key pair
    and all subsequent key pairs. Hence, it is the child's
    responsibility to not use multiple keys with the same KeyId.
	
SCOPE:
    2 bits, i.e. four possible values. In this document two of the
    values are defined:
	
	SCOPE=0: Zone inquiry. The sender is requesting information about
	the key state for the specified child zone and key from the parent
	(or registrar) UPDATE Receiver.
	
	SCOPE=1: UPDATE Receiver policy inquiry. The sender is requesting
	information about the UPDATE Receiver policy for SIG(0) child
	keys. Specifically whether automatic bootstrapping is supported or
	manual bootstrapping is required.
	
UNUSED:
    6 bits. These six bits are not specified in this document, but may
    be defined in a future document.
	
KEY-STATE:
    8 bits. Currently defined values are listed in {key-states} below.

EXTRA-TEXT:
    a variable-length sequence of octets that may hold additional 
    information. This information is intended for human consumption
	(typically a reason or additional detail), not automated
    parsing. The length of the EXTRA-TEXT MUST be derived from the
    OPTION-LENGTH field. The EXTRA-TEXT field may be zero octets in
    length.

The State option may be included in any outgoing message (QUERY, 
NOTIFY, UPDATE, etc.) and in any response (SERVFAIL, NXDOMAIN, 
REFUSED, even NOERROR, etc.) to a query that includes an OPT 
pseudo-RR {{!RFC6891}}. 

The State option may always be ignored by the recipient. However, if 
the recipient does understand the State option and responding with its 
own corresponding State option make sense, then it is expected to do so. 

This document includes a set of initial "state" codepoints but is 
extensible via the IANA registry defined and created in Section 5.2. 

# Defined and Reserved Values for SIG(0) Key States {#key-states}

This document defines a number of initial KEY-STATE codes The
mechanism is intended to be extensible and additional KEY-STATE codes
can be registered in the "KeyState Option Codes" registry
({keystate-code-registry}). The KEY-STATE code from the "KeyState" EDNS(0)
option is used to serve as an index into the "KeyState Option
Codes" registry with the initial valuses defined below.

0: Reserved. Must not be used.

1: SIG(0) key is unknown

2: SIG(0) key is invalid (eg. key data doesn't match algorithm)

3: SIG(0) key is refused (eg. algorithm not accepted)

4: SIG(0) key is known but validation has failed

5: SIG(0) key is known but not trusted, automatic bootstrapping
ongoing

6: SIG(0) key is known but not trusted, manual bootstrapping required

7: SIG(0) key is known and trusted

128-255: Reserved for private use.

To ensure that automatic delegation is correctly prepared and  
bootstrapped, the child (or an agent for the child) sends a DNS QUERY
to the parent UPDATE Reciever with QNAME="child.parent." and 
QTYPE=ANY containing an State OPT with State-Code=1 and the KeyId of
the SIG(0) key to inquire state for in the EXTRA-DATA.

The response should contain both the KeyId and the Key State in the
EXTRA-DATA, encoded as described above.


# Security Considerations.

...

# IANA Considerations.

## A New "State" EDNS Option

IANA is requested to assign a value to for the "State"
EDNS(0) Option in the "DNS EDNS0 Option Codes (OPT) registry.

# A New Registry for State Option State-Codes {#state-code-registry}

IANA is requested to create and maintain a new registry called
"State Option State-Codes" on the "Domain Name System (DNS)
Parameters" web page as follows:

Reference
: (this document)

| Range  | Purpose               | Reference       |
| ------ | ---------| --------------------- | --------------- |
| 0      | Reserved | Delegation management | (this document) |

-------

# References

## Normative References

# Acknowledgements.

...

--- back

# Change History (to be removed before publication)

* draft-berra-dnsop-opt-transaction-state-00

> Initial public draft.
