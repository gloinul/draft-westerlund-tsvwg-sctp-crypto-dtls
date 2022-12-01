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

normative:
  RFC2119:
  RFC6347:
  RFC9147:
  RFC9260:


  SCTP-CRYPT-CHUNK:
    target: "https://T.B.D/draft.pdf"
    title: "Stream Control Transmission Protocol (SCTP) Encryption Chunk"
    seriesinfo: "IETF Draft"
    author:
      -
         ins: M. Westerlund
         name: Magnus Westerlund
      -
         ins: J. Preuß Mattsson
         name: John Preuß Mattsson
      -
         ins: C. Porfiri
         name: Claudio Porfiri
    date: January 2023

--- abstract

This document describes a method for adding DTLS support to the
Stream Control Transmission Protocol (SCTP) Encryption Chunk.
SCTP Encryption Chunk is intended to enable communications privacy
for applications that use SCTP as their transport protocol and
allows applications to communicate in a way that is designed to
prevent eavesdropping and detect tampering or message forgery.

The DTLS support defined here complements the Encryption Chunk
solution with respect of DTLS being used as Encryption Engine.
In this respect, this specification defines how
the content of the crypto chunk is encrypted, authenticated and
protected against replay, as well as how keymanagement is accomplised.

Applications using DTLS in SCTP can use all transport
features provided by SCTP and its extensions.

Based on {{SCTP-CRYPT-CHUNK}}

--- middle

# Introduction {#introduction}

## Overview

   This document describes the usage of the Datagram Transport Layer
   Security (DTLS) protocol, as defined in DTLS 1.2 {{RFC6347}}, and
   DTLS 1.3 {{RFC9147}}, as Encryption Engine in the Stream Control
   Transmission Protocol (SCTP), as defined in {{RFC9260}} with
   SCTP encryption chunk {{SCTP-CRYPT-CHUNK}}.



