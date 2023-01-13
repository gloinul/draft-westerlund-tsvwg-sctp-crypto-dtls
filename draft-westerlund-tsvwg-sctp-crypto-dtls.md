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
   RFC4895:
   RFC5061:
   RFC6083:
   RFC8996:
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
the SCTP association with mutual authentication of the peers. The
specification is also targeting very long-lived sessions of weeks and
months and supports mutual re-authentication and forward secrecy
rekeying. This is intended as an alternative to using DTLS/SCTP (RFC
6083) and SCTP-AUTH (RFC 4895).

--- middle

# Introduction {#introduction}



## Overview

   This document describes the usage of the Datagram Transport Layer
   Security (DTLS) protocol, as defined in DTLS 1.2 {{RFC6347}}, and
   DTLS 1.3 {{RFC9147}}, as Encryption Engine in the Stream Control
   Transmission Protocol (SCTP), as defined in {{RFC9260}} with SCTP
   encryption chunk {{I-D.westerlund-tsvwg-sctp-crypto-chunk}}.  This
   pecification is intended as an alternative to DTLS/SCTP {{RFC6083}}
   and usage of SCTP-AUTH {{RFC4895}}.

   This specification provides mutual authentication of endpoints,
   data confidentiality, data origin authentication, data integrity
   protection, and data replay protection of SCTP packets. Ensuring
   these security services to the application and its upper layer
   protocol over SCTP.  Thus, it allows client/server applications to
   communicate in a way that is designed with communications
   privacy and preventing eavesdropping and detect tampering or
   message forgery.

   Applications using DTLS in SCTP can use all currently existing
   transport features provided by SCTP and its extensions. DTLS in
   SCTP supports:

   * preservation of message boundaries.

   * no limitation on number of unidirectional and bidirectional streams.

   * ordered and unordered delivery of SCTP user messages.

   * the partial reliability extension as defined in {{RFC3758}}.

   * the dynamic address reconfiguration extension as defined in
      {{RFC5061}}.

   * User messages of any size.

   * SCTP Packets with a protected set of chunks up to a size of
     2<sup>14</sup> bytes.

## Protocol Overview

   DTLS in SCTP is an encryption engine specification for the SCTP
   Encryption chunk {{I-D.westerlund-tsvwg-sctp-crypto-chunk}} that
   utilizes DTLS 1.2 or 1.3 for the security functions like
   encryption, authentication, replay protection and security
   handshakes. The basic functionalities and how things are related are
   described below.

   In a SCTP association initiation where DTLS in SCTP is chosen as
   the encryption engine for the Encryption Chunk the DTLS handshakes
   are exchanged encapsulated in the Encryption Chunk until an initial
   DTLS session has been established. If the DTLS handshake fails, the
   SCTP association is aborted. When the DTSL session has been
   established the EVALID chunk is exchanged to verify that no
   downgrade attack between different encryption engines has
   occurred. To prevent manipulation of the EVALID chunk it is
   encrypted and integrity protected as plain text SCTP chunk in an
   DTLS application data record that is then encapsulated in an
   Encryption Chunk and provided with a SCTP common header to form a
   complete SCTP packet.

   Assuming that the EVALID validation is successful the SCTP
   association is established and the ULP can start sending data over
   the SCTP association. From this point all sets of chunks intended to
   form a SCTP packet will be protected just like EVALID chunk by being the
   plaint text application data input to DTLS. When DTLS has protected
   the plaint text input, the produced encrypted DTLS application data
   record is encapsulated in the Encryption Chunk and the packet is
   transmitted.

   In the receiving SCTP stack each incoming SCTP packet on any of
   its interfaces and ports are matched to the SCTP association based
   on ports and VTAG in the common header. In that association context
   for the Encryption chunk there will exist reference to one or more
   DTLS connections used to protect the data. The DTLS connection actually
   used to protect this packet is identified by two DCI bits in the
   Encryption chunk's flags. Using the identified DTLS session the
   content of the Encryption chunk is attempted to be processed,
   including replay protection, decryption, and integrity. And if
   decryption was successful the produced plain text of one or more
   SCTP chunks are provided for normal SCTP processing in the
   identified SCTP association along with associated meta data such as
   path received on, original packet size, and ECN bits.

   When mutual re-authentication or forward secrecy rekeying is
   needed or desired by either endpoint a new DTLS connection handshake
   is performed between the SCTP endpoints. A different DTLS Connection
   ID (DCI) than currently used among the Encryption chunk flags are used to
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

   * Provides confidentiality, integrity protection and source
     authentication for each packet.

   * Provides replay protection on SCTP packet level preventing
     malicious replay attacks on SCTP, both protecting the data as well
     as the SCTP functions themselves.

   * Provides mutual authentication of the endpoints based on any
     authentication mechanism supported by DTLS.

   * Uses parallel DTLS connections to enable mutual re-authentication
     and forward secrecy rekeying. Thus, enabling SCTP association
     lifetimes without known limitations.

   * Uses core of DTLS as it is and updates and fixes to DTLS security
     properties can be implemented without further changes to this
     specification.

   * Secures all SCTP packets exchanged after SCTP association has
     reached the established state. Making targeted attacks against
     the SCTP protocol and implementation much harder.

   * DTLS in SCTP results in no limitations on user message
     transmission, those properties are the same as for an unprotected
     SCTP association.

   * Limited overhead on a per packet basis, with 4 bytes for the
     Encryption Chunk plus the DTLS record overhead. That DTLS
     overhead is dependent on the DTLS version.

   * Support of SCTP packet plain text payload sizes up to
     2<sup>14</sup> bytes.


