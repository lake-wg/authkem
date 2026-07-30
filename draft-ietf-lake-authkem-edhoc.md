---
title: "KEM-based Authentication for EDHOC"
abbrev: "EDHOC-KEM"
category: std

docname: draft-ietf-lake-authkem-edhoc-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Lightweight Authenticated Key Exchange"
keyword:
 - EDHOC
 - Post-Quantum Cryptography
 - Key Encapsulation Mechanism (KEM)
venue:
  group: "Lightweight Authenticated Key Exchange"
  type: "Working Group"
  mail: "lake@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/lake/"
  github: "lake-wg/authkem"
  latest: "https://LPFraile.github.io/authkem/draft-ietf-lake-authkem-edhoc.html"

author:
  -
    fullname: Lidia Pocero Fraile
    initials: L.
    surname: Pocero Fraile
    organization: ISI, R.C. ATHENA
    street: Patras Science Park building
    city: Platani, Patras
    code: "26504"
    country: Greece
    email: pocero@athenarc.gr
  -
    fullname: Christos Koulamas
    initials: C.
    surname: Koulamas
    organization: ISI, R.C. ATHENA
    street: Patras Science Park building
    city: Platani, Patras
    code: "26504"
    country: Greece
    email: koulamas@athenarc.gr
  -
    fullname: Apostolos P. Fournaris
    initials: A. P.
    surname: Fournaris
    organization: ISI, R.C. ATHENA
    street: Patras Science Park building
    city: Patras
    code: "26504"
    country: Greece
    email: fournaris@athenarc.gr
  -
    fullname: Evangelos Haleplidis
    initials: E.
    surname: Haleplidis
    organization: ISI, R.C. ATHENA
    street: Patras Science Park building
    city: Platani, Patras
    code: "26504"
    country: Greece
    email: haleplidis@athenarc.gr



normative:
  RFC9528:
  RFC9052:
  I-D.ietf-lamps-kyber-certificates:
  RFC8392:
  RFC9360:
  RFC8949:
  RFC8742:
  RFC5116:
  I-D.spm-lake-pqsuites:

informative:
  RFC9794:
  RFC9053:
  RFC7252:
  RFC7959:
  RFC9177:
  I-D.uri-lake-pquake:
  I-D.celi-wiggers-tls-authkem:
  I-D.ietf-pquip-pqc-engineers:
  Noise:
    title: "The Noise Protocol Framework"
    author:
      -
        ins: "T. Perrin"
        name: "Trevor Perrin"
    date: "2018-07"
    target: "https://noiseprotocol.org/noise.html"
    seriesinfo:
      Revision: "34"

  PQNoise-CCS22: DOI.10.1145/3548606.3560577

  KEMBinding-CCS24: DOI.10.1145/3658644.3670283

  PQ-EDHOC-Access25: DOI.10.1109/ACCESS.2025.3633843
...

--- abstract

This document specifies extensions to the Ephemeral Diffie-Hellman Over COSE (EDHOC) protocol to provide resistance against quantum computer adversaries by incorporating Post-Quantum Cryptography (PQC) Key Encapsulation Mechanisms (KEMs) for both key exchange and authentication. It defines a new signature-free KEM-based authentication method in which both parties authenticate using KEMs, enabling quantum-resistant authentication without relying on digital signatures when PQC KEMs, such as the NIST-standardized ML-KEM, are used.


--- middle

# Introduction

The purpose of this document is to address the quantum-resistant transition of the Ephemeral Diffie-Hellman Over COSE (EDHOC) protocol by defining a new authentication method in which both parties use Key Encapsulation Mechanism (KEM)-based authentication together with Post-Quantum Cryptography (PQC) cipher suites supporting PQC KEM algorithms such as the NIST-standardized ML-KEM-512.

The specified protocol is part of a broader analysis of the post-quantum transition for EDHOC {{PQ-EDHOC-Access25}}.

## Motivation

The emerging Quantum Computing technologies bring new potential risks to the existing cryptographic infrastructures. Security mechanisms that rely on integer factorization or the discrete logarithm problem will be vulnerable to attacks by a Cryptographically Relevant Quantum Computer (CRQC). The European Commission recently issued a roadmap for the transition to Post-Quantum Cryptography (PQC), establishing a 2030 deadline for high-risk use cases and 2035 for medium-risk use cases, in alignment with the 2035 deadline set by the U.S. government for completing the transition to PQC in federal systems.

The U.S. National Institute of Standards and Technology (NIST) has concluded its PQC standardization process with the release of its first standardized PQC algorithms in three new Federal Information Processing Standards (FIPS): FIPS 203 (ML-KEM, based on CRYSTALS-Kyber), FIPS 204 (ML-DSA, based on CRYSTALS-Dilithium), and FIPS 205 (SLH-DSA, based on SPHINCS+). Additionally, FALCON has been selected for future standardization, and NIST has launched a new initiative to evaluate alternative PQC signature schemes with compact signatures and efficient verification speeds. Complementing these efforts, the Post-Quantum Use in Protocols (PQUIC) IETF Working Group (WG) is developing operational and design guidelines to support the transition. For example, {{RFC9794}} defines terminology for post-quantum/traditional Hybrid schemes, while ongoings draft such as {{I-D.ietf-pquip-pqc-engineers}} analyze the impact of CRQCs on existing systems and the challenges involved in transitioning to post-quantum algorithms.

The growing urgency to transition to PQC highlights the need to adapt EDHOC, whose current security relies on traditional Elliptic-Curve Cryptography(ECC), based on the discrete logarithm problem that is known to be vulnerable to attacks by CRQCs. The integration of the PQC mechanism into EDHOC raises important considerations around performance, as the protocol is explicitly designed for constrained environments where the number of handshake message rounds, network overhead, processing time, and power consumption are critical factors.

PQC algorithms generally have higher computational and memory costs compared to the classical cryptography algorithms they aim to replace because they often involve complex calculations and require larger byte sizes. Notably, the PQC digital signature schemes standardized by NIST, such as ML-DSA and SLH-DSA, use significantly large public keys and signatures, which can be difficult to transmit over constrained networks. It is important to note that while FALCON, also selected for standardization by NIST, provides much shorter signatures than the lattice-based schemes, its current implementations have been shown to be vulnerable to side-channel attacks. The new compact schemes under NIST evaluation should be more suitable for constrained environments. However, the current Cortex-M4 implementations of some of the most compact PQC signature schemes, like SNOVA, MAYO and OV-LP, still demand substantial memory resources, making them impractical for many constrained devices. Additionally, others, such as SQISign, have only recently been supported on such platforms, and performance benchmarks for their signature operations are still unavailable.

