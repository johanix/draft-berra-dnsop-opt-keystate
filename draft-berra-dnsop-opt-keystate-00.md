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

This document introduces the KeyState EDNS(0) option code, to enable
the exchange of SIG(0) key state information between DNS entities via
the DNS protocol. The KeyState option allows DNS clients and servers to
include key state data in both queries and responses, facilitating mutual
awareness of SIG(0) key status between child and parent zones.
This mechanism addresses the challenges of maintaining synchronization
of SIG(0) keys used for securing DNS UPDATE messages, thereby enhancing
the efficiency and reliability of DNS operations that require coordinated
key management.

This document proposes such a mechanism.

TO BE REMOVED: This document is being collaborated on in Github at:
[https://github.com/johanix/draft-berra-dnsop-opt-transaction-state](https://github.com/johanix/draft-berra-dnsop-transaction-state-00).
The most recent working version of the document, open issues, etc, should all be
available there.  The authors (gratefully) accept pull requests.

--- middle

# Introduction



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

There is a specific use case where the proposed new KeyState EDNS(0)
will solve current problem. In that case private EDE info codes were
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

# Security Considerations
Key state signals in OPT queries and answers are unauthenticated unless the
transaction carrying the state signal is secured via mechanisms such as
{{!RFC2845}}, {{!RFC2931}}, {{!RFC8094}} or {{!RFC8484}}. Unauthenticated
information MUST NOT be trusted as the state signals influence the DNS
protocol processing. For instance, an attacker might cause a denial-of-service
by forging a response claiming that the victim's key is invalid, thereby
halting the delegation synchronization procedure.

# IANA Considerations.
## New KeyState and SetKeyState EDNS Options
This document defines a new EDNS(0) option, entitled "KeyState",
assigned a value of TBD "DNS EDNS0 Option Codes (OPT)" registry
[to be removed upon publication:
[https://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml#dns-
parameters-11]

+-------+--------------------+----------+--------------------+
| Value | Name               | Status   | Reference          |
+-------+--------------------+----------+--------------------+
| TBD   | KeyState           | Standard | [ This document ]  |
+-------+--------------------+----------+--------------------+

## A New Registry for EDNS Option KeyState State Codes
The KeyState option also defines a 16-bit state field, for which IANA is
requested to create and mainain a new registry entitled "KeyState Codes", used
by the KeyState option. Initial values for the "KeyState Codes" registry
are given below; future assignments in  in the 8-127 range are to be made
through Specification Required review [BCP26].

+-----------+---------------------------------------------+-------------------+
| KEY STATE | Mnemonic                                    | Reference         |
+-----------+---------------------------------------------+-------------------+
| 0         | UNUSED                                      | [ This document ] |
+-----------+---------------------------------------------+-------------------+
| 1         | KEY_UNKNOWN                                 | [ This document ] |
+-----------+---------------------------------------------+-------------------+
| 2         | KEY_INVALID                                 | [ This document ] |
+-----------+---------------------------------------------+-------------------+
| 3         | KEY_REFUSED                                 | [ This document ] |
+-----------+---------------------------------------------+-------------------+
| 4         | VALIDATION_FAIL                             | [ This document ] |
+-----------+---------------------------------------------+-------------------+
| 5         | AUTO_BOOTSTRAP_ONGOING                      | [ This document ] |
+-----------+---------------------------------------------+-------------------+
| 6         | MANUAL_BOOTSTRAP_REQUIRED                   | [ This document ] |
+-----------+---------------------------------------------+-------------------+
| 7         | KEY_TRUSTED                                 | [ This document ] |
+-----------+---------------------------------------------+-------------------+
| 8-127     | Unassigned                                  | [ This document ] |
+-----------+---------------------------------------------+-------------------+
| 128-65535 | Private use                                 | [ This document ] |
+-----------+---------------------------------------------+-------------------+


-------

# References

## Normative References

# Acknowledgements.

...

--- back

# Change History (to be removed before publication)

* draft-berra-dnsop-opt-transaction-state-00

> Initial public draft.