### Benefits compared to DTLS/SCTP

   DTLS/SCTP as defined by {{I-D.ietf-tsvwg-dtls-over-sctp-bis}}
   has several important differences most to the benefit of DTLS in
   SCTP. This section reviews these differences.

   * Replay Protection in DTLS/SCTP has some limitations due to
     SCTP-AUTH {{RFC4895}} and its interaction with the SCTP implementation and
     dependencies on the actual SCTP-AUTH rekeying frequency. DTLS
     in SCTP relies on DTLS mechanism for replay protection that can
     prevent both duplicates from being delivered as well as
     preventing packets from outside the current window to be
     delivered. Thus, a stronger protection especially for non DATA
     chunk are provided and protects the SCTP stack from replayed or
     duplicated packets.

   * Encryption in DTLS/SCTP is only applied to ULP data. For
     DTLS in SCTP all chunk type after the association has reached
     established state will be encrypted. This, makes protocol attacks
     harder as a third-party attacker will have less insight into SCTP
     protocol state. Also, protocol header information likes PPIDs will
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
     message data that requires it. Also, the algorithm for when to
     discard a DTLS connection can be much simpler.

   * DTLS/SCTP rekeying can put restrictions on user message sizes
     unless the right APIs exist to the SCTP implementation to
     determine the state of user messages. No such restriction exists
     in DTLS in SCTP.

   * By using the Encryption chunk that is acting on SCTP packet level
     instead of user messages the consideration for extensions are
     quite different. Only extensions that would affect the common
     header or how packets are formed would interact with this
     mechanism, any extension that just defines new chunks or
     parameters for existing chunks is expected to just work and be
     secured by the mechanism. DTLS/SCTP do interact with
     anything that affects how user messages are handled.

   * A known downside is that the defined DTLS in SCTP usage creates a
     limitation on the maximum SCTP packet size that can be used of
     2<sup>14</sup> bytes. If the DTLS implementation does not support
     the maximum DTLS record size the maximum supported packet size
     might be even lower. However, this value needs to be compared to
     the supported MTU of IP, and are thus in reality often not an
     actual limiation. Only for some special deployments or over loop
     back may this limitation be visible.

   There are several significant differences in regard to
   implementation between the two realizations.

   * DTLS in SCTP do requires the Encryption Chunk to be implemented
     in the SCTP stack implementation, and not as an adaptation layer
     above it as DTLS/SCTP. This has some extra challenges for
     operating system level implementations. However, as some updates
     anyway will be required to support the corrected SCTP-AUTH the
     implementation burden is likely similar in this regard.

   * DTLS in SCTP can use a DTLS implementation that does not rely on
     features from outside of the core protocol, where DTLS/SCTP
     required a number of features as listed below:

        * DTLS Connection ID to identify which DTLS connection that
          should process the DTLS record.

        * Support for DTLS records of maximum size of 16 KB.

        * Optional to support negotiation of maximum DTLS record size
          unless not supporting 16 KB records when it is
          required. Even if implementing the negotiation,
          interoperability failure may occur. DTLS in SCTP will only
          require supporting DTLS record sizes that matches the
          largest IP packet size that endpoint support or the SCTP
          implementation.

        * Implementation is required to support turning off the DTLS
          replay protection.

        * Implementation is required to not use DTLS Key-update
          functionality. Where DTLS in SCTP is agnostic to its usage,
          and it provides a useful tool to ensure that the key life
          time never is an issue.

   The conclusion of these implementation details is that where DTLS
   in SCTP can use existing DTLS implementations, including OpenSSL's
   DTLS 1.2 implementation. It is not known if any DTLS stack exist
   that fully support the requirements in DTLS/SCTP. It is
   expected that a DTLS/SCTP implementation will have to also
   extended some DTLS implementation.



