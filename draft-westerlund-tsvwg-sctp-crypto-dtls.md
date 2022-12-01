---
docname: draft-westerlund-tsvwg-sctp-crypto-dtls-latest
title: Datagram Transport Layer Security (DTLS) in the Stream Control Transmission Protocol (SCTP) Encryption Chunk
abbrev: DTLS in SCTP
obsoletes:
cat: std
ipr: trust200902
wg: TSVWG
area: Transport
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"


author:
-
   ins:  M. Westerlund
   name: Magnus Westerlund
   org: Ericsson
   email: magnus.westerlund@ericsson.com
-
   ins: J. Preuß Mattsson
   name: John Preuß Mattsson
   org: Ericsson
   email: john.mattsson@ericsson.com
-
   ins: C. Porfiri
   name: Claudio Porfiri
   org: Ericsson
   email: claudio.porfiri@ericsson.com

informative:
   RFC3758:
   RFC5061:
   RFC6083:
   I-D.ietf-tsvwg-dtls-over-sctp-bis:

normative:
  RFC2119:
  RFC6347:
  RFC9147:
  RFC9260:



  I-D.westerlund-tsvwg-sctp-crypto-chunk:
    target: "https://datatracker.ietf.org"
    title: "Stream Control Transmission Protocol (SCTP) Encryption Chunk"
    author:
      -
       ins:  M. Westerlund
       name: Magnus Westerlund
       org: Ericsson
       email: magnus.westerlund@ericsson.com
      -
       ins: J. Preuß Mattsson
       name: John Preuß Mattsson
       org: Ericsson
       email: john.mattsson@ericsson.com
      -
       ins: C. Porfiri
       name: Claudio Porfiri
       org: Ericsson
       email: claudio.porfiri@ericsson.com

    date: December 2022


--- abstract

This document defines a usage of DTLS 1.2 or 1.3 to protect the
content of SCTP packets using the framework provided by the SCTP
Crypto Chunk which we name DTLS in SCTP. DTLS in SCTP provides
encryption, source authentication, integrity and replay protection for
the SCTP association with mutual authetication of the peers. The
specification is also targeting very long lived sessions of weeks and
months and supports mutual re-authentication and forward secerecy
rekeying.

--- middle

# Introduction {#introduction}



## Overview

   This document describes the usage of the Datagram Transport Layer
   Security (DTLS) protocol, as defined in DTLS 1.2 {{RFC6347}}, and
   DTLS 1.3 {{RFC9147}}, over the Stream Control
   Transmission Protocol (SCTP), as defined in {{RFC9260}} with
   SCTP encryption chunk {{I-D.westerlund-tsvwg-sctp-crypto-chunk}}.

   This specification provides mutual authentication of endpoints,
   data confidentiality, data origin authentication, data integrity
   protection, and data replay protection of SCTP packets. Thus
   ensuring these security services to the SCTP using application and
   its upper layer protocol.  Thus, it allows client/server applications to
   communicate in a way that is designed to give communications
   privacy and to prevent eavesdropping and detect tampering or
   message forgery.

   Applications using DTLS in SCTP can use all currently existing
   transport features provided by SCTP and its extensions. DTLS/SCTP
   supports:

   * preservation of message boundaries.

   * a large number of unidirectional and bidirectional streams.

   * ordered and unordered delivery of SCTP user messages.

   * the partial reliability extension as defined in {{RFC3758}}.

   * the dynamic address reconfiguration extension as defined in
      {{RFC5061}}.

   * User messages of any size.

   * SCTP Packets with a protected set of chunks up to a size of
     2<sup>14</sup> bytes.

## Protocol Overview

   DTLS in SCTP is a encryption engine specification for the SCTP
   Encryption chunk {{I-D.westerlund-tsvwg-sctp-crypto-chunk}} that
   utilizes DTLS 1.2 or 1.3 for the security functions like
   encryption, authentication, replay protection and security
   handshakes. The basic functionalities and how things related are
   described below.

   In a SCTP association initiation where DTLS in SCTP is choosen as
   the encryption engine for the Encryption Chunk the DTLS handshakes
   are exchanged encapsulated in the Encryption Chunk until an intial
   DTLS session has been established. If the DTLS handshake fails the
   SCTP association is aborted. When the DTSL session has been
   established the EVALID chunk is exchanged to verify that no
   downgrade attack between different encryption engines has
   occurred. To prevent manipulation of the EVALID chunk it is
   encrypted and integrity protected as plain text SCTP chunk in an
   DTLS application data record that is then encapsulated in an
   Encryption Chunk and provided with a SCTP common header to form a
   complete SCTP packet.

   Assuming that the EVALID valdiation is successful the SCTP
   association is established and the ULP can start sending data over
   the SCTP assocation. From this point all sets of chunks intended to
   form a packet will be protected just like EVALID chunk by being the
   plaint text application data input to DTLS.

   In the receiving SCTP stack each incomming SCTP packet on any of
   its interfaces and ports are matched to the SCTP association based
   on ports and VTAG in the common header. In that association context
   for the Encryption chunk there will exist reference to one or more
   DTLS sessions used to protect the data. The DTLS session actually
   used to protect this packet is identifed by two bits in the
   Encryption chunk's flags. Using the identified DTLS session the
   content of the Encryption chunk is attempted to be processed,
   including replay protection, decryption and integrity. And if
   decryption was successful the produced plain text of one or more
   SCTP chunks are provided for normal SCTP processing in the
   identified SCTP association.

   When mutual re-authentication or forward secrecy rekeying is
   needed/desired by either endpoint a new DTLS connection handshake
   is performed between the SCTP endpoints. A differnt DTLS Connection
   ID than currently used among the Encryption chunk flags are used to
   indicate that this a new handshake. When the handshake has
   completed the DTLS in SCTP implementation can simply switch to use
   this DTLS connection to protect the plain text payload. After a
   short while (no longer than 2 min) to enable any outstanding
   packets to drain from the network path between the endpoints the
   old DTLS connection can be terminated.

   The DTLS connection is free to send any alert, handshake message or
   other non application data to its peer at any point in time. Thus,
   enabling DTLS 1.3 Key Updates usage for example.

