---
docname: draft-westerlund-tsvwg-sctp-crypto-dtls-latest
title: Datagram Transport Layer Security (DTLS) in the Stream Control Transmission Protocol (SCTP) CRYPTO Chunk
abbrev: DTLS in SCTP
obsoletes:
cat: std
ipr: trust200902
wg: TSVWG
area: Transport
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"

venue:
  group: Transport Area Working Group (tsvwg)
  mail: tsvwg@ietf.org
  github: gloinul/draft-westerlund-tsvwg-sctp-crypto-dtls

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
  RFC9113:
  RFC9325:
  I-D.ietf-tls-rfc8446bis:
  I-D.ietf-tsvwg-dtls-over-sctp-bis:
  I-D.ietf-uta-rfc6125bis:

  ANSSI-DAT-NT-003:
    target: <https://www.ssi.gouv.fr/uploads/2015/09/NT_IPsec_EN.pdf>
    title: Recommendations for securing networks with IPsec
    seriesinfo:
      ANSSI Technical Report DAT-NT-003
    author:
      -
        ins: Agence nationale de la sécurité des systèmes d'information
    date: August 2015

normative:
  RFC4820:
  RFC6347:
  RFC9147:
  RFC9260:

  I-D.westerlund-tsvwg-sctp-crypto-chunk:
    target: "https://datatracker.ietf.org"
    title: "Stream Control Transmission Protocol (SCTP) CRYPTO chunk"
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

This document defines a usage of Datagram Transport Layer Security (DTLS)
1.2 or 1.3 to protect the content of Stream Control Transmission Protocol
(SCTP) packets using the framework provided by the SCTP
CRYPTO chunk which we name DTLS in SCTP. DTLS in SCTP provides
encryption, source authentication, integrity and replay protection for
the SCTP association with mutual authentication of the peers. The
specification is also targeting very long-lived sessions of weeks and
months and supports mutual re-authentication and rekeying with ephemeral
key exchange. This is intended as an alternative to using DTLS/SCTP (RFC
6083) and SCTP-AUTH (RFC 4895).

--- middle

# Introduction {#introduction}



## Overview

   This document describes the usage of the Datagram Transport Layer
   Security (DTLS) protocol, as defined in DTLS 1.2 {{RFC6347}}, and
   DTLS 1.3 {{RFC9147}}, as protection engine in the Stream Control
   Transmission Protocol (SCTP), as defined in {{RFC9260}} with SCTP
   CRYPTO chunk {{I-D.westerlund-tsvwg-sctp-crypto-chunk}}.  This
   specification is intended as an alternative to DTLS/SCTP {{RFC6083}}
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
   transport features provided by SCTP and its extensions, in some
   cases with some limitations, as specified in
   {{I-D.westerlund-tsvwg-sctp-crypto-chunk}}. DTLS in SCTP supports:

   * preservation of message boundaries.

   * no limitation on number of unidirectional and bidirectional streams.

   * ordered and unordered delivery of SCTP user messages.

   * the partial reliability extension as defined in {{RFC3758}}.

   * multi-homing of the SCTP association per {{RFC9260}}.

   * the dynamic address reconfiguration extension as defined in
      {{RFC5061}}.

   * User messages of any size.

   * SCTP Packets with a protected set of chunks up to a size of
     2<sup>14</sup> bytes.