On the other hand, the standardized ML-KEM offers significantly higher computational efficiency compared to all other PQC KEMs (order of magnitude faster) and is at least three times more efficient than the fastest PQC signature schemes. Therefore, extending EDHOC with a new authentication method that enables a signature-free KEM-based EDHOC has the potential to reduce memory and processing requirements when ML-KEM is used. The approach can also result in lower network overhead compared to signature-based EDHOC implementations that rely on standardized PQC signature-based algorithms.

Some standardization efforts propose adopting the KEM-based authentication mechanism to mitigate the overhead introduced by PQC digital signatures. For example, {{I-D.celi-wiggers-tls-authkem}} specifies a KEM-based authentication scheme for TLS 1.3, while {{I-D.uri-lake-pquake}} aims to define a general Post-Quantum Authentication Key exchange protocoll, which based on the same approach.

This document describes a KEM-based authentication mechanism specifically for the EDHOC protocol, introducing a new authentication method intended to provide a PQC signature-free variant as the static DH authentication method intends. The static-DH authentication of EDHOC is based on the XX pattern of the Noise framework protocol {{Noise}}, where channel security guarantees are increasingly established by encrypting transmitted messages with keys derived from chains of shared secrets, as soon as those secrets become available. To align with this model, the KEM-based authentication Method defined in this document follows the approach outlined in {{PQNoise-CCS22}}, which provides a recipe for transforming classical Noise patterns into PQ variants. This specification defines the necessary modifications to the EDHOC protocol to support the PQ-Noise framework while preserving security properties comparable to those of the static-DH authentication method.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

## Key Encapsulation Mechanisms (KEMs) {#KEMs}

The Key Encapsulation Mechanism consists of 3 algorithms:

- **( pk, sk ) <- KEM.KeyGen( )**: The probabilistic key generation algorithm generates a KEM key pair consisting of a public encapsulation key ( pk ) and secret decapsulation key ( sk ).
- **( ss , ct ) <- KEM.Encapsulate( pk )**: The probabilistic encapsulation algorithm takes as input a public encapsulation key ( pk ) and produces a shared secret ( ss ) and ciphertext ( ct ).
- **( ss ) <- KEM.Decapsulate( ct, sk )**: The decapsulation algorithm takes as input a secret encacpsulation key ( sk ) and produce a shared secret ( ss ).

# Protocol Overview {#ProtoOverview}

This document defines a new authentication method for EDHOC for general scenarios in which both parties authenticate using KEMs and may initially be mutually unknown. It aims to provide a free-signature authentication scheme as the static DH authentication EDHOC method 3 does, which relies on the XX pattern from the Noise framework {{Noise}}, supporting mutual authentication and the transmission of encrypted public credentials. The proposed protocol adopts the approach provided by {{PQNoise-CCS22}} to transform the classical Noise XX pattern in EDHOC into a PQ Noise XX variant. This results in a quantum-resistant, KEM-only version of EDHOC when a PQC KEM is used.

The PQ translation of the Noise XX pattern requires introducing up to one additional round trip. With KEMs, the owner of the static key cannot combine their static private key with the ephemeral public key belonging to the other party to immediately prove their identity in the next message, as is possible with DH. Instead, the party must first receive a ciphertext that encapsulates its static public key, generated by the peer, before it can authenticate itself. This necessitates an additional key-confirmation message from the key owner, using the key derived from the encapsulated value.

The KEM-based EDHOC protocol consists of five mandatory messages (message_1, message_2, message_3, message_4, and message_5), and an error message, between an Initiator (I) and a Responder (R). Error handling and cipher suit negotiation mechanisms are the same as defined in {{RFC9528}}, Section 6. All EDHOC messages are CBOR Sequences as specified in {{RFC9528}}. {{Exchange}} illustrates a KEM-based authentication EDHOC message flow as well as the content of each message. The protocol elements in {{Exchange}} are introduced in this Section and in {{messages}}. Message formatting and processing are specified in {{messages}}.

**Figure: EDHOC Message Flow using the KEM-based Authentication Method**

~~~
Initiator                                                   Responder
|               METHOD, SUITES_I, pk_eph, C_I, EAD_1                |
+------------------------------------------------------------------->
|                             message_1                             |
|                                                                   |
|               ct_eph, Enc( C_R, ID_CRED_R, EAD_2 )                |
<-------------------------------------------------------------------+
|                             message_2                             |
|                                                                   |
|                   ct_R, Enc( ID_CRED_I, EAD_3 )                   |
+------------------------------------------------------------------->
|                             message_3                             |
|                                                                   |
|                     ct_I, AEAD( EAD_4, MAC_2 )                    |
<-------------------------------------------------------------------+
|                             message_4                             |
|                                                                   |
|                         AEAD( EAD_5, MAC_3 )                      |
+------------------------------------------------------------------->
|                             message_5                             |
~~~