~~~~~~~~~~~ aasvg
+---------------------+
|                     |
|        ULP          |
|                     |
+---------------------+ <- User Level Messages
|                     |
| SCTP Chunks Handler |
|                     |
+---------------------+ <- SCTP Plain Payload
|  Encryption Chunk   |  +-------------------+
|      Handler        |<-| DTLS in SCTP      |
|                     |->|                   |
|                     |  +-------------------+
+---------------------+ <- SCTP Encrypted Payload
|                     |
| SCTP Header Handler |
|                     |
+---------------------+

~~~~~~~~~~~
{: #overview-layering title="DTLS in SCTP layer
in regard to SCTP and upper layer protocol"}


## Properties of DTLS in SCTP

   DTLS in SCTP has a number of properties that are attractive.


### Benefits compared to DTLS/SCTP

   DTLS/SCTP as defined by {{I-D.ietf-tsvwg-dtls-over-sctp-bis}}
   has several important differences most to the benefit of DTLS in
   SCTP. This section reviews these differences.

   * Replay Protection in DTLS/SCTP has some limitations due to
     SCTP-AUTH and its interaction with the SCTP implementation and
     dependencies on the actual SCTP-AUTH rekeying frequency. DTLS
     in SCTP relies on DTLS mechanism for replay protection that can
     prevent both duplicates from being delivered as well as
     preventing packets from outside the current window to be
     delivered. Thus, a stronger protection especially for non DATA
     chunk are provided and protects the SCTP stack from replayed or
     duplicated packets.

   * Encryption in DTLS/SCTP is only applied to ULP data. For
     DTLS in SCTP all chunk type after the assocation has reached
     established state will be encrypted. This, makes protocol attacks
     harder as a third party attacker will have less insight into SCTP
     protocol state. Also protocol header information likes PPIDs will
     also be encrypted, which makes targeted attacks harder but also
     make management and debugging harder.

   * DTLS/SCTP Rekeying is complicated and require advanced API or
     user message tracking to determine when a key is no longer needed
     so that it can be discarded. A DTLS/SCTP key that is prematurely
     discarded can result in loss of parts of a user message and
     failure of the assumptions on the transport where the sender
     believes it delivered and the receiver never gets it. This
     usually will result in the need to terminate the SCTP association
     to restart the ULP session to avoid worse issues. DTLS in SCTP is
     robust to discarding the DTLS key after having switched to a new
     established DTLS connection. Any outstanding packets that have
     not been decoded yet will simply be treated as lost between the
     SCTP endpoints and SCTP's retransmission will retransmit any user
     message data that requires it. Also the algorithm for when to
     discard a DTLS connection can be much simpler.

   * DTLS/SCTP rekeying can put restrictions on user message sizes
     unless the right APIs exist to the SCTP implementation to
     determine the state of user messages. No such restriction exist
     in DTLS in SCTP.

   There are several significant difference in regards to
   implementation between the two realizations.

   * DTLS in SCTP do requires the Encryption Chunk to be implemented
     in the SCTP stack implementation, and not as an adaptation layer
     above it as DTLS/SCTP. This have some extra challenges for
     operating system level implementations. However, as some updates
     anyway will be required to support the corrected SCTP-AUTH the
     implementation burden is likely similar in this regard.

   * DTLS in SCTP can use a DTLS implementation that does not rely on
     features from outside of the core protocol, where DTLS/SCTP
     required a number of features as listed below:

        * DTLS Connection ID to identify which DTLS connection that
          should process the DTLS record.

        * Support for DTLS records of maximum size of 16 KB.

        * Optinal to support negotiation of maximum DTLS record size
          unless not supporting 16 KB records when it is
          required. Even if implementing the negotation,
          interoperability failure may occurr. DTLS in SCTP will only
          require to support DTLS record sizes that matches the
          largest SCTP packets that the SCTP stack can support.

        * Implementation is required to support turning of the DTLS
          replay protection.

        * Implementation is required to not use DTLS Key-update
          functionality. Where DTLS in SCTP is agnostic to its usage,
          and it provides a useful tools to ensure that the key life
          time never is an issue.

   The conclusion of these implementation details is that where DTLS
   in SCTP can use existing DTLS implementations, include OpenSSL's
   DTLS 1.2 implementation it is not known if any DTLS stack exist
   that fully support the requirements in DTLS/SCTP. It is
   expected that a DTLS/SCTP implemetnation will have to also
   extended a DTLS implementation.



# DTLS usage of Encryption Chunk