## Protocol Overview

   DTLS in SCTP is a protection engine specification for the SCTP
   CRYPTO chunk {{I-D.westerlund-tsvwg-sctp-crypto-chunk}} that
   utilizes DTLS 1.2 or 1.3 for the security functions like
   key exchange, authentication, encryption, integrity protection,
   and replay protection. The basic functionalities and how things
   are related are described below.

   In a SCTP association initiation where DTLS in SCTP is chosen as
   the protection engine for the CRYPTO chunk the DTLS handshake
   is exchanged encapsulated in plain DATA chunks with Protection
   Engine PPID (see section 10.6 of {{I-D.westerlund-tsvwg-sctp-crypto-chunk}})
   until an initial DTLS connection has been established.
   If the DTLS handshake fails, the
   SCTP association is aborted. When the DTLS connection has been
   established PVALID chunks are exchanged to verify that no
   downgrade attack between different protection engines has
   occurred. To prevent manipulation, the PVALID chunks are protected
   by encapsulating them in DTLS protected CRYPTO chunks.

   Assuming that the PVALID validation is successful the SCTP
   association is established and the Upper Layer Protocol (ULP) can
   start sending data over the SCTP association. From this point all
   chunks will be protected by encapsulating them in DTLS protected
   CRYPTO chunks. The SCTP chunks to be included in an SCTP packet
   are the plain text application data input to DTLS. The
   encrypted DTLS application data record is then encapsulated in the
   CRYPTO chunk and the packet is transmitted, see {{chunk-processing}}.

   In the receiving SCTP endpoint each incoming SCTP packet on any of
   its interfaces and ports are matched to the SCTP association based
   on ports and VTAG in the common header. In that association context
   for the CRYPTO chunk there will exist reference to one or more DTLS
   connections used to protect the data. The DTLS connection actually
   used to protect this packet is identified by two DCI bits in the
   CRYPTO chunk's flags. Using the identified DTLS session the content
   of the CRYPTO chunk is attempted to be processed, including replay
   protection, decryption, and integrity checking. And if decryption
   and integrity verification was successful the produced plain text
   of one or more SCTP chunks are provided for normal SCTP processing
   in the identified SCTP association along with associated meta data
   such as path received on, original packet size, and ECN bits.

   When mutual re-authentication or rekeying with ephemeral key exchange is
   needed or desired by either endpoint a new DTLS connection handshake
   is performed between the SCTP endpoints. A different DTLS Connection
   Index (DCI) than currently used among the CRYPTO chunk flags are used to
   indicate that this is a new handshake. When the handshake has
   completed the DTLS in SCTP implementation can simply switch to use
   this DTLS connection to protect the plain text payload. After a
   short while (no longer than 2 min) to enable any outstanding
   packets to drain from the network path between the endpoints the
   old DTLS connection can be terminated.

   The DTLS connection is free to send any alert, handshake message, or
   other non-application data to its peer at any point in time. Thus,
   enabling DTLS 1.3 Key Updates for example.
   All non-application data SHOULD be sent by means of SCTP DATA chunks
   with Protection Engine PPID as specified in
   {{I-D.westerlund-tsvwg-sctp-crypto-chunk}}.

