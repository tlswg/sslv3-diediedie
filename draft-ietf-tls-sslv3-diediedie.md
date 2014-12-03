---
title: Deprecating Secure Sockets Layer Version 3.0
abbrev: SSLv3 is not Secure
docname: draft-ietf-tls-sslv3-diediedie-latest
date: 2014
category: std
ipr: trust200902
updates: 5246

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: R. Barnes
    name: Richard Barnes
    org: Mozilla
    email: rlb@ipv.sx
 -
    ins: M. Thomson
    name: Martin Thomson
    org: Mozilla
    email: martin.thomson@gmail.com
 -
    ins: A. Pironti
    name: Alfredo Pironti
    org: INRIA
    email: alfredo@pironti.eu
 -
   ins: A. Langley
   name: Adam Langley
   org: Google
   email: agl@google.com


normative:
  RFC6101:
  I-D.ietf-tls-prohibiting-rc4:
  RFC2119:
  RFC2246:
  RFC4346:
  RFC4492:
  RFC5246:
  RFC6101:
  RFC7366:

informative:
  RFC1321:
  RFC3174:
  RFC5077:
  RFC6176:
  RFC6347:
  RFC7301:
  FIPS180-2:
    title: NIST FIPS 180-2, Secure Hash Standard
    author:
      name: NIST
      ins: National Institute of Standards and Technology, U.S. Department of Commerce
    date: 2002-08
  POODLE:
    target: http://googleonlinesecurity.blogspot.com/2014/10/this-poodle-bites-exploiting-ssl-30.html
    title: "This POODLE bites: exploiting the SSL 3.0 fallback"
    author:
      name: Bodo Moeller
      ins: B. Moeller
    date: 2014-10-14
  Ray09:
    title: "Authentication Gap in TLS Renegotiation"
    author:
      name: Marsh Ray
      ins: M. Ray
    date: 2009
  TRIPLE-HS:
    title: "Triple Handshakes and Cookie Cutters: Breaking and Fixing Authentication over TLS"
    author:
      -
        name: Karthikeyan Bhargavan
        ins: K. Bhargavan
        org: INRIA
      -
        name: Antoine Delignat-Lavaud
        ins: A. Delignat-Lavaud
        org: INRIA
      -
        name: Cédric Fournet
        ins: C. Fournet
        org: Microsoft Research
      -
        name: Alfredo Pironti
        ins: A. Pironti
        org: INRIA
      -
        name: Pierre-Yves Strub
        ins: P-Y. Strub
        org: IMDEA
    seriesinfo:
      IEEE: "Symposium on Security and Privacy"
    date: 2014


--- abstract

Secure Sockets Layer version 3.0 (SSLv3) {{RFC6101}} is no longer secure.  This
document requires that SSLv3 not be used.  The replacement versions, in
particular Transport Layer Security (TLS) 1.2 {{RFC5246}}, are considerably more
secure and capable protocols.

This document updates the backward compatibility sections of the TLS RFCs to
prohibit fallback to SSLv3.

--- middle

# Introduction

The SSLv3 protocol has been subject to a long series of attacks, both on its key
exchange mechanism and on the encryption schemes it supports since it was
released in 1996.  Despite being replaced by TLS 1.0 {{RFC2246}} in 1999, and
subsequently TLS 1.1 in 2002 {{RFC4346}} and 1.2 in 2006 {{RFC5246}},
availability of these replacement versions has not been universal.  As a result,
many implementations of TLS have permitted the negotiation of SSLv3.

The predecessor of SSLv3, SSL version 2, is no longer considered secure
{{RFC6176}}.  SSLv3 now follows.

# Do Not Use SSL Version 3.0

SSLv3 MUST NOT be used {{RFC2119}}.  Negotiation of SSLv3 from any version of TLS
MUST NOT be permitted.

Any version of TLS is more secure than SSLv3, though the highest version
available is preferable.

Pragmatically, clients MUST NOT send a ClientHello with
ClientHello.client_version set to {03,00}.  Similarly, servers MUST NOT send a
ServerHello with ServerHello.server_version set to {03,00}.  Any party
receiving a Hello message with the protocol version set to {03,00} MUST
respond with a "protocol_version" alert message and close the connection.

Historically, TLS specifications were not clear on what the record layer version
number (TLSPlaintext.version) could contain when sending ClientHello.  Appendix
E of {{RFC5246}} notes that TLSPlaintext.version could be selected to maximize
interoperability, though no definitive value is identified as ideal.  That
guidance is still applicable; therefore, TLS servers MUST accept any value
{03,XX} (including {03,00}) as the record layer version number for ClientHello,
but they MUST NOT negotiate SSLv3.


# A Litany of Attacks

## Record Layer

The non-deterministic padding used in the CBC construction of SSLv3
trivially permits the recovery of plaintext {{POODLE}}. More generally,
the cipher block chaining (CBC) modes of SSLv3 use a flawed MAC-then-encrypt
construction that has subsequently been replaced in TLS versions {{RFC7366}}.
Unfortunately, the mechanism to correct this flaw relies on extensions: a
feature added in TLS 1.0.  SSLv3 cannot be updated to correct this flaw in the
same way.

The flaws in the CBC modes in SSLv3 are mirrored by the weakness of the stream
ciphers it defines.  Of those defined, only RC4 is currently in widespread use.
RC4, however, exhibits serious biases and is also no longer fit for use
{{I-D.ietf-tls-prohibiting-rc4}}.

This leaves SSLv3 with no suitable record protection mechanism.


## Key Exchange

The SSLv3 key exchange is vulnerable to man-in-the-middle attacks when
renegotiation {{Ray09}} or session resumption {{TRIPLE-HS}} are used.
Each flaw has been fixed in TLS by means of extensions. Again,
SSLv3 cannot be updated to correct these flaws.


## Custom Cryptographic Primitives

SSLv3 defines custom constructions for PRF, HMAC and digital signature
primitives.  Such constructions lack the deep cryptographic scrutiny that
standard constructions used by TLS have received.  Furthermore, all SSLv3
primitives rely on SHA-1 {{RFC3174}} and MD5 {{RFC1321}}: these hash algorithms
are considered weak and are being systematically replaced with stronger hash
functions, such as SHA-256 {{FIPS180-2}}.


# Limited Capabilities

SSLv3 is unable to take advantage of the many features that have been added to
recent TLS versions.  This includes the features that are enabled by ClientHello
extensions, which SSLv3 does not support.

Though SSLv3 can benefit from new cipher suites, it cannot benefit from new
cryptographic modes.  Of these, the following are particularly prominent:

* Authenticated Encryption with Additional Data (AEAD) modes are added in
  {{RFC5246}}.

* Elliptic Curve Diffie-Hellman (ECDH) and Digital Signature Algorithm (ECDSA)
  are added in {{RFC4492}}.

* Stateless session tickets {{RFC5077}}.

* A datagram mode of operation, DTLS {{RFC6347}}.

* Application layer protocol negotiation {{RFC7301}}.


# IANA Considerations

This document has no IANA actions.


# Security Considerations

This entire document aims to improve security by identifying a protocol that is
not secure.