{: #Exchange}

The parties exchange ephemeral and static KEM public keys, along with ciphertexts that encapsulate these keys, compute shared secrets and pseudorandom keys PRK, and derive symmetric session keys to encrypt message elements contained in intermediate handshake messages. All handshake messages include encrypted components protected with these derived session keys, offering varying levels of confidentiality and authenticity, except for the first message, which is sent in plaintext. The parties compute a shared secret session key, PRK_out, from which symmetric application keys are derived to protect application data. The Initiator derives these keys after receiving message_4, and the Responder after receiving message_5.

- pk_eph is the ephemeral KEM public key generated by the Initiator.
- ct_eph is the ephemeral ciphertext computed by the Responder with the KEM.encapsulation algorithm over the received ephemeral public key (pk_eph).
- ct_R is the responder ciphertext computed by the Initiator with the KEM.encapsulation algorithm over the static KEM public key of the Responder, retrieved from the received ID_CRED_R in message_2.
- ct_I is the Iniatiator ciphertext computed by the Responder with the KEM.encapsulation algorithm over the static KEM public key of the Initiator, retrieved from the received ID_CRED_I in message_1.
- "CRED_I and CRED_R are the authentication credentials containing the public authentication keys of I and R, respectively", as defined in {{RFC9528}}, Section 2.
- "ID_CRED_I and ID_CRED_R are used to identify and optionally transport the credentials of I and R, respectively", as defined in {{RFC9528}}, Section 2.
- "Enc(), AEAD(), and MAC() denote encryption, Authenticated Encryption with Associated Data, and Message Authentication Code, crypto algorithms applied with keys derived from one or more shared secrets calculated during the protocol", as defined in {{RFC9528}}, Section 2.
- "SUITES_I contains cipher suites supported by the Initiator and formatted and processed as specified in {{RFC9528}}, Section 3.6 and 6.3.2".
- "METHOD is an integer specifying the authentication method",as defined in {{RFC9528}}, Section 3.2. In this case method 5; see {{Method}}.
- C_I and C_R are Connection Identifiers chosen by the Initiator and Responder, respectively, as specified in {{RFC9528}}, Section 3.3.
- EAD_1, EAD_2, EAD_3, EAD_4, EAD_5 are External Authorization Data included in message_1, message_2, message_3, message_4 and message_5 respectively.

This protocol is designed so that it follows the provisions of {{RFC9528}}, that is, to encrypt and integrity protect as much information as possible and derive symmetric keys and random material using EDHOC_KDF with as much previous information as possible

## Protocol Elements {#ProtoElemnts}

This section describes the principal protocol elements that differ from the definitions of EDHOC and highlights the most important similarities. For the missing elements, the definitions in {{RFC9528}}, Section 3 SHOULD be consulted.

### Ephemeral KEM {#EphemeralKEM}

The ephemeral KEM is used to provide forward secrecy. The Initiator generates a new ephemeral KEM key pair in every new session to ensure that the compromise of long-term keys does not compromise past communications. The elements of the Ephemeral KEM are:

- The ephemeral KEM key pair ( pk_eph, sk_eph ) is generated by the Initiator using the following function:

  ~~~
  pk_eph, sk_eph <- KEM.KeyGen()

  ~~~
- The ephemeral shared secret ( ss_eph ) and the ephemeral ciphertext ( ct_eph ) are generated using the encapsulation and decapsulation functions: in the Responder  in the Initiator

  ~~~
  ss_eph, ct_eph <-  KEM.Encapsulate( pk_eph )
  ss_eph <-  KEM.decapsulation( ct_eph, sk_eph )

  ~~~

### Method {#Method}

The protocol extends EDHOC with a new KEM-based authentication method, where both parties use static KEM key pairs. The authentication is provided by a Message Authentication Code (MAC) included in message_4 and message_5 to authenticate the Responder and Initiator, respectively. The Initiator and Responder must agree on the authentication method to be used. The selected method is indicated by the Initiator in message_1.

**Table: Authentication Keys for Method Types**

| Method Type Value | Initiator Authentication Key | Responder Authentication Key |
| --- | --- | --- |
| 5 (suggested) | Static KEM Key | Static KEM Key |
{: #tab-method-types}

### Authentication Parameters {#AuthneticationPara}

The protocol performs the same authentication-related operations as described in {{RFC9528}}, Section 3.5

The protocol transports information about credentials ID_CRED_I and ID_CRED_R in message_2 and message_3, respectively. The authentication of these credentials is verified through MAC_2 and MAC_3, sent by the Responder and the Initiator in message_4 and message_5, respectively.

#### Authentication Keys {#Auth-keys}

The authentication key MUST be a static KEM authentication key pair.

The authentication key algorithm must be compatible with the chosen method and selected cipher suite. The same KEM algorithm selected for the EDHOC key exchange in the cipher suite MUST be used for both the ephemeral KEM key exchange and the authentication static KEM keys. The Initiators and Responders private and public authentication keys are denoted as follows:

- The Initiator static KEM authentication key pair: ( pk_I, sk_I )
- The Responder static KEM authentication key pair: ( pk_R, sk_R )

#### Authentication Credentials {#Auth-cred}

The authentication credentials, CRED_I and CRED_R, contain the authentication public key of the Initiator and Responder, respectively, as described in {{RFC9528}}, Section 3.5.2.

- The authentication credentials can be X.509 certificates seconded as bstr, as defined in {{RFC9528}}, Section 3.5.2, using {{RFC9360}}. {{I-D.ietf-lamps-kyber-certificates}} describes the conventions for using the ML-KEM in X.509 Public Key Infrastructure.
- Additionally, the authentication credential may include a COSE_key, formatted as specified in {{RFC8392}}, to reduce the credential size and avoid the PQC signature verification needed when X.509 certificates are used. New IANA value registries should be defined to extend COSE Algorithms with the corresponding KEMs algorithm values.

#### Identification of Credentials {#Ident-cred}

The ID_CRED fields are used to identify and optionally transport credentials as defined in {{RFC9528}}, Section 3.5.3. The authentication method defined in this document operates within the general EDHOC framework described in {{RFC9528}}, Section 3.5.3, where ID_CRED_X can either contain the full CRED_X credentials or an identifier of those credentials if they have already been provided out-of-band.

- "ID_CRED_R is intended to facilitate for the Initiator retrieving the authentication credential CRED_R and the authentication key of R", as defined in {{RFC9528}}, Section 3.5.3. For the authentication method defined in this document, the authentication key is the static KEM public key.
- "ID_CRED_I is intended to facilitate for the Responder retrieving the authentication credential CRED_I and the authentication key of I", as defined in {{RFC9528}}, Section 3.5.3. For the authentication method defined in this document, the authentication key is the static KEM public key.

### Cipher Suites {#Ciphersuit}

The authentication method specified in this document uses the EDHOC cipher suites element, as defined in {{RFC9528}}, Section 3.6. An EDHOC cipher suit consists of an ordered set of algorithms from the "COSE Algorithms" IANA registry {{RFC9053}}. The predefined EDHOC cipher suites are also listed in the IANA registry, as specified in {{RFC9528}}, Section 10.2.

A new predefined cipher suite SHOULD be added to the IANA registry, specifying each supported KEM in the EDHOC Key Exchange Algorithm parameter. An example of this, when ML-KEM is used, is shown in {{IANA}}. The same KEM algorithm selected for key exchange SHOULD also be used for KEM-based authentication when method 5 is selected. Furthermore, the KEM algorithms used SHOULD also be added to the COSE Algorithms IANA registry to identify them, as is shown in {{IANA}}.

### Transport {#Trasport}

The KEM-based authentication method for EDHOC is not bound to any specific transport layer, similar to the classical EDHOC methods defined in {{RFC9528}}, Section 3.4. However, the resulting message sizes are expected to be larger than those of the original EDHOC methods specified in {{RFC9528}}. This is because the currently standardized NIST KEM algorithms use comparatively large public keys and key encapsulation (ciphertext) sizes, thereby increasing the overall size of EDHOC messages.

In highly constrained networks, larger message sizes MAY necessitate transport support for fragmentation. For example, if the network MTU is insufficient to carry a complete message, the messages can be transported over CoAP {{RFC7252}} using the Block-Wise Transfer mechanism to support fragmentation and reassembly, as specified in {{RFC7959}} or {{RFC9177}}. {{RFC7959}} defines the Block1 and Block2 options for request/response block-wise transfer in CoAP, while {{RFC9177}} extends this mechanism with the Q-Block1 and Q-Block2 options, allowing multiple blocks to be transmitted without waiting for per-block acknowledgments.

# Key Derivation {#key-derive}

This section highlights the differences and similarities in the key derivation process of the KEM-based authentication method compared to {{RFC9528}}. An overview of the EDHOC key schedule when using the KEM-based authentication method is shown in {{key}}, and each key derivation step is explained in the following subsections.

**Figure: EDHOC Message Key Derivation using the KEM-based Authentication Method**

~~~

          +-------+
          | TH_2  |
          +-------+
           |
+----+  +--v+  +------+  +---+  +-----+  +-+  +---+
|ss_e|->|Ext|->|PRK_2e|--|Exp|->|KEY_2|->| |->|C_2|
+----+  +---+  +--+---+  +--++  +-----+  |X|  +---+
                  |         |  PLAIN_2-->| |
           +------+         |            +-+
           |           +--+----------------------+
           |           |TH_2=H(H(Message1),ct_eph)|
           |           +-------------------------+
           |
+----+  +--++  +--v-----+  +------+  +---+  +----+  +---+
|ss_R|->|Ext|->|PRK_3e2m|+-|Expand|->|K_3|->|AEAD|->|C_3|
+----+  +---+  +----+---+ |+--+---+  +---+  +----+  +---+
                    |     |   |               |
                    |     |   |          PLAINTEXT_3
           +--------+     | +--+--------------------+
           |              | |TH_3=                  |
                          | |  H(TH_2,PLAINTEXT_2,  |
                          | |  CRED_R,ct_R)        |
           |              | +-----------------------+
           |              |
           |              |  +------+  +-----+
           |              +--|Expand|->|MAC_2|
           |                 +-+----+  +-----+
           |                   |
           |   +---------------+--------------------+
           |   |TH_4=H(TH_3,PLAINTEXT_3,CRED_I,ct_I)|
           |   +------------------+-----------------+
           |                      |
+----+  +--+-+  +--------+ +---+  +---+  +----+  +---+
|ss_I|->|Ext|->|PRK_4e3m|+-|Exp|->|K_4|->|AEAD|->|C_4|
+----+  +----+  +--------+|+---+ |+---+ |+-^--+  +---+
                          |       |     |  |
                          |       |     | PLAINTEXT_4
                          |       |     |+----+  +---+
                          |       |     -|AEAD|->|C_5|
+------------------------+|       |      +-^--+  +---+
|TH_5=H(TH_4,PLAINTEXT_4)||       |        |
+--+---------------------+|       |       PLAINTEXT_5
                    |     |       |
      +-----+   +-+----+  |       |
      |MAC_3|<--|Expand|--|       |
      +-----+   +------+          |   +-------+
                                  |-->|PRK_out|
                                      +--+----+
                                         |
                                      +--v---+
                                      |Expand|
                                      +------+
                                         |
                                         v
                                    +------------+
                                    |PRK_exporter|
                                    +---+--------+
                                        |
                                     +--v---+
                                     |Expand|
                                     +--+---+
                                        |
                                        v
                                  Aplication Key

~~~

{: #key}

## Keys for EDHOC Message Processing

### EDHOC_Extract

The pseudorandom keys (PRKs) used for KEM-based authentication method are derived using the same EDHOC_Extract function defined in {{RFC9528}}, where the input keying material (IKM) and Salt are specified for each PRK below.

#### PRK_2e

The pseudorandom key PRK_2e is derived with the following input:

- The salt SHALL be TH_2.
- The IKM SHALL be the ephemeral KEM shared secret (ss_eph)

When SHA-256 is used PRK_2e is produced as follows:

~~~
PRK_2e = HMAC-SHA-256( TH_2, ss_eph )

~~~

Where the ephemeral shared secret ss_eph is the output of the following functions in the Initiator and Responder respectively

Initiator:

~~~
ss_eph -  KEM.Decapsulate( ct_eph, sk_eph )

~~~

Responder:

~~~
ss_eph, ct_eph -  KEM.Encapsulate( pk_eph )

~~~

#### PRK_3e2m {#prk_3e2m}

The pseudorandom key PRK_3e2m is derived with the following input:

- The salt SHALL be the SALT_3e2m derived from PRK_2e
- The IKM SHALL be the KEM shared secret ss_R, used to authenticate the Responder

PRk_3e2m is derived as follows:

~~~
PRK_3e2m = EDHOC_Extract( SALT_3e2m, ss_R )

~~~

Where the KEM shared secret ss_R used to authenticate the Responder is the output of the following functions in the Initiator and Responder, respectively

Initiator:

~~~
ss_R, ct_R ->  KEM.Encapsulate( pk_R )

~~~

Responder:

~~~
ss_R ->  KEM.Decapsulate( ct_R, sk_R )

~~~

#### PRK_4e3m {#PRK_4e3m}

The pseudorandom key PRK_4e3m is derived with the following input:

- The salt SHALL be the SALT_4e3m, derived from PRK_3e2m
- The IKM SHALL be the KEM shared secret ss_I, used to authenticate the Initiator

PRk_4e3m is derived as follows:

~~~
PRK_4e3m = EDHOC_Extract( SALT_4e3m, ss_I )

~~~

Where the KEM shared secret ss_I used to authenticate the Initiator is the output of the following functions in the Initiator and Responder, respectively

Initiator:

~~~
ss_I -  KEM.Decapsulate( ct_I, sk_I )

~~~

Responder:

~~~
ss_I, ct_I -  KEM.Encapsulate( pk_I )

~~~

### EDHOC_Expand and EDHOC_KDF {#edhoc_kdf}

The output key materials (OKMs) are derived from the PRKs in the same way as described in {{RFC9528}}, Section 4.1.2, with modifications in the transcript hashes THs input contraction as specified in {messages}.

The same OKMs, including keys, initialization vectors (IV), and salts as those shows in {{RFC9528}}, Section 4.1.2 Figure 6 are derived, just notice that:

- K_3 and IV_3 are computed to provide integrity protection and confidentiality for message_3 ensuring that the Initiators identity is protected against active attacks. However, this does not provide authentication of the Initiators identity.

The final key derivations using EDHOC_KDF is shwon in {key-derivations-kdf}. Further details of the key derivation and how the output keying material is used are specified in {{messages}}

**Figure: Key Derivations Using EDHOC_KDF for the KEM-based Authentication Methods**

~~~
KEYSTREAM_2   = EDHOC_KDF( PRK_2e,   0, TH_2,      plaintext_length )
SALT_3e2m     = EDHOC_KDF( PRK_2e,   1, TH_2,      hash_length )
MAC_2         = EDHOC_KDF( PRK_3e2m, 2, context_2, mac_length_2 )
K_3           = EDHOC_KDF( PRK_3e2m, 3, TH_3,      key_length )
IV_3          = EDHOC_KDF( PRK_3e2m, 4, TH_3,      iv_length )
SALT_4e3m     = EDHOC_KDF( PRK_3e2m, 5, TH_4,      hash_length )
MAC_3         = EDHOC_KDF( PRK_4e3m, 6, context_3, mac_length_3 )
PRK_out       = EDHOC_KDF( PRK_4e3m, 7, TH_4,      hash_length )
K_4           = EDHOC_KDF( PRK_4e3m, 8, TH_4,      key_length )
IV_4          = EDHOC_KDF( PRK_4e3m, 9, TH_4,      iv_length )
PRK_exporter  = EDHOC_KDF( PRK_out, 10, h'',       hash_length )
~~~

{: #key-derivations-kdf}

Notice that a new key session (K_5/IV_5) can be derived from the same PRK_4e3m, using TH_5 as the info parameter, to encrypt message_5, if separate keys are shown to enhance security in any way. The initial version of this protocol adopts a simpler approach by deriving a single session key to protect both messages, which are different from the MAC keys.

### PRK_out {#PRK_out}

The pseudorandom key PRK_out is the output session key of a completed EDHOC session and is derived as follows:

~~~
PRK_out = EDHOC_KDF( PRK_4e3m, TH_4, hash_length )

~~~

## Keys for EDHOC Applications {#key_app}

Keying material for the application can be derived using the same EDHOC_Exporter interface defined in {{RFC9528}}, Section 4.2.1

# Message Formatting and Processing {#messages}

This section outlines the message format and the procedures for composing and processing each message.

## KEM-based Authentication EDHOC Message 1 {#message1}

### Formating of Message 1 {#fmessage1}

message_1 retains the same format as defined in {{RFC9528}}, Section 5.2.1. The same fields are used, except that GX is replaced by the KEM ephemeral public key ( pk_eph ) computed by the Initiator.

~~~ cddl
message_1 = (
  METHOD : int,
  SUITES_I : suites,
  pk_eph : bstr,
  C_I : bstr / -24..23,
  ? EAD_1,
)

suites = [ 2* int ] / int
EAD_1 = 1* ead
~~~

The KEM-based authentication method (proposed method 5) should be seletect in the METHOD field.

### Initiator Composition of Message 1 {#icmessage1}

The Initiator SHALL compose message_1 as follows:

- Construct SUITES_I following the {{RFC9528}}, Section 5.2.2 specifications
- Generate an ephemeral KEM Key pair (pk_eph) using the KEM algorithm from the selected cipher suit. The ephemeral key pair is computed by the Initiator using the following function:

~~~
  pk_eph, sk_eph -> KEM.KeyGen()

~~~

- Choose a conection identifier as in {{RFC9528}}, Section 5.2.2.
- Encode message_1 as sequence of CBOR-encoded elements, as specified in {{fmessage1}}

### Responder Processing of Message 1 {#rpmessage1}

The Responder SHALL process message_1 in the following order:

1. "Decode message_1", as specified in {{RFC9528}}, Section 5.2.3
2. "Process message_1", as specify in {{RFC9528}}, Section 5.2.3
3. "If all processing is completed successfully, and if EAD_1 is present, then make it available to the application", as specified in {{RFC9528}}, Section 5.2.3

## KEM-based authentication EDHOC Message 2 {#message2}

### Formating of Message 2 {#fmessage2}

message_2 keeps the same formatting as {{RFC9528}}, Section 5.3.1. The same fields are used instead GY is replaced with the ephemeral KEM ciphertext ( ct_eph ) computed on the Responder.

~~~
message_2 = (
  ct_eph_CIPHERTEXT_2 : bstr,
)

~~~

where cc_eph_CIPHERTEXT_2 is the concatenation of ct_eph and CIPHERTEXT_2.

### Responder Composition of Message 2 {#rcmessage2}

The Responder SHALL compose message_2 as follows:

- Encapsulate the ephemeral KEM key received within message_1 using the KEM algorithm in the selected cipher suit. The ephemeral KEM ciphertext and the KEM ephemeral shared secret are computed by the Responder using the following function:

  ~~~
    ss_eph, ct_eph <-  KEM.Encapsulate(pk_eph)

  ~~~

- Compute the transcript hash TH_2 = H(pk_eph,H(message_1)) as specified in {{RFC9528}}, Section 5.3.2
- Compute the PRK_2e pseudorandom key from the ephemeral KEM shared secret ( ss_eph )
- "Choose a connection identifier C_R", as specified in {{RFC9528}}, Section 5.3.2
- At this point, the Responder is not jet able to authenticate itself, so MAC_2 is not computed
- CIPHERTEXT_2 is calculated, with a binary additive stream cipher as in {{RFC9528}}, Section 5.3.2, using a keystream (KEYSTREAM_2) generated with EDHOC_Expand and the following plaintext:
  - Compute PLAINTEXT_2 as:  where C_R, ID_CRED_R and EAD_2 elements corresponds with the ones in {{RFC9528}}, Section 5.3.2.

    ~~~
       PLAINTEXT_2 = (C_R,ID_CRED_R,?EAD_2)

    ~~~

  - Compute KEYSTREAM_2 as in {{edhoc_kdf}}
  - Compute CIPHERTEXT_2 as in {{RFC9528}}, Section 5.3.2

    ~~~cddl
    CIPHERTEXT_2 = PLAINTEXT_2 XOR KEYSTREAM_2

    ~~~

- Encode message_2 as a sequence of CBOR-encoded data items as specified in {{fmessage2}}

### Initiator Processing of Message 2 {#ipmessage2}

The Initiator SHALL process message_2 in the following order:

1. Decode message_2
2. "Retrieve the protocol state" as proposed in {{RFC9528}}, Section 5.3.3
3. Compute the ephemeral KEM shared_secret ( ss_eph ) by decapsulating the KEM ciphertext ( ct_eph ) received in message_2 using the ephemeral secret key ( sk_eph ). The ephemeral KEM shared secret is computed by the Initiator using the following function:

~~~
  ss_eph <-  KEM.Decapsulate( ct_eph, sk_eph )

~~~

4. Compute the transcript hash TH_2 = H(pk_eph,H(message_1))
5. Compute the PRK_2e pseudorandom key from the ephemeral KEM shared secret ( ss_eph )
6. Derive KEYSTREAM_2 as in {{edhoc_kdf}}
7. Decrypt CIPHERTEXT_2; see {{rcmessage2}}
8. If all processing is completed successfully, ID_CRED_R and (if present) EAD_2 SHALL be made available to the application, as specified in {{RFC9528}}, Section 5.3.3. In this specification, the application MUST authenticate and validate the credentials associated with ID_CRED_R at this point before proceeding. The Initiators credentials are transmitted in the subsequent message and are encrypted under a key that can only be derived by a party possessing the private key corresponding to ID_CRED_R. Prior to sending its credentials, the Initiator MUST ensure that the credentials associated with ID_CRED_R have been successfully validated and accepted according to local policy. This prevents disclosure of the Initiators credentials to a party presenting credentials that are cryptographically valid but untrusted or unintended.
9. Obtain the authentication credential (CRED_R) from the (ID_CRED_R) as in {{RFC9528}}, Section 5.3.3, and the static authentication key of the Responder
10. Encapsulate the retrieved static KEM authentication key of the Responder ( pk_R ) calculating the corresponding ciphertext ( ct_R ) and shared secret ( ss_R ) with the following function:

~~~
  ss_R, ct_R ->  KEM.Encapsulate(pk_R)

~~~
11. Compute the new PRK_3e2m from a chain that includes both the ephemeral KEM shared secret ( ss_eph ) and the latest KEM shared secret for the Authentication of the Responder ( ss_R ), as defined in {{prk_3e2m}}

## KEM-based authentication EDHOC Message 3 {#message3}

### Formating of Message 3 {#fmessage3}

message_3 SHALL be a CBOR Sequence as defined below:

~~~ cddl
message_3 = (
  ct_R : bstr,
  CIPHERTEXT_3 : bstr,
)
~~~

### Initiator Composition of Message 3 {#icmessage3}

The Initiator SHALL process the composition of message_3 as follows:

- Compute the transcript hash TH_3=H(ct_R,TH_2,PLAINTEXT_2,CRED_R) as specified in {{RFC9528}}, Section 5.4.2.
- Derive the new session key K_3/IV_3 as defined in {{edhoc_kdf}}. The Initiator can use this key to compute CIPHERTEXT_3, but it cannot be used to authenticate itself.
- At this point, the Responder is not jet able to authenticate itself, so MAC_3 is not computed.
- Compute a COSE_Encrypt0 object as defined in {{RFC9052}}, Section 5.2 and 5.3, with the EDHOC AEAD algorithm of the selected cipher suite, using the encryption key K_3, the initialization vector IV_3 (if used by the AEAD algorithm), the plaintext PLAINTEXT_3, and the following parameters as input:  CIPHERTEXT_3 is the 'ciphertext' of COSE_Encrypt0.
  - protected = h''
  - external_aad = TH_3
  - K_3 and IV_3 are defined in {{edhoc_kdf}}
  - PLAINTEXT_3 = (C_I,ID_CRED_I,?EAD_3) where C_I, ID_CRED_I and EAD_3 elements corresponds with the ones in {{RFC9528}}, Section 5.3.3
- Encode message_3 as a CBOR data item as specified in {{fmessage3}}.

### Responder Processing of Message 3 {#rpmessage3}

The Responder SHALL process message_3 in the following order:

1. Decode message_3
2. "Retrieve the protocol state", as defined in {{RFC9528}}, Section 5.4.3
3. Compute the KEM shared_secret ( ss_R ) for the authentication of the Responder by decapsulating the KEM ciphertext ( ct_R ) received in message_3 using the Responder static KEM secret key ( sk_R ). The KEM shared secret is computed by the Responder using the following function:

~~~cddl
  ss_R -  KEM.Decapsulate( ct_R, sk_R )

~~~

4. Compute the new PRK_3e2m from a chain that includes both the ephemeral KEM shared secret ( ss_eph ) and the latest KEM shared secret for the Authentication of the Responder ( ss_R ), as defined in {{prk_3e2m}}
5. Compute the transcript hash TH_3=H(ct_R,TH_2,PLAINTEXT_2,CRED_R)
6. Compute K_3/IV_3 as in {{edhoc_kdf}}, where plaintext_length is the length of PLAINTEXT_3
7. Decrypt CIPHERTEXT_3; see {{icmessage3}}
8. "If all processing is completed successfully, then make ID_CRED_I and (if present) EAD_2 available to the application", as in {{RFC9528}}, Section 5.3.4
9. "Obtain the authentication credential (CRED_I) from the (ID_CRED_I)" as in {{RFC9528}}, Section 5.3.4 and the static authentication key of the Initiator.

## KEM-based authentication EDHOC Message 4 {#message4}

### Formating of Message 4 {#fmessage4}

message_4 SHALL be a CBOR Sequence as defined below:

~~~ cddl
message_3 = (
  ct_I : bstr,
  CIPHERTEXT_4 : bstr,
)
~~~

### Responder Composition of Message 4 {#rcmessage4}

The Responder SHALL process the composition of message_4 as follows:

- Encapsulate the retrieved static KEM authentication key of the Initiator ( pk_I ) calculating the corresponding ciphertext ( ct_I ) and shared secret ( ss_I ) with the following function:

  ~~~cddl
   ss_I, ct_I -  KEM.Encapsulate(pk_I)

  ~~~

- Compute the transcript hash TH_4 = H(ct_I,TH_3, PLAINTEXT_3, CRED_I)
- Compute MAC_2 as defined in {{edhoc_kdf}}, with context_2 = C_R, ID_CRED_R, TH_4, CRED_R, ? EAD_4
  - The Responder authenticates with a PRK_3e2m derived from the KEM ephemeral shared secret and with the shared secret computed over its static KEM key.
  - The mac_lenght_2 is equal to the EDHOC MAC length of the selected cipher suit.
  - The C_R, ID_CRED_R and CRED_R elements corresponds with the ones in {{RFC9528}}, Section 5.3.2
  - The latest transcript hash TH_4 and the External Application Data included in Message 4 (EAD_4) are used.
- Compute the new PRK_4e3m from a chain that includes the ephemeral KEM shared secret ( ss_eph ), the KEM shared secret for the Authentication of the Responder ( ss_R ) , and the latest KEM shared secret for the Authentication of the Initiator ( ss_I ) as defined in {{PRK_4e3m}}
- Derive the session key K_4/IV4 as in {{edhoc_kdf}}.
- Compute a COSE_Encrypt0 object as defined in {{RFC9052}}, Section 5.2 and 5.3, with the EDHOC AEAD algorithm of the selected cipher suite, using the encryption key K_4, the initialization vector IV_4 (if used by the AEAD algorithm), the plaintext PLAINTEXT_4, and the following parameters as input:  CIPHERTEXT_4 is the 'ciphertext' of COSE_Encrypt0.
  - protected = h''
  - external_aad = TH_4
  - K_4 and IV_4 are defined in {{edhoc_kdf}}
  - PLAINTEXT_4 = ( MAC_2, ?EAD_4 )
- Compute the transcript hash TH_5 = H(TH_4, PLAINTEXT_4)
- Encode message_4 as a CBOR data item as specified in {{fmessage4}}.

### Initaitor Processing of Message 4 {#ipmessage4}

The Initiator SHALL process message_4 in the following order:

1. Decode message_4
2. "Retrieve the protocol state using available message correlation", as in {{RFC9528}}, Section 3.4.2.
3. Compute the KEM shared secret ( ss_I ) for the authentication of the Initiator by decapsulating the KEM ciphertext ( ct_I ) received in message_4 using the Responder static KEM secret key ( sk_I ). The KEM shared secret is computed by the Initiator using the following function:

~~~cddl
  ss_I -  KEM.Decapsulate( ct_I, sk_I )

~~~

4. Compute the new PRK_4e3m from a chain that includes the ephemeral KEM shared secret ( ss_eph ), the KEM shared secret for the Authentication of the Responder ( ss_R ), and the latest KEM shared secret for the Authentication of the Initiator ( ss_I ) as defined in {{PRK_4e3m}}
5. Derive the session key K_4/IV4 as in {{edhoc_kdf}}.
6. Decrypt and verify the COSE_Encrypt0 (CIPHERTEXT_4) as defined {{RFC9052}}, Section 5.2 and 5.3, with the EDHOC AEAD algorithm in the selected cipher suite and the parameters defined in {{rcmessage4}}.
7. Verify MAC_2 as defined in {{rcmessage4}}, and make the result of the verification available to the application.

## KEM-based authentication EDHOC Message 5 {#message5}

### Formating of Message 5 {#fmessage5}

message_5 SHALL be a CBOR Sequence as defined below:

~~~ cddl
message_3 = (
  CIPHERTEXT_5 : bstr,
)
~~~

### Initiator Composition of Message 5 {#icmessage5}

The Initiator SHALL process the composition of message_5 as follows:

- Compute the transcript hash TH_5 = H(TH_4, PLAINTEXT_4)
- Compute MAC_3 as defined in {{edhoc_kdf}}, with context_3 = C_I, ID_CRED_I, TH_5, CRED_I, ? EAD_5
  - The Initiator authenticates with a PRK_4e3m derived from the three shared secrets, including the shared secret computed over its static KEM key ( ss_I ).
  - The mac_lenght_3 is equal to the EDHOC MAC length of the selected cipher suit.
  - The C_I, ID_CRED_I and CRED_I elements corresponds with the ones in {{RFC9528}}, Section 5.4.2
  - The latest transcript hash TH_5 and the External Application Data included on Message 5 (EAD_5) are used.
- Compute a COSE_Encrypt0 object as defined in{{RFC9052}}, Section 5.2 and 5.3, with the EDHOC AEAD algorithm of the selected cipher suite, using the encryption key K_4, the initialization vector IV_4 (if used by the AEAD algorithm), the plaintext PLAINTEXT_5, and the following parameters as input:  CIPHERTEXT_5 is the 'ciphertext' of COSE_Encrypt0.
  - protected = h''
  - external_aad = TH_5
  - K_5 and IV_5 are defined in {{edhoc_kdf}}
  - PLAINTEXT_5 = ( MAC_3, ? EAD_5 )
- Calculate PRK_out as defined in {{PRK_out}}. The Initiator can now derive application keys using the EDHOC_Exporter interface; see {{key_app}}
- Encode message_5 as a CBOR data item as specified in {{fmessage5}}
- "Make the connection identifiers (C_I and C_R) and the application algorithms in the selected cipher suite available to the application" as in {{RFC9528}}, Section 5.4.2

After creating message_5, the Initiator can compute PRK_out and derive application keys using the EDHOC_Exporter interface. The Initiator SHOULD now persistently store PRK_out or application keys and send protected application data, since it has already verified message_4, which is protected with a derived application key by the Responder, and the application has authenticated the Responder.

### Responder Processing of Message 5 {#rpmessage5}

The Responder SHALL process message_5 in the following order:

1. Decode message_5
2. "Retrieve the protocol state using available message correlation" as in {{RFC9528}}, Section 3.4.2.
3. Decrypt and verify the COSE_Encrypt0 (CIPHERTEXT_5) as defined in {{RFC9052}}, Section 5.2 and 5.3, with the EDHOC AEAD algorithm in the selected cipher suite and the parameters defined in {{icmessage5}}.
4. Verify MAC_3 as defined in {{icmessage5}}, and make the result of the verification available to the application.
5. Calculate PRK_out as defined in {{PRK_out}}. The Initiator can now derive application keys using the EDHOC_Exporter interface; see {{key_app}}

After verifying message_5, the Responder can compute PRK_out and derive application keys using the EDHOC_Exporter interface. The Responder SHOULD now persistently store PRK_out or application keys and send protected application data, since it has already verified message_5, which is protected with a derived application key by the Initiator, and the application has authenticated the Initiator.


# Security Considerations {#Security}

## Security Properties {#SecurityP}

EDHOC protocol with static DH keys enables the Initiator and Responder to generate an ephemeral-static shared secret using the other party's ephemeral public keys and their own credentials. This shared secret is then used to derive a session key for authentication. Messages 2 and 3 provide explicit authentication through MACs, which also bind the exchanged credentials to prevent misbinding attacks, as is described in {{RFC9528}}, Section 9.1

In contrast, the KEM-based authentication mechanism requires an initial action from the other party. The Responder must first receive the encapsulation of its static public key generated by the Initiator to authenticate itself. To perform this encapsulation, the Initiator must retrieve the static KEM public key of the Responder from the ID_CRED_R sent in Message 2. As a result, the Responder cannot authenticate itself until Message 3 is processed, which contains the ct_R ciphertext necessary to derive the ss_R shared secret. Then it cannot generate MAC_2 or authenticate itself until then. Similarly, the Initiator cannot generate MAC_3 or authenticate itself before sending Message 3. This highlights the main challenge in integrating KEM-based authentication method within the EDHOC handshake.

To address this issue and maintain the same level of identity protection than EDHOC, against active attacks on the Initiator and passive attacks on the Responder, the credentials continue to be encrypted in Messages 2 and 3. In message_2, the Responders credentials are included in a plaintext that is XORed with a key derived from the ephemeral shared secrets, as defined in {{RFC9528}}, Section 5.3. By employing the same construction, this specification provides equivalent identity protection for the Responder against passive attackers. The credentials of the Initiator ( ID_CRED_I ) are encrypted using an AEAD algorithm to provide integrity protection and confidentiality, but not authentication, because the Initators shared secret is not yet available to prove its identity. The encryption key is derived from a combination of both ephemeral KEM shared secret (ss_eph) and the Responder static KEM shared secret ( ss_R ), used to authenticate the Responder. At this stage in the protocol, the specific encryption provided a form of weak forward secrecy, as the Initiator has not yet verify the static KEM public key of the Responder. However, the Initiators credentials are still protected against active attacks, as only the legitimate Responder, who possesses the corresponding private key ( sk_R ) is capable of deriving the session key and decrypting message_3.

Furthermore, the protocol is extended with two additional messages (Messages 4 and 5) to enable both parties to:

- Prove possession of the final session key, ensuring key confirmation to the other party
- Ensure mutual authentication by explicitly authenticating themselves using the final session key, which incorporates all three shared secrets: the ephemeral KEM shared secret ( ss_eph ) and the KEM shared secrets ss_I and ss_R used to authenticate the Initiator and Responder, respectively.
- Provide credential binding by including MAC_2 and MAC_3, ensuring the integrity and authenticity of the credentials exchanged in messages 2 and 3.

In {{RFC9528}}, the transcript hashes (THs) are constructed as an accumulative hash, combining previous TH values with the current plain-text message. Each new plain-text message in the handshake is concatenated with the previous TH value, and the resulting hash forms the new TH. This process links each message in the sequence to all prior messages, creating a verifiable and continuous chain. As a result, any changes to the message content are detected during subsequent integrity verification using the transcript hashes. The KEM-based authentication method described in this document extends this approach. Both parties only need to verify the integrity and authenticity of the latest TH_4 and TH_5, which encompass all previous messages in the handshake. To facilitate this, the MAC-protected data in Messages 4 and 5 is modified to include TH_4 and TH_5 respectively. At the end of the handshake, both the Initiator and Responder can verify the integrity and authenticity of the entire handshake by checking the received MACs.

The payload security properties for the static DH authentication method and the KEM-based authentication method differ during the handshake. Unlike the static DH authentication method, the KEM-based method exhibits no authentication until the final two messages. It provides the same level of destination confidentiality for the first two and the last two messages, while message_3 offers weaker forward secrecy. The Initiators credentials are encrypted within message_3 using a key derived from the Responders static public key and the ephemeral key, ensuring that only the intended Responder can decrypt the credential, and protect them against active attacks.

Full forward secrecy and explicit mutual authentication are achieved once the KEM-based method handshake is completed, similar to the static-DH method handshake (described in {{RFC9528}}, Section 9.1). Additionally, a potential misbinding attack will not be detected until the handshake concludes, specifically when the Initiator verifies Message 4 and the Responder verifies Message 5. Therefore, EAD data should be treated as unprotected, and keying materials should not be persistently stored until the protocol is complete, as with the static-DH method (described in {{RFC9528}}, Section 9.1). The final Application Session Key should only be derived at the end of the handshake, after ensuring mutual authentication, message handshake integrity, credentials authenticity, and proof of key possession.

The KEM-based authentication method does not provide non-repudiation, but only implicit proof of participation, similar to EDHOC with static DH keys. It also maintains an equivalent level of downgrade protection, as the negotiation base of the protocol is unchanged.

## KEM Security Considerations {#KEMsec}

{{KEMBinding-CCS24}} demonstrates that IND-CCA2 security alone does not preclude re-encapsulation attacks in KEM-based key exchange protocols. Such attacks can lead to unknown key-share conditions, in which two honest parties derive the same shared secret while associating it with different peer identities. Therefore, any KEM used in this specification MUST achieve IND-CCA2 security and MUST ensure that the derived shared secret is cryptographically bound to the recipients public key. This requirement prevents re-encapsulation and related key-substitution attacks.

## Four-Message Variant {#Four}

The proposed KEM-based authentication method with a 5-message handshake is designed to meet the same security requirements as static-DH method. However, it can be adapted to reduce the number of round trips while remaining suitable for scenarios where neither party knows the other beforehand. This is achieved by transmitting the ID_CRED_I credentials of the Initiator in plain-text within the message_1, similar to the Noise IX pattern, where the Initiator's static key is immediately transmitted to the Responder, despite or absent identity protection. This modification allows the Initiator to authenticate itself in Message 2, eliminating the need for Message 5. Since the Responder includes its credentials in the first message, Message 4 remains necessary to ensure explicit authentication of the Responder. This adaptation reduces the message exchange to four but sacrifices identity protection for the Initiator's credentials.

# IANA Considerations {#IANA}

## COSE Algorithms Registry {#cose-algorithms-registry}

The "COSE Algorithms" Registry from "CBOR Object Signing and Encryption (COSE)" group SHOULD be extended with new values to include PQC KEM algorithms. The extension of values from the "Standards Action with Expert Review" range for ML-KEM algorithms at NIST security levels 1 and 3, is proposed in {{cose-table}}

**Registry Name:**
COSE Algorithms

**Reference:**
draft-ietf-lake-authkem-edhoc-00

The columns of the registry are Name, Value and Description, where Value is an integer and the other columns are text strings. For both new registrations, the change controller is the IETF, and the reference field should point to this document. The new values proposed are:

**Table: COSE Algorithms**

| Name        | Value           | Description                               |
| ----------- | --------------- | ----------------------------------------- |
| ML-KEM-512  | -54 (suggested) | CBOR object KEM Algorithm for ML-KEM-512  |
| ML-KEM-1024 | -55 (suggested) | CBOR object KEM Algorithm for ML-KEM-1024 |
{: #cose-table}

## EDHOC Cipher Suites Registry {#edhoc-cipher-suites-registry}

The "EDHOC Cipher Suites" Registry from group "Ephemeral Diffie-Hellman Over COSE (EDHOC)" SHOULD be extended with new values to include the cipher suits with the KEM algorithm used. While the KEM-based authentication protocol specified in this document can support different KEM algorithms, the NIST-standardized ML-KEM is RECOMMENDED. The extension of values from the "Standards Action with Expert Review" range, when the ML-KEM algorithm is used at NIST security levels 1 and 3, is proposed in {{cipher-table}}

**Registry Name:**
EDHOC Cipher Suites

**Reference:**
draft-ietf-lake-authkem-edhoc-00

The columns of the registry are Value, Array, Description, and Reference, where Value is an integer and the other columns are text strings. The new values proposed are:

**Table: EDHOC Cipher Suites**

| Value         | Array                           | Description                                                                        | Reference |
| ------------- | ------------------------------- | ---------------------------------------------------------------------------------- | --------- |
| 7 (suggested) | 30, -16, 16, -54, -48 , 10, -16 | AES-CCM-16-128-128, SHA-256, 16, ML-KEM-512, ML-DSA-44, AES-CCM-16-64-128, SHA-256 | draft-ietf-lake-authkem-edhoc-00 |
| 8 (suggested) | 10, -16, 8, 1, -55, -49 , -16   | A256GCM, SHA-384, 16, ML-KEM-1024, ML-DSA-65, A256GCM, SHA-384                     | draft-ietf-lake-authkem-edhoc-00 |
{: #cipher-table}

The PQC ML-DSA signature algorithms are selected as the EDHOC signature algorithm parameter to verify X.509 certificate signatures when the X.509 credential type is used.

## EDHOC  Method Types Registry {#edhoc-method-types-registry}

The "EDHOC Method Types" Registry from group "Ephemeral Diffie-Hellman Over COSE (EDHOC)" SHOULD be extended with a new value that identifies the KEM-based authentication method. The extension value from the "Standards Action with Expert Review" range, is proposed in {{method-table}}

**Registry Name:**
EDHOC Method Types

**Reference:**
draft-ietf-lake-authkem-edhoc-00

The columns of the registry are Value, Initiator Authentication Key, Responder Authentication Key and Reference, where Value is an integer and the other columns are text strings. The new value proposed is:

**Table: EDHOC Method Types**

| Value         | Initiator Authentication Key | Responder Authentication Key | Reference                          |
| ------------- | ---------------------------- | ---------------------------- | ---------------------------------- |
| 5 (suggested) | Static KEM Key               | Static KEM Key               | 'draft-ietf-lake-authkem-edhoc-00' |
{: #method-table}

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