~~~~~~~~~~~ aasvg
+---------------+ +--------------------+
|               | | Protection Engine  |  Keys
|      ULP      | |                    +-------------.
|               | |   Key Management   |              |
+---------------+-+---+----------------+              |
|                     |                 \    User     |
|                     |                  +-- Level    |
| SCTP Chunks Handler |                      Messages |
|                     |                               |
|                     | +-- SCTP Unprotected Payload  |
|                     |/                              |
+---------------------+    +---------------------+    |
|        CRYPTO       |    | Protection Engine   |    |
|        Chunk        |<-->|                     |<--'
|       Handler       |    | Protection Operator |
+---------------------+    +---------------------+
|                     |\
| SCTP Header Handler | +-- DTLS Encrypted SCTP Payload
|                     |
+---------------------+
~~~~~~~~~~~
{: #overview-layering title="DTLS in SCTP layer
in regard to SCTP and upper layer protocol"}


## Properties of DTLS in SCTP

   DTLS in SCTP has a number of properties that are attractive.

   * Provides confidentiality, integrity protection, and source
     authentication for each packet.

   * Provides replay protection on SCTP packet level preventing
     malicious replay attacks on SCTP, both protecting the data as well
     as the SCTP functions themselves.

   * Provides mutual authentication of the endpoints based on any
     authentication mechanism supported by DTLS.

   * Uses parallel DTLS connections to enable mutual re-authentication
     and rekeying with ephemeral key exchange. Thus, enabling SCTP association
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
     CRYPTO chunk plus the DTLS record overhead. The DTLS
     overhead is dependent on the DTLS version.

   * Support of SCTP packet plain text payload sizes up to
     2<sup>14</sup> bytes.


### Benefits Compared to DTLS/SCTP

   DTLS/SCTP as defined by {{I-D.ietf-tsvwg-dtls-over-sctp-bis}}
   has several important differences most to the benefit of DTLS in
   SCTP. This section reviews these differences.

   * Replay Protection in DTLS/SCTP has some limitations due to
     SCTP-AUTH {{RFC4895}} and its interaction with the SCTP implementation and
     dependencies on the actual SCTP-AUTH rekeying frequency. DTLS
     in SCTP relies on DTLS mechanism for replay protection that can
     prevent both duplicates from being delivered as well as
     preventing packets from outside the current window to be
     delivered. Thus, a stronger protection especially for non-DATA
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

   * By using the CRYPTO chunk that is acting on SCTP packet level
     instead of user messages the consideration for extensions are
     quite different. Only extensions that would affect the common
     header or how packets are formed would interact with this
     mechanism, any extension that just defines new chunks or
     parameters for existing chunks is expected to just work and be
     secured by the mechanism. DTLS/SCTP instead interact with
     extensions that affects how user messages are handled.

   * A known downside is that the defined DTLS in SCTP usage creates a
     limitation on the maximum SCTP packet size that can be used of
     2<sup>14</sup> bytes. If the DTLS implementation does not support
     the maximum DTLS record size the maximum supported packet size
     might be even lower. However, this value needs to be compared to
     the supported MTU of IP, and are thus in reality often not an
     actual limitation. Only for some special deployments or over
     loopback may this limitation be visible.

   There are several significant differences in regard to
   implementation between the two realizations.

   * DTLS in SCTP do requires the CRYPTO chunk to be implemented in
     the SCTP stack implementation, and not as an adaptation layer
     above the SCTP stack which DTLS/SCTP instead requires. This has
     some extra challenges for operating system level
     implementations. However, as some updates anyway will be required
     to support the corrected SCTP-AUTH the implementation burden is
     likely similar in this regard.

   * DTLS in SCTP can use a DTLS implementation that does not rely on
     features from outside of the core protocol, where DTLS/SCTP
     required a number of features as listed below:

        * DTLS Connection Index to identify which DTLS connection that
          should process the DTLS record.

        * Support for DTLS records of the maximum size of 16 KB.

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
          and it provides a useful tool to ensure that the key lifetime
          never is an issue.

   The conclusion of these implementation details is that where DTLS
   in SCTP can use existing DTLS implementations, including OpenSSL's
   DTLS 1.2 implementation. It is not known if any DTLS stack exist
   that fully support the requirements in DTLS/SCTP. It is
   expected that a DTLS/SCTP implementation will have to also
   extend some DTLS implementation.



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
   : DTLS Connection Index

   DTLS:
   : Datagram Transport Layer Security

   MTU:
   : Maximum Transmission Unit

   PPID:
   : Payload Protocol Identifier

   SCTP:
   : Stream Control Transmission Protocol

   SCTP-AUTH:
   : Authenticated Chunks for SCTP {{RFC4895}}

   ULP:
   : Upper Layer Protocol


## Conventions

{::boilerplate bcp14}

# DTLS Identification

This section identifies how the extension described in this document
is identified in the Crypto Chunk and its negotiation
{{I-D.westerlund-tsvwg-sctp-crypto-chunk}}.

## New protection Engines {protection-engines}

This document specifies the adoption of DTLS as protection engine
for SCTP Crypto Chunks for DTLS1.2 and DTLS1.3

The following table applies.

| VALUE | DTLS VERSION | REFERENCE |
| 0 | DTLS 1.2 | RFC-To-Be |
| 1 | DTLS 1.3 | RFC-To-Be |
{: #dtls-protection-engines title="DTLS protection engines" cols="r l l"}

The values specified above shall be used in the Protected Association
parameter as protection engines as specified in
{{I-D.westerlund-tsvwg-sctp-crypto-chunk}} and are registered with
IANA below in {{iana-protection-engines}}.

# DTLS Usage of CRYPTO Chunk

   DTLS in SCTP uses the CRYPTO chunk in the following way. Fields
   not discussed are used as specified in
   {{I-D.westerlund-tsvwg-sctp-crypto-chunk}}.

~~~~~~~~~~~ aasvg
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Type = 0x4x   |   Flags   |DCI|         Chunk Length          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
|                            Payload                            |
|                                                               |
|                               +-------------------------------+
|                               |           Padding             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~~~~~~~
{: #sctp-CRYPTO-chunk-structure title="CRYPTO Chunk Structure"}

   {: vspace="0"}
   DCI: 2 bits (unsigned integer)
   : DTLS Connection Index is the
   lower two bits of an DTLS Connection Index counter. This is a
   counter implemented in DTLS in SCTP that is used to identify which
   DTLS connection instance that is capable of processing any received
   packet. This counter is recommended to be 64-bit to guarantee no
   lifetime issues for the SCTP Association.

   Flags: 6 bits
   : Chunk Flag bits not currently used by DTLS in SCTP. They
   MUST be set to zero (0) and MUST be ignored on reception. They MAY
   be used in future updated specifications for DTLS in SCTP.

   Payload: variable length
   : One or more DTLS records. In cases more
   than one DTLS record is included all DTLS records except the last
   MUST include a length field. Note that this matches what is specified in
   DTLS 1.3 {{RFC9147}} and DTLS 1.2 will always include the length
   field in each record.

# Crypto Chunk Integration

There are a set of requirements stated in {{I-D.westerlund-tsvwg-sctp-crypto-chunk}} that
need to be addressed in this specification, this section deals with those
requirements and how they are met in the current specification.

## State Machine

The CRYPTO Chunk allows the protection engine to have inband or
out-of-band key establishment. DTLS in SCTP uses inband key
establishment, thus the DTLS handshake establishes shared keys with the
remote peer. As soon as the SCTP State Machine enters PROTECTION
PENDING state, DTLS is responsible for progressing to the PROTECTED
state when DTLS handshake has completed. The DCI counter is
initialized to the value zero that is used for the initial DTLS
handshake.

### PROTECTION PENDING state

When entering PROTECTION PENDING state, DTLS will start the handshake
according to {{dtls-handshake}}.

DTLS protection engine being initialized for a new SCTP association
will set the DCI counter = 0, which implies a DCI field value of 0,
for the initial DTLS connection. The DTLS handshake messages are
transmitted from this endpoint to the peer using DATA chunks with the
PPID value set to Protection Engine Protocol Identifier
{{I-D.westerlund-tsvwg-sctp-crypto-chunk}}.

When a successful handshake has been completed, DTLS protection engine
will inform CRYPTO chunk Handler that will move SCTP State Machine
into PROTECTED state.

### PROTECTED state

In the PROTECTED state the currently active DTLS connection is used
for protection operation of the payload of SCTP chunks in each packet
per below specification.  When necessary to meet requirements on
periodic re-authentication of the peer and establishment of new
forward secrecy keys a new parallel DTSL connection is established as
further specified in {{parallel-dtls}}.

### SHUTDOWN states

When the SCTP association leaves the ESTABLISHED state per {{RFC9260}}
to be shutdown the DTLS connection is kept and continues to protect
the SCTP packet payloads through the shutdown process.
Even new DTLS connections can be established if necessary to maintain
the protection of the SCTP packet. Although it is recommended to avoid
establishing new DTLS connection if not necessary to be able to
conclude the shutdown process.

When the association reaches the CLOSED state as part of the SCTP
association closing process all DTLS connections that existed are
terminated without further transmissions, i.e. DTLS close_notify is
not transmitted.


## DTLS Connection Handling {#dtls-connection-handling}

It's up to DTLS protection engine to manage the DTLS connections and
their related DCI.

### Add a New DTLS Connection {#add-dtls-connection}

Either peer can add a new DTLS connection to the SCTP association at
any time, but no more than 2 DTLS connections can exist at the same
time.  The new DCI value shall be the last active DCI increased by one
modulo 4, this makes the attempt to create a new DTLS connection to
use the same, known, value of DCI from both peers.  A new handshake
will be initiated by DTLS using the new DCI.  Details of the handshake
are described in {{dtls-handshake}}.

As either endpoint can initiate a DTLS handshake at the same time,
either endpoint may receive a DTLS ClientHello message when it has
sent its own ClientHello. In this case the ClientHello from the
endpoint that had the DTLS Client role in the establishment of the
previous DTLS connection shall be continued to be processed and the
other dropped.

When the handshake has been completed successfully, the new DTLS
connection will be possible to use for traffic, if the handshake is
not completed successfully, the new DCI value will not be considered
and a next attempt will reuse that DCI.

### Remove an existing DTLS Connection {#remove-dtls-connection}

Either peers can initialize the removal of a DTLS connection from the
current SCTP association when it is no longer the active one, i.e. when a
newer DTLS connection is in use. It is RECOMMENDED to not initiate
removal until at least one SCTP packet protected by the new DTLS
connection has been received, and any transmitted packets protected
using the new DTLS connection has been acknowledge, alternatively one
Maximum Segment Lifetime (120 seconds) has passed since the last SCTP
packet protected by the old DTLS connection was transmitted.

The closing of the DTLS connection when the SCTP association is in
PROTECTED and ESTABLISHED state is done by having the DTLS connection
send a DTLS close_notify. Note the difference in process for DTLS 1.2
and DTLS 1.3. Where sending the DTLS 1.2 close_notify will trigger an
immediate close also in the peer. Which is why it is recommended to
ensure that one have received packets from the peer using the new DTLS
connection.

When DTLS closure for a DTLS connection is completed, the related DCI is
released in the DTLS protection engine.

## Error Cases

As DTLS has its own error reporting mechanism by exchanging DTLS alert
messages no new DTLS related cause codes are defined to use the error
handling defined in {{I-D.westerlund-tsvwg-sctp-crypto-chunk}}.

When DTLS encounters an error it may report that issue using DTLS
alert message to its peer by putting the created DTLS record in a
DATA chunk with Protection Engine PPID and sending it in an SCTP packet.
This is independent of what to do in relation to the SCTP association.
Depending on the severance of the error different paths can be the result:

   Non-critical:
   : the DTLS connection can continue to protect
   the SCTP association. In this case the issue may be worth reporting
   to the peer using a DTLS alert message, but otherwise continue
   without further action.

   Critical, but recoverable:
   : In cases the DTLS connection fails fatally but it is not ensured
   that the issue will persist the protection engine SHOULD attempt to
   establish a new DTLS connection. If the DTLS handshake fails in the
   end the SCTP association will time out as the SCTP higher layer
   during this period is unable to pass any packets acknowledging a
   working path.

   Critical, non-recoverable but not immediately fatal:
   : If the
   error requires termination of the SCTP association but allows for
   sending some additional SCTP packets. Then this critical issue MUST be
   reported to the SCTP association so that it can send an ERROR chunk
   with the Error in Protection cause code without any extra cause
   code, combined with an ABORT chunk. This will terminate the SCTP
   association immediately, provide ULP with notification of the failure
   and speeding up any higher layer management of the failure.

   Critical, non-recoverable and immediately fatal:
   : If the DTLS connection fails so that no further data can be
   protected (i.e. either sent or received) with maintained security
   and establishing a new DTLS connection will not address the failure
   then the protection engine will have to indicate this to the SCTP
   implementation so it can perform a one sides SCTP association
   termination. This will lead to an eventual SCTP association timeout
   in the peer.

# DTLS Considerations

## Version of DTLS

   This document defines the usage of either DTLS 1.3 {{RFC9147}}, or
   DTLS 1.2 {{RFC6347}}.  Earlier versions of DTLS MUST NOT be used
   (see {{RFC8996}}).  DTLS 1.3 is RECOMMENDED for security and
   performance reasons.  It is expected that DTLS in SCTP as described in
   this document will work with future versions of DTLS.

   Only one version of DTLS MUST be used during the lifetime of an
   SCTP Association, meaning that the procedure for replacing the DTLS
   version in use requires the existing SCTP Association to be
   terminated and a new SCTP Association with the desired DTLS version
   to be instantiated.

## Configuration of DTLS

### General

   The DTLS Connection ID SHALL NOT be included in the DTLS records as
   it is not needed, the CRYPTO chunk indicates which DTLS connection
   the DTLS records are intended for using the DCI bits. Avoiding
   overhead and addition implementation requirements on DTLS
   implementation.

   The DTLS record length field is normally not needed as the CRYPTO
   Chunk provides a length field unless multiple records are put in
   same chunk payload. If multiple DTLS records are included in one
   CRYPTO chunk payload the DTLS record length field MUST be present
   in all but the last.

   DTLS record replay detection MUST be used.

   Sequence number size can be adapted based on how quickly it wraps.

   Many of the TLS registries have a "Recommended" column. Parameters
   not marked as "Y" are NOT RECOMMENDED to support in DTLS in
   SCTP. Non-AEAD cipher suites or cipher suites without
   confidentiality MUST NOT be supported. Cipher suites and parameters
   that do not provide ephemeral key exchange MUST NOT be supported.

### Authentication and Policy Decisions

DTLS in SCTP MUST be mutually authenticated. Authentication is the
process of establishing the identity of a user or system and verifying
that the identity is valid. DTLS only provides proof of possession of
a key. DTLS in SCTP MUST perform identity authentication. It is
RECOMMENDED that DTLS in SCTP is used with certificate-based
authentication. When certificates are used the application using DTLS
in SCTP is responsible for certificate policies, certificate chain
validation, and identity authentication (HTTPS does for example match
the hostname with a subjectAltName of type dNSName). The application
using DTLS in SCTP defines what the identity is and how it is encoded
and the client and server MUST use the same identity format. Guidance
on server certificate validation can be found in
[I-D.ietf-uta-rfc6125bis]. DTLS in SCTP enables periodic transfer of
mutual revocation information (OSCP stapling) every time a new
parallel connection is set up. All security decisions MUST be based on
the peer's authenticated identity, not on its transport layer
identity.

It is possible to authenticate DTLS endpoints based on IP addresses in
certificates. SCTP associations can use multiple IP addresses per SCTP
endpoint. Therefore, it is possible that DTLS records will be sent
from a different source IP address or to a different destination IP
address than that originally authenticated. This is not a problem
provided that no security decisions are made based on the source or
destination IP addresses.

### New Connections

Implementations MUST set up new DTLS connections before any of the
certificates expire. It is RECOMMENDED that all negotiated and
exchanged parameters are the same except for the timestamps in the
certificates. Clients and servers MUST NOT accept a change of identity
during the setup of a new connections, but MAY accept negotiation of
stronger algorithms and security parameters, which might be motivated
by new attacks.

Allowing new connections can enable denial-of-service attacks. The
endpoints MUST limit the number of simultaneous connections to two.

To force attackers to do dynamic key exfiltration and limits the
amount of compromised data due to key compromise implementations MUST
have policies for how often to set up new connections with ephemeral
key exchange such as ECDHE. Implementations SHOULD set up new
connections frequently to force attackers to dynamic key
extraction. E.g., at least every hour and every 100 GB of data which
is a common policy for IPsec [ANSSI-DAT-NT-003]. See
[I-D.ietf-tls-rfc8446bis] for a more detailed discussion on key
compromise and key exfiltration in (D)TLS.

For many DTLS in SCTP deployments the SCTP association is expected to
have a very long lifetime of months or even years. For associations
with such long lifetimes there is a need to frequently re-authenticate
both client and server by setting up new connections. TLS Certificate
lifetimes significantly shorter than a year are common which is
shorter than many expected SCTP associations protected by DTLS in
SCTP.

### Padding of DTLS Records

Both SCTP and DTLS contains mechanisms to padd SCTP payloads, and DTLS
records respectively. If padding of SCTP packets are desired to hide
actual message sizes it RECOMMEDED to use the SCTP Padding Chunck
{{RFC4820}} to generate a consisted SCTP payload size. Support of this
chunk is only required on the sender side. However, if the PAD chunk
is not supported DTLS padding MAY be used.

It needs to be noted that independent if SCTP padding or DTLS padding
is used the padding is not taken into account by the SCTP congestion
control. Extensive use of padding has potential for worsen congestion
situations as the SCTP association will consume more bandwidth than
its derived share by the congestion control.

The use of SCTP PAD chunk is recommened as it at least can enable
future extension or SCTP implementation that account also for the
padding. Use of DTLS padding hides this packet expansion from SCTP.


### DTLS 1.2

The updates in Section 13 [RFC9147] SHALL be followed for DTLS
1.2. DTLS 1.2 MUST be configured to disable options known to provide
insufficient security. HTTP/2 [RFC9113] gives good minimum
requirements based on the attacks that where publicly known in 2022.

The AEAD limits in DTLS 1.3 are equally valid for DTLS 1.2 and SHOULD
be followed for DTLS in SCTP, but are not mandated by the DTLS 1.2
specification.

Use of renegotiation is NOT RECOMMENDED as it is disables in many
implementations and does not provide any benefits in DTLS in SCTP
compared to setting up a new connection. Resumption MAY be used but
does not provide ephemeral key exchange as in DTLS 1.3

### DTLS 1.3

DTLS 1.3 is preferred over DTLS 1.2 being a newer protocol that
addresses known vulnerabilities and only defines strong algorithms
without known major weaknesses at the time of publication.

DTLS 1.3 requires rekeying before algorithm specific AEAD limits have
been reached. Implementations MAY setup a new DTLS connection instead
of using key update.

In DTLS 1.3 any number of tickets can be issued in a connection and
the tickets can be used for resumption as long as they are valid,
which is up to seven days. The nodes in a resumed connection have the
same roles (client or server) as in the connection where the ticket
was issued. Resumption can have significant latency benefits for
quickly restarting a broken DTLS/SCTP association. If tickets and
resumption are used it is enough to issue a single ticket per
connection.

The PSK key exchange mode psk_ke MUST NOT be used as it does not
provide ephemeral key exchange.

# Establishing DTLS in SCTP

   This section specifies how DTLS in SCTP is established after
   Protected Association Parameter with DTLS 1.2 or DTLS 1.3 as
   protection engine has been negotiated in the Init and Init-ACK
   exchange per {{I-D.westerlund-tsvwg-sctp-crypto-chunk}}.

## DTLS Handshake {#dtls-handshake}

   ### Handshake of initial DTLS connection

   As soon the SCTP Association has entered the SCTP state PROTECTION
   PENDING as defined by {{I-D.westerlund-tsvwg-sctp-crypto-chunk}}
   the DTLS handshake procedure is initiated by the endpoint that
   has initiated the SCTP association.

   The DTLS endpoint needs if necessary fragment the handshake into
   multiple records each meeting the known MTU limit of the path
   between SCTP endpoints. Each DTLS handshake message fragment is
   encapsulated in a DATA chunk with Protection Engine PPID.
   The DTLS instance SHALL NOT use DTLS retransmission to repair
   any packet losses of handshake message fragment.

   Both SCTP endpoints SHALL perform authentication of the peer
   endpoint. This may require exchange or input from the ULP
   application for what peer identity that is accepted.

   If the DTLS handshake is successful in establishing a security
   context to protect further communication and the peer identity is
   accepted then the SCTP association is informed that it can
   move to the PROTECTED state.

   If the DTLS handshake failed the SCTP association SHALL be aborted
   and an ERROR chunk with the Error in Protection error cause, with
   the appropriate extra error causes is generated, the right
   selection of "Error During Protection Handshake" or "Timeout During
   Protection Handshake or Validation".

  ### Handshake of further DTLS connections

   When the SCTP Association has entered the ESTABLISHED state,
   each of the endpoint can initiate an handshake.

   The DTLS endpoint needs if necessary fragment the handshake into
   multiple records each meeting the known MTU limit of the path
   between SCTP endpoints. Each DTLS handshake message fragment is
   encapsulated in a DATA chunk with Protection Engine PPID.
   The DTLS instance SHALL NOT use DTLS retransmission to repair
   any packet losses of handshake message fragment.

   Both SCTP endpoints SHALL perform authentication of the peer
   endpoint. This may require exchange or input from the ULP
   application for what peer identity that is accepted.

   If the DTLS handshake failed the SCTP association SHALL generate
   an ERROR chunk with the Error in Protection error cause, with
   extra error causes "Error During Protection Handshake".


## Validation Against Downgrade Attacks

   When the SCTP association has entered the PROTECTED state after the
   DTLS handshake has completed, the protection against downgrade in
   the negotiation of protection engine is performed per
   {{I-D.westerlund-tsvwg-sctp-crypto-chunk}}. The PVALID chunk will
   sent inside a CRYPTO chunk protecting the plain text chunk as
   defined in {{chunk-processing}}.

   If the validation completes successful the SCTP association will
   enter ESTABLISHED state. ULP data exchanges can now happen and
   will be protected together will all other SCTP packets.

# Processing a CRYPTO Chunk {#chunk-processing}

## Sending

CRYPTO chunk sending happens either when DTLS needs to send its own
data directly to the DTLS peer i.e. due to handshaking or when SCTP
requires transferring control or DATA chunk to the remote SCTP Endpoint.
For a proper handling, DCI shall be set to an established instance
of DTLS connection.

### DTLS Signaling

DTLS shall transfer DTLS records to SCTP Header Handler as array of
bytes. Each array has maximum size equal to the maximum size of SCTP
payload as computed by SCTP minus the size of the CRYPTO chunk header.
Each array shall contain one or more DTLS records, this is up to DTLS.
From SCTP perspective each array is opaque data and will be used as
payload of one CRYPTO chunk.

### SCTP Payload

SCTP Chunk handler will create the payload of a legacy SCTP packet
according to {{RFC9260}}. Such payload will assume a PMTU that is
equal to the value computed by SCTP minus the size of the CRYPTO Chunk
header and DTLS record and authentication tag overhead. It's up to
SCTP Chunk Handler to implement all the SCTP rules for bundling and
retransmission mechanism.  Once ready, the payload will be
transferred to DTLS as a single array of bytes.

Once DTLS has created the related DTLS record (or DTLS records), it
will transfer the encrypted data as an array of bytes to CRYPTO chunk
handler for encapsulation into a CRYPTO chunk and being forwarded to
the SCTP header handler for transmission.

The interface between SCTP and DTLS related to SCTP Payload will need
to carefully evaluate the PMTU as seen by SCTP and DTLS so that each
payload generated by SCTP Chunk Handler will not cause the finished
SCTP packet to exceed the known path MTU unless it is a Path MTUD
discovery packet.

## Receiving

When receiving an SCTP packet containing a CRYPTO Chunk it may
be part of the DTLS signaling or SCTP signaling. Since there's at most
one CRYPTO Chunk per SCTP packet, the payload of that chunk will
be transferred to the proper DTLS instance according to DCI for
decryption and processing.

As discussed in CRYPTO Chunk specification when receiving packets
certain meta data will be needed to associate with the protected
CRYPTO chunk payload for SCTP to correctly process it. This includes
packet size, source IP and arrival interface, i.e. path information,
ECN bits.

### DTLS Signaling

The payload contains a DTLS record that is addressed to DTLS,
e.g. handshaking, DTLS will handle it and behave according. If there
are no DTLS connection state for this DCI the DTLS will have to treat
this as incoming to a DTLS server accepting new connection.

### SCTP Signaling

When DTLS processes a DTLS record with decryption and integrity
verification and that contains application data, it will output the
data as an array of bytes and transfer it back to the CRYPTO Handler
that delivers it for SCTP chunk handling.

SCTP Chunk handler will threat the array as the payload of an SCTP
packet, thus it will extract all the chunks and handle them according
to {{RFC9260}}.

# Parallel DTLS Rekeying {#parallel-dtls}

Rekeying in this specification is implemented by replacing the DTLS connection
getting old with a new one. This feature exploits the capability of parallel
DTLS connections and the possibility to add and remove DTLS connections
during the lifetime of the SCTP Association.

## Criteria for Rekeying

The criteria for rekeying may vary depending on the ULP requirement on
security properties, chosen cipher suits etc. Therefore it is assumed
that the implementation will be configurable by the ULP to meet its demand.

Likely criteria to impact the need for rekeying through the usage of
new DTLS connection are:

   * Maximum time since last authentication of the peer

   * Amount of data transferred since last forward secrecy preserving
     rekeying

   * The cipher suit's maximum key usage being reached. Although for
     DTLS 1.3 usage of the Key Update mechanism can generate new keys
     without forward secrecy properties.


## Procedure for Rekeying

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
|          |  DRAIN  |  The aged DTLS connection
|          +----+----+  is drained before being ready
|               |       to be closed
|               |
|       DRAINED | DTLS close_notify
|               V
|          +---------+
|          |  DEAD   |  In DEAD state the aged
|          +----+----+  connection is closed
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
initiated, see {{remove-dtls-connection}}.

## Race Condition in Rekeying

A race condition may happen when both peer experience local AGING event at
the same time and start creation of a new DTLS connection.

Since the criteria for calculating a new DCI is known and specified in
{{add-dtls-connection}}, the peers will use the same DCI for
identifying the new DTLS connection. And the race condition is solved
as specified in {{add-dtls-connection}}.


# PMTU Discovery Considerations

Due to the DTLS record limitation for application data SCTP MUST use
2<sup>14</sup> as input to determine absolute maximum MTU when running
PMTUD and using DTLS in SCTP as protection engine.

The DTLS protection engine MUST provide its maximum overhead for DTLS
records and authentication tags when protecting the SCTP payload. This
so that SCTP PMTUD can take this into consideration and ensure that
produced packets that are not PMTUD probes does not become oversized.
This may require updating during the SCTP associations lifetime due to
future handshakes affecting cipher suit in use, or changes to record layer
configurations.

DTLS protection engine is RECOMMENDED to be provided with known path
MTU from SCTP so that it can operate its signaling message safely.
As the used MTU for the DTLS signaling will be DTLS responsibility.

Note that this implies that DTLS protection engine is expected to
accept application data payloads of potentially larger sizes than what
it configured to use for messages the DTLS implementation generates
itself for signaling.

# Security Considerations

## General

The security considerations given in {{RFC9147}}, {{RFC6347}}, and
{{RFC9260}} also apply to this document. BCP 195 {{RFC9325}}
{{RFC8996}} provides recommendations and requirements for improving
the security of deployed services that use DTLS. BCP 195 MUST be
followed which implies that DTLS 1.0 SHALL NOT be supported and are
therefore not defined.

## Privacy Considerations

Although DTLS in SCTP provides privacy for the actual user message as
well as almost all chunks, some fields are not confidentiality
protected.  In addition to the DTLS record header, the SCTP common
header and the CRYPTO chunk header are not confidentiality
protected. An attacker can correlate DTLS connections over the same
SCTP association using the SCTP common header.

To provide identity protection it is RECOMMENDED that DTLS in SCTP is
used with certificate-based authentication in DTLS 1.3 {{RFC9147}} and
to not reuse tickets.  DTLS 1.2 and DTLS 1.3 with external PSK
authentication does not provide identity protection.

By mandating ephemeral key exchange and cipher suites with
confidentiality DTLS in SCTP effectively mitigate many forms of
passive pervasive monitoring.  By recommending implementations to
frequently set up new DTLS connections with (EC)DHE force attackers to
do dynamic key exfiltration and limits the amount of compromised data
due to key compromise.

# IANA Consideration

This document adds the two new entries listed in
{{dtls-protection-engines}} into the "CRYPTO Chunk Protection
Engine Identifiers" registry in the Stream Control Transmission
Protocol (SCTP) Parameters grouping.

## Protection Engine Registration {#iana-protection-engines}

IANA is requested to register two Protection Engine Identifiers in the
"CRYPTO Chunk Protection Engine Identifiers" registry defined by
{{I-D.westerlund-tsvwg-sctp-crypto-chunk}}. The entries to be
registered are provided in {{iana-protection-engines-table}}.

| ID VALUE | Name | Reference | Contact |
| 0 | DTLS 1.2 | RFC-To-Be | Authors |
| 1 | DTLS 1.3 | RFC-To-Be | Authors |
{: #iana-protection-engines-table title="CRYPTO Chunk protection engines" cols="r l l l"}