## Terminology

   This document uses the following terms:

   Association:
   : An SCTP association.

   Connection:
   : A DTLS connection. It is uniquely identified by a
   connection identifier.

   Stream:
   : A unidirectional stream of an SCTP association.  It is
   uniquely identified by a stream identifier.

## Abbreviations

   AEAD:
   : Authenticated Encryption with Associated Data

   DCI:
   : DTLS Connection Identifier

   DTLS:
   : Datagram Transport Layer Security

   HMAC:
   : Keyed-Hash Message Authentication Code

   MTU:
   : Maximum Transmission Unit

   PPID:
   : Payload Protocol Identifier

   SCTP:
   : Stream Control Transmission Protocol

   SCTP-AUTH:
   : Authenticated Chunks for SCTP {{RFC4895}}

   TCP:
   : Transmission Control Protocol

   TLS:
   : Transport Layer Security

   ULP:
   : Upper Layer Protocol

# DTLS identification

In this section the extension described in this document will
be specified.

## New Encryption Engines

This document specifies the adoption of DTLS as Encryprion Engine
for SCTP Crypto Chunks for DTLS1.2 and DTLS1.3

The following table applies.

~~~~~~~~~~~ aasvg
   VALUE            DTLS VERSION                             REFERENCE
  ------           -----------------                        ----------
   xxx (0xxxxx)     DTLS 1.2                                 nnnn
   xxx (0xxxxx)     DTLS 1.3                                 nnnn
~~~~~~~~~~~
{: #dtls-encryption-engines title="DTLS Encryption Engines"}

The values specified above shall be used in the parameter CRYPT
as Encryption Engines as specified in {{I-D.westerlund-tsvwg-sctp-crypto-chunk}}.

# DTLS usage of Encryption Chunk

   DTLS in SCTP uses the Encryption chunk in the following way. Fields
   not discussed are used as specified in
   {{I-D.westerlund-tsvwg-sctp-crypto-chunk}}.


~~~~~~~~~~~ aasvg
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Type = 0x0x   |   Flags   |DCI|             Length            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
\                            Payload                            /
/                                                               \
|                               +-------------------------------+
|                               |           Padding             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~~~~~~~
{: #sctp-encryption-chunk-structure title="ENCRYPT Chunk Structure"}

   DCI:
   : DTLS Connection Identifier is the lower two bits of an DTLS
   Connection Identifier counter. This is a counter implemented in
   DTLS in SCTP that is used to identify which DTLS connection
   instance that is capable of processing any received packet.

   Flags: : Chunk Flag bits not currently used by DTLS in SCTP. They
   MUST be set to zero (0) and MUST be ignored on reception. They may
   be used in future updated specifications for DTLS in SCTP.

   Payload: : One or More DTLS record. In cases more than one DTLS
   record is included all DTLS records except the last needs to
   include a length field as specified in DTLS 1.3 {{RFC9147}}.

# Crypto Chunk Integration

There are a set of requirements stated in {{I-D.westerlund-tsvwg-sctp-crypto-chunk}} that
need to be addressed in this specification, this section deals with those
requirements and how they are met in the current specification.

## State Machine

The Crypto Chunk state machine allows the Crypto Engine to have inband
or offband configuration. DTLS SHALL use inband configuration, thus
the implementation SHALL provide proper certificates to DTLS
and then let DTLS handshake the keys with the remote peer.
As soon as the SCTP State Machine enters CRYPT PENDING state, DTLS
is responsible for enabling the ENCRYPTED state. At the same time
DCI shall be initialized to the value zero.

### CRYPT PENDING state

When entering CRYPT PENDING state, DTLS will start the handshake
according to {{dtls-handshake}}.

Encryption Chunk Handler will use DCI = 0 for the initial
DTLS Connection.
Encryption Chunk Handler in this state will put DTLS records in
Crypto Chunks and deliver to the remote peer.

When a successfull handshake has been completed, DTLS will inform
Encryption Chunk Handler that will move SCTP State Machine into
ENCRYPTED state.

### ENCRYPTED state

Compliant to {{I-D.westerlund-tsvwg-sctp-crypto-chunk}}.

## DTLS Connection Handling {#dtls-connection-handling}

It's up to Encryption Chunk Handler to manage SCTP Connection and
the related DCI.

### Add a new DTLS Connection {#add-dtls-connection}

Either peers can add a new DTLS connection to the current
SCTP Association at any time, but no more
than 2 DTLS connection can be active at the same time.
The new DCI value shall be the last active DCI increased by one module 4,
this makes the attempt to create a new DTLS connection to use
the same, known, value of DCI from both peers so that DTLS
can solve the race condition.

If there are no active DTLS connections, the DCI will be set to zero.

A new handshake will be initiated by DTLS using the new DCI.
Details of the handshake are described in {{dtls-handshake}}.
When the handshake has been completed successfully, the new DTLS Connection
will be possible to use for traffic, if the handshake is not
completed successfully, the new DCI value will not be considered
and a next attempt will reuse that DCI.

### Remove an existing DTLS Connection {#remove-dtls-connection}

Either peers can remove a DTLS connection from the current SCTP Association.

When DTLS closure for a DTLS connection is completed, the related DCI is
released.

## Error cases

Any error in DTLS will be handled according to {{I-D.westerlund-tsvwg-sctp-crypto-chunk}}.

# DTLS Considerations

## Version of DTLS

   This document defines the usage of either DTLS 1.3 {{RFC9147}}, or
   DTLS 1.2 {{RFC6347}}.  Earlier versions of DTLS MUST NOT be used
   (see {{RFC8996}}).  DTLS 1.3 is RECOMMENDED for security and
   performance reasons.  It is expected that DTLS/SCTP as described in
   this document will work with future versions of DTLS.

   Only one version of DTLS MUST be used during the lifetime of an
   SCTP Association, meaning that the procedure for replacing the DTLS
   version in use requires the existing SCTP Association to be
   terminated and a new SCTP Association with the desired DTLS version
   to be instantiated.

## Configuration of DTLS

### General

   It is RECOMMENDED that the DTLS Connection ID is not included in
   the DTLS records as it is need, the Encryption Chunk indicate which
   DTLS connection this is inteded for using the the DCI bits.

   The DTLS record length field is normally not need as the Encryption
   Chunk provides a length field unless multiple records are put in
   same chunk payload.

   Sequence number size can be adapted based on how quickly it wraps.

### DTLS 1.2

Is renegotation allowed?

### DTLS 1.3

Key-Update MAY be used


# Establishing DTLS in SCTP

   This sections specifies how DTLS in SCTP is established after
   having been selected by the Encryption Chunk with DTLS as
   Encryption Engine has been negotatied in the Init and Init-ACK
   exchange per {{I-D.westerlund-tsvwg-sctp-crypto-chunk}}.

## DTLS Handshake {#dtls-handshake}

   As soon the SCTP Association has entered the SCTP state Crypt
   Pending as defined by {{I-D.westerlund-tsvwg-sctp-crypto-chunk}}
   the DTLS handshake procedure is initated by the SCTP client.

   The DTLS endpoint needs if necessary fragment the handshake into
   multiple records each meeting the known MTU limit of the path
   between SCTP endpoints. Each DTLS handshake message fragment is
   encapsulated in a Encryption Chunk. The DTLS instance SHALL use
   DTLS retransmission to repair any packet losses of handshake
   message fragment.

   Both SCTP endpoints SHALL perform authentication of the peer
   endpoint. This may require exchange or input from the ULP
   application for what peer identity that is accepted.

   If the DTLS handshake is successful to establish a security context
   to protect further communication and the peer identity is accepted
   then the SCTP assocation shall be informed that it can move to the
   Encrypted state.

   If the DTLS handshake failed the SCTP assocation SHALL be aborted
   and the appropriate error is generated.


## Validation against Downgrade Attacks

   When the SCTP association has entered the Encrypted State after the
   DTLS handshake has completed the protection against encryption
   engine negotiation downgrade is perforemd per
   {{I-D.westerlund-tsvwg-sctp-crypto-chunk}}. The EVALID chunk will
   sent inside a Encryption chunk protecting the plain text chunk
   as defined in {{chunk-processing}}.

   If the validation completes successful the SCTP association will
   enter Established State and all future SCTP packet exchanges will
   be protected by SCTP. If the validation fails the SCTP association
   will be aborted.

# Processing a Encryption Chunk {#chunk-processing}

## Sending

Encryption Chunk sending happens either when DTLS needs to send own
data directly to the DTLS peer i.e. due to handshaking or when SCTP
requires to transfer Control or Data chunk to the remote SCTP Endpoint.
For a proper handling, DCI shall be set to an established instance
of DTLS connection.

### DTLS signaling

DTLS shall transfer DTLS records to SCTP Header Handler as array of bytes
(unsigned char). Each array has maximum size equal to the maximum size
of SCTP payload as computed by SCTP minus the size of the Encryption Chunk
header.
Each array shall contain one or more DTLS records, this is up to DTLS.
From SCTP perspective each array is opaque data and will be used as
payload of one Encryption Chunk.

### SCTP signaling

SCTP Chunk handler will create the payload of a legacy SCTP packet
according to {{RFC9260}}. Such payload will assume a PMTU that is
equal to the value computed by SCTP minus the size of the Encryption
Chunk header.
It's up to SCTP Chunk Handler to implement all the SCTP rules
for bundling and take care of retransmission mechanisms.
Once ready, the payload will be transferred to DTLS as a single array of bytes
(unsigned char).

Once DTLS has created the related dtls record (or dtls records) according
to the maximum size permitted by the PMTU, it will transfer the
encrypted data as an array of bytes (unsigned char) to SCTP Header handler
for delivery.

The interface between SCTP and DTLS related to SCTP Signaling will
need to carefully evaluate the PMTU as seen by SCTP and DTLS so that
each payload generated by SCTP Chunk Handler will not result in more
than one single dtls record.

## Receiving

When receiving an SCTP packet containing an Encrypted Chunk it may
be part of the DTLS signaling or SCTP signaling. Since there's at most
one Crypto Chunk per SCTP packet, the payload of that chunk will
be transferred to the proper DTLS instance according to CID for
decryption.

### DTLS Signaling

The payload contains a dtls record that is addressed to DTLS,
i.e. handshaking, DTLS will handle it and behave according.

### SCTP Signaling

When DTLS detects a dtls record addressed to SCTP, it will decode
the data as an array of bytes and transfer it to SCTP Chunk Handler.

SCTP Chunk handler will threat the array as the payload of an SCTP
packet, thus it will exctract all the chunks and handle them according
to {{RFC9260}}

# Parallel DTLS Rekeying

Rekeyng in this specification is implemented by replacing the DTLS connection
getting old with a new one. This feature exploits the capability of parallel
DTLS connections and the possibility to add and remove DTLS connections
during the lifetime of the SCTP Association.

## Criteria for rekeying

It shall be specified rule for deciding that a DTLS connection is too old,
based on age and data consumption.

## Procedure for rekeiyng

This specification allows up to 2 DTLS connection to be active at the same
time for the current SCTP Association.
The following state machine applies.

~~~~~~~~~~~ aasvg
           +---------+
+--------->|  YOUNG  |  There's only one
|          +----+----+  DTLS connection until
|               |       aging criteria are met
|               |
|        AGING  |  REMOTE AGING
|               V
|          +---------+
|          |  AGED   |  When in AGED state a
|          +----+----+  new DTLS connection
|               |       is added with a new DCI
|      NEW DTLS |
|               V
|          +---------+
|          |   OLD   |  In OLD state there
|          +----+----+  are 2 active DTLS connections
|               |       Traffic is switched to the new one
|      SWITCH   |
|               V
|          +---------+
|          |  DEAD   |  In DEAD state the aged
|          +----+----+  connection is removed
|               |
|      REMOVED  |
+---------------+

~~~~~~~~~~~
{: #dtls-rekeying-state-diagram title="State Diagram for Rekeying"}

Trigger for rekeying can either be a local AGING event, triggered by
the DTLS connection meeting the criteria for rekeying, or a REMOTE AGING
event, triggered by receiving a DTLS record on the DCI that would be
used for new DTLS connection. In such case a new DTLS connection
shall be added according to {{add-dtls-connection}} with a new DCI.

As soon as the new DTLS connection completes handshaking, the traffic is moved
from the old one, then the procedure for closing the old DTLS connection is
initiated.

## Race condition in rekeying

A race condition may happen when both peer experience local AGING event at
the same time and start creation of a new DTLS connection.

Since the criteria for calculating a new DCI is known and specified
in {{add-dtls-connection}}, the peers will use the same DCI for
identifying the new DTLS connection. Race condition will be solved
by means of DTLS protocol.

# Security Considerations

## General


## DTLS 1.2


## DTLS 1.3


# IANA Consideration

## Registartion of DTLS as Encryption Engine

