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
  RFC9935:
  RFC8392:
  RFC9360:
  RFC8949:
  RFC8742:
  RFC5116:
  I-D.ietf-lake-pqsuites:
  I-D.ietf-jose-pqc-kem:

informative:
  RFC9794:
  RFC9053:
  RFC7252:
  RFC7959:
  RFC9177:
  I-D.uri-lake-pquake:
  I-D.celi-wiggers-tls-authkem:
  RFC9958:
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

  NIST-SP-800-227: DOI.10.6028/NIST.SP.800-227
...

--- abstract

This document specifies extensions to the Lightweight Authenticated Key Exchange (LAKE) protocol, formerly known as Ephemeral Diffie-Hellman over COSE (EDHOC), to provide resistance against quantum computer adversaries by incorporating Post-Quantum Cryptography (PQC) Key Encapsulation Mechanisms (KEMs) for both key exchange and authentication. It defines a new signature-free KEM-based authentication method in which both parties authenticate using KEMs, enabling quantum-resistant authentication without relying on digital signatures when PQC KEMs, such as the NIST-standardized ML-KEM, are used.


--- middle

# Introduction

The purpose of this document is to address the quantum-resistant transition of Lightweight Authenticated Key Exchange (LAKE) protocol, formerly known as Ephemeral Diffie-Hellman over COSE (EDHOC), by defining a new authentication method in which both parties use Key Encapsulation Mechanism (KEM)-based authentication. The method is independent of any specific KEM construction and, when instantiated with Post-Quantum Cryptography (PQC) KEM algorithms such as the NIST-standardized ML-KEM-512, enables signature-free, quantum-resistant authentication for LAKE.

KEMs are primarily key-establishment mechanisms that enable two parties to establish shared secret keying material over a public channel. However, KEMs can also be used in authenticated key-establishment schemes by using static KEM key pairs associated with the parties' identities or credentials. In such constructions, a ciphertext generated using the peer's static KEM public key can be decapsulated only using the corresponding static private key. As described in {{NIST-SP-800-227}}, authentication can then be based on key confirmation, which provides assurance that the peer possesses matching keying material and can serve as proof of possession of the corresponding private key. This specification applies this property to replace signature-based authentication with authentication based on static KEM key pairs.

## Motivation

The emerging Quantum Computing technologies bring new potential risks to the existing cryptographic infrastructures. Security mechanisms that rely on integer factorization or the discrete logarithm problem will be vulnerable to attacks by a Cryptographically Relevant Quantum Computer (CRQC). The European Commission recently issued a roadmap for the transition to Post-Quantum Cryptography (PQC), establishing a 2030 deadline for high-risk use cases and 2035 for medium-risk use cases, in alignment with the 2035 deadline set by the U.S. government for completing the transition to PQC in federal systems.

The U.S. National Institute of Standards and Technology (NIST) has concluded its PQC standardization process with the release of its first standardized PQC algorithms in three new Federal Information Processing Standards (FIPS): FIPS 203 (ML-KEM, based on CRYSTALS-Kyber), FIPS 204 (ML-DSA, based on CRYSTALS-Dilithium), and FIPS 205 (SLH-DSA, based on SPHINCS+). Additionally, FALCON has been selected for future standardization, and NIST has launched a new initiative to evaluate alternative PQC signature schemes with compact signatures and efficient verification speeds. Complementing these efforts, the Post-Quantum Use in Protocols (PQUIC) IETF Working Group (WG) is developing operational and design guidelines to support the transition. For example, {{RFC9794}} defines terminology for post-quantum/traditional Hybrid schemes, while ongoings draft such as {{RFC9958}} analyze the impact of CRQCs on existing systems and the challenges involved in transitioning to post-quantum algorithms.

The growing urgency to transition to PQC highlights the need to adapt LAKE, whose current security relies on traditional Elliptic-Curve Cryptography (ECC), based on the discrete logarithm problem that is known to be vulnerable to attacks by CRQCs. The integration of the PQC mechanism into LAKE raises important considerations around performance, as the protocol is explicitly designed for constrained environments where the number of handshake message rounds, network overhead, processing time, and power consumption are critical factors.

PQC algorithms generally have higher computational and memory costs compared to the classical cryptography algorithms they aim to replace because they often involve complex calculations and require larger byte sizes. Notably, the PQC digital signature schemes standardized by NIST, such as ML-DSA and SLH-DSA, use significantly large public keys and signatures, which can be difficult to transmit over constrained networks. It is important to note that while FALCON, also selected for standardization by NIST, provides much shorter signatures than the lattice-based schemes, its current implementations have been shown to be vulnerable to side-channel attacks. The new compact schemes under NIST evaluation should be more suitable for constrained environments. However, the current Cortex-M4 implementations of some of the most compact PQC signature schemes, like SNOVA, MAYO and OV-LP, still demand substantial memory resources, making them impractical for many constrained devices. Additionally, others, such as SQISign, have only recently been supported on such platforms, and performance benchmarks for their signature operations are still unavailable.

On the other hand, the standardized ML-KEM offers significantly higher computational efficiency compared to all other PQC KEMs (order of magnitude faster) and is at least three times more efficient than the fastest PQC signature schemes. Therefore, extending LAKE with a new authentication method that enables a signature-free KEM-based LAKE has the potential to reduce memory and processing requirements when ML-KEM is used. The approach can also result in lower network overhead compared to signature-based LAKE implementations that rely on standardized PQC signature-based algorithms.

Some standardization efforts propose adopting the KEM-based authentication mechanism to mitigate the overhead introduced by PQC digital signatures. For example, {{I-D.celi-wiggers-tls-authkem}} specifies a KEM-based authentication scheme for TLS 1.3, while {{I-D.uri-lake-pquake}} aims to define a general Post-Quantum Authentication Key exchange protocol, which based on the same approach.

This document describes a KEM-based authentication mechanism specifically for the LAKE protocol, introducing a new authentication method intended to provide a PQC signature-free variant as the static DH authentication method intends. The static-DH authentication of LAKE is based on the XX pattern of the Noise framework protocol {{Noise}}, where channel security guarantees are increasingly established by encrypting transmitted messages with keys derived from chains of shared secrets, as soon as those secrets become available. To align with this model, the KEM-based authentication method defined in this document follows the approach outlined in {{PQNoise-CCS22}}, which provides a recipe for transforming classical Noise patterns into PQ variants. This specification defines the necessary modifications to the LAKE protocol to support the PQ-Noise framework while preserving security properties comparable to those of the static-DH authentication method.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

## Key Encapsulation Mechanisms (KEMs) {#KEMs}

The Key Encapsulation Mechanism consists of 3 algorithms:

- **( pk, sk ) <- KEM.KeyGen( )**: The probabilistic key generation algorithm generates a KEM key pair consisting of a public encapsulation key ( pk ) and secret decapsulation key ( sk ).
- **( ss , ct ) <- KEM.Encapsulate( pk )**: The probabilistic encapsulation algorithm takes as input a public encapsulation key ( pk ) and produces a shared secret ( ss ) and ciphertext ( ct ).
- **( ss ) <- KEM.Decapsulate( ct, sk )**: The decapsulation algorithm takes as input a secret encacpsulation key ( sk ) and produce a shared secret ( ss ).

# Protocol Overview {#ProtoOverview}

This document defines a new authentication method for LAKE for general scenarios in which both parties authenticate using KEMs and may initially be mutually unknown. It aims to provide a free-signature authentication scheme as the static DH authentication LAKE method 3 does, which relies on the XX pattern from the Noise framework {{Noise}}, supporting mutual authentication and the transmission of encrypted public credentials. The proposed protocol adopts the approach provided by {{PQNoise-CCS22}} to transform the classical Noise XX pattern in LAKE into a PQ Noise XX variant. This results in a quantum-resistant, KEM-only version of LAKE when a PQC KEM is used.

The PQ translation of the Noise XX pattern requires introducing up to one additional round trip. With KEMs, the owner of the static key cannot combine their static private key with the ephemeral public key belonging to the other party to immediately prove their identity in the next message, as is possible with DH. Instead, the party must first receive from its peer a ciphertext encapsulated to its static public key before it can authenticate itself. This necessitates an additional key-confirmation message from the key owner, using the key derived from the encapsulated value.

The KEM-based LAKE protocol consists of five mandatory messages (message_1, message_2, message_3, message_4_KEM, and message_5_KEM), and an error message, between an Initiator (I) and a Responder (R). Error handling and cipher suite negotiation mechanisms are the same as defined in Section 6 of {{RFC9528}}. All LAKE messages are CBOR Sequences as specified in {{RFC9528}}. {{Exchange}} illustrates a KEM-based authentication LAKE message flow as well as the content of each message. The protocol elements in {{Exchange}} are introduced in this Section and in {{messages}}. Message formatting and processing are specified in {{messages}}.

{: #Exchange title="LAKE Message Flow using the KEM-based Authentication Method"}
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
|                  ct_R, AEAD( ID_CRED_I, EAD_3 )                   |
+------------------------------------------------------------------->
|                             message_3                             |
|                                                                   |
|                     ct_I, AEAD( MAC_2, EAD_4 )                    |
<-------------------------------------------------------------------+
|                         message_4_KEM                             |
|                                                                   |
|                         AEAD( MAC_3, EAD_5 )                      |
+------------------------------------------------------------------->
|                         message_5_KEM                             |
~~~


The parties exchange ephemeral and static KEM public keys, along with ciphertexts that encapsulate these keys, compute shared secrets and pseudorandom keys PRK, and derive symmetric session keys to encrypt message elements contained in intermediate handshake messages. All handshake messages include encrypted components protected with these derived session keys, offering varying levels of confidentiality and authenticity, except for the first message, which is sent in plaintext. The parties compute a shared secret session key, PRK_out, from which symmetric application keys are derived to protect application data. The Initiator derives these keys after receiving message_4_KEM, and the Responder after receiving message_5_KEM.

- pk_eph is the ephemeral KEM public key generated by the Initiator.
- ct_eph is the ephemeral ciphertext computed by the Responder with the KEM.encapsulation algorithm over the received ephemeral public key (pk_eph).
- ct_R is the ciphertext for Responder computed by the Initiator with the KEM.encapsulation algorithm over the static KEM public key of the Responder, retrieved from the received ID_CRED_R in message_2.
- ct_I is the ciphertext for Initiator computed by the Responder with the KEM.encapsulation algorithm over the static KEM public key of the Initiator, retrieved from the received ID_CRED_I in message_3.
- "CRED_I and CRED_R are the authentication credentials containing the public authentication keys of I and R, respectively", as defined in Section 2 of {{RFC9528}}.
- "ID_CRED_I and ID_CRED_R are used to identify and optionally transport the credentials of I and R, respectively", as defined in Section 2 of {{RFC9528}}.
- "Enc(), AEAD(), and MAC() denote encryption, Authenticated Encryption with Associated Data, and Message Authentication Code, crypto algorithms applied with keys derived from one or more shared secrets calculated during the protocol", as defined in Section 2 of {{RFC9528}}.
- "SUITES_I contains cipher suites supported by the Initiator and formatted and processed as specified in Section 3.6 and 6.3.2 of {{RFC9528}}".
- "METHOD is an integer specifying the authentication method",as defined in Section 3.2 of {{RFC9528}}. In this case method 5; see {{Method}}.
- C_I and C_R are Connection Identifiers chosen by the Initiator and Responder, respectively, as specified in Section 3.3 of {{RFC9528}}.
- EAD_1, EAD_2, EAD_3, EAD_4, EAD_5 are External Authorization Data included in message_1, message_2, message_3, message_4_KEM and message_5_KEM respectively.
- TH_2, TH_3, TH_4 and TH_5 are "transcript hashes (hashes of message data), used for key derivation and as additional authentication data", as conceptually defined in Section 2 of {{RFC9528}}. Their computation, however, is specified by the message flow defined in this document.

This protocol is designed so that it follows the provisions of {{RFC9528}}, that is, to encrypt and integrity protect as much information as possible and derive symmetric keys and random material using EDHOC_KDF with as much previous information as possible

## Protocol Elements {#ProtoElemnts}

This section describes the principal protocol elements that differ from the definitions of LAKE and highlights the most important similarities. For the missing elements, the definitions in Section 3 of {{RFC9528}} SHOULD be consulted.

### Ephemeral KEM {#EphemeralKEM}

The ephemeral KEM is used to provide forward secrecy. The Initiator generates a new ephemeral KEM key pair in every new session to ensure that the compromise of long-term keys does not compromise past communications. The elements of the Ephemeral KEM are:

- The ephemeral KEM key pair ( pk_eph, sk_eph ) is generated by the Initiator using the following function:

  ~~~
  pk_eph, sk_eph <- KEM.KeyGen()

  ~~~
- The ephemeral shared secret ( ss_eph ) and the ephemeral ciphertext ( ct_eph ) are generated using the encapsulation and decapsulation functions: in the Responder

  ~~~
  ss_eph, ct_eph <-  KEM.Encapsulate( pk_eph )

  ~~~

  in the Initiator

  ~~~
  ss_eph <-  KEM.decapsulation( ct_eph, sk_eph )

  ~~~

### Method {#Method}

The protocol extends LAKE with a new KEM-based authentication method, where both parties use static KEM key pairs. The authentication is provided by a Message Authentication Code (MAC) included in message_4_KEM and message_5_KEM to authenticate the Responder and Initiator, respectively.  This specification assumes that the Initiator and Responder have agreed in advance to use the speicif authentication method as defined in Section 3.2 of {{RFC9528}}. The selected method is then indicated by the Initiator in message_1.

{: #tab-method-types title="Authentication Keys for Method Types"}
| Method Type Value | Initiator Authentication Key | Responder Authentication Key |
| --- | --- | --- |
| 5 (suggested) | Static KEM Key | Static KEM Key |

### Authentication Parameters {#AuthneticationPara}

The protocol performs the same authentication-related operations as described in Section 3.5 of {{RFC9528}}.

The protocol transports information about credentials ID_CRED_R and ID_CRED_I in message_2 and message_3, respectively. The authentication of these credentials is verified through MAC_2 and MAC_3, sent by the Responder and the Initiator in message_4_KEM and message_5_KEM, respectively.

#### Authentication Keys {#Auth-keys}

Each party, the Initiator and the Responder, MUST hold its own static, long-term KEM key pair for authentication.

The authentication key algorithm must be compatible with the chosen method and selected cipher suite. The same KEM algorithm selected for the LAKE key exchange in the cipher suite MUST be used for both the ephemeral KEM key exchange and the authentication static KEM keys. The Initiator's and Responder's private and public authentication keys are denoted as follows:

- The Initiator static KEM authentication key pair: ( pk_I, sk_I )
- The Responder static KEM authentication key pair: ( pk_R, sk_R )

#### Authentication Credentials {#Auth-cred}

The authentication credentials, CRED_I and CRED_R, contain the authentication public key of the Initiator and Responder, respectively, as described in Section 3.5.2 of {{RFC9528}}.

- The authentication credentials can be X.509 certificates seconded as bstr, as defined in Section 3.5.2 of {{RFC9528}}, using {{RFC9360}}. {{RFC9935}} describes the conventions for using the ML-KEM in X.509 Public Key Infrastructure.
- Additionally, the authentication credential may include a COSE_key, formatted as specified in {{RFC8392}}, to reduce the credential size and avoid the PQC signature verification needed when X.509 certificates are used.The conventions for representing and using PQ-KEM keys with CBOR Object Signing and Encryption (COSE) are described in {{I-D.ietf-jose-pqc-kem}}.

#### Identification of Credentials {#Ident-cred}

The ID_CRED_R and the ID_CRED_I fields are fields are used to identify and optionally transport credentials as defined in Section 3.5.3 of {{RFC9528}}. The authentication method defined in this document operates within the general LAKE framework described in Section 3.5.3 of {{RFC9528}}, where ID_CRED_X can either contain the full CRED_X credentials or an identifier of those credentials if they have already been provided out-of-band.

- "ID_CRED_R is intended to facilitate for the Initiator retrieving the authentication credential CRED_R and the authentication key of R", as defined in Section 3.5.3 of {{RFC9528}}. For the authentication method defined in this document, the authentication key is the static KEM public key.
- "ID_CRED_I is intended to facilitate for the Responder retrieving the authentication credential CRED_I and the authentication key of I", as defined in Section 3.5.3 of {{RFC9528}}. For the authentication method defined in this document, the authentication key is the static KEM public key.

### Cipher Suites {#Ciphersuit}

The authentication method specified in this document uses the LAKE cipher suites element, as defined in Section 3.6 of {{RFC9528}}. An LAKE cipher suite consists of an ordered set of algorithms from the "COSE Algorithms" IANA registry {{RFC9053}}. The predefined quantum-resistant cipher suites for LAKE are defined in {{I-D.ietf-lake-pqsuites}},while the conventions for using PQ KEMs with COSE, including the corresponding algorithm registrations, are specified in {{I-D.ietf-jose-pqc-kem}}. The same KEM algorithm selected for key exchange SHOULD also be used for KEM-based authentication when method 5 is selected.

### Cipher Suite Negotiation and Error Handling {#Negotiation}

This specification relies on the error handling and cipher suite negotiation procedures defined in Section 6 of {{RFC9528}}. Among the defined error codes, error code 2 indicates a wrong selected cipher suite. In this case, the Responder returns SUITES_R, allowing the Initiator to select a supported cipher suite for the next protocol iteration.

### Transport {#Trasport}

The KEM-based authentication method for LAKE is not bound to any specific transport layer, similar to the classical LAKE methods defined in Section 3.4 of {{RFC9528}}. However, the resulting message sizes are expected to be larger than those of the original LAKE methods specified in {{RFC9528}}. This is because the currently standardized NIST KEM algorithms use comparatively large public keys and key encapsulation (ciphertext) sizes, thereby increasing the overall size of LAKE messages.

In highly constrained networks, larger message sizes MAY necessitate transport support for fragmentation. For example, if the network MTU is insufficient to carry a complete message, the messages can be transported over CoAP {{RFC7252}} using the Block-Wise Transfer mechanism to support fragmentation and reassembly, as specified in {{RFC7959}} or {{RFC9177}}. {{RFC7959}} defines the Block1 and Block2 options for request/response block-wise transfer in CoAP, while {{RFC9177}} extends this mechanism with the Q-Block1 and Q-Block2 options, allowing multiple blocks to be transmitted without waiting for per-block acknowledgments.

# Key Derivation {#key-derive}

This section highlights the differences and similarities in the key derivation process of the KEM-based authentication method compared to {{RFC9528}}. An overview of the LAKE key schedule when using the KEM-based authentication method is shown in {{key}}, and each key derivation step is explained in the following subsections.

{: #key title="LAKE Message Key Derivation using the KEM-based Authentication Method"}
~~~
       +-------+
       | TH_2  |
       +---+---+
           |
+----+  +--v-+  +------+  +------------+
|ss_e|->|Ext.|->|PRK_2e|--| EDHOC_KDF  |  +-----+  +-+  +---+
+----+  +----+  +--+---+  |L=0 ctx=TH_2|->|KEY_2|->| |->|C_2|
                   |      +------------+  +-----+  |X|  +---+
        +----------v-+               PLAINTEXT_2-->| |
        | EDHOC_KDF  |                             +-+
        |L=1 ctx=TH_3|
        +--+---------+
           |                                     PLAINTEXT_3
+----+  +--v-+  +--------+   +------------+           |
|ss_R|->|Ext.|->|PRK_3e2m|+->|EDHOC_KDF   |  +---+  +-+--+  +---+
+----+  +----+  +-+------+|  |L=3 ctx=TH_3|->|K_3|->|AEAD|->|C_3|
                  |       |  +------------+  +---+  +--+-+  +---+
        +---------v--+    | +-----------------+
        | EDHOC_KDF  |    | |EDHOC_KDF        |  +-----+
        |L=5 ctx=TH_4|    ->|L=2 ctx=context_2|->|MAC_2|
        +--+---------+      +-----------------+  +-----+
           |                                     PLAINTEXT_4
+----+  +--v-+  +--------+   +--------------+           |
|ss_I|->|Ext.|->|PRK_4e3m|+->|EDHOC_KDF     |  +---+  +-+--+  +---+
+----+  +----+  +--------+|  |L=8,9 ctx=TH_4|->|K_4|->|AEAD|->|C_4|
                          |  +--------------+  +---+  +----+  +---+
                          |  +-----------------+  +-----+
                          |  |EDHOC_KDF        |->|MAC_3|
                          |->|L=6 ctx=context_3|  +-----+
                          |  +-----------------+   PLAINTEXT_4
                          |  +----------------+           |
                          |  |EDHOC_KDF       |  +---+  +-+--+  +---+
                          |->|L=11,12 ctx=TH_5|->|K_5|->|AEAD|->|C_5|
                          |  +----------------+  +---+  +----+  +---+
                          |  +--------------+  +--------------+
                          |  |EDHOC_KDF     |  |EDHOC_KDF     |
                          |->|L=7 ctx=TH_5  |->|L=10 ctx=h'   |
                             +--------------+  +--------------+
                                                      |
                                                      v
                                               Aplication Key

~~~

## Keys for LAKE Message Processing

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
ss_eph <-  KEM.Decapsulate( ct_eph, sk_eph )

~~~

Responder:

~~~
ss_eph, ct_eph <-  KEM.Encapsulate( pk_eph )

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
ss_R, ct_R <-  KEM.Encapsulate( pk_R )

~~~

Responder:

~~~
ss_R <-  KEM.Decapsulate( ct_R, sk_R )

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
ss_I <-  KEM.Decapsulate( ct_I, sk_I )

~~~

Responder:

~~~
ss_I, ct_I <-  KEM.Encapsulate( pk_I )

~~~

### EDHOC_Expand and EDHOC_KDF {#edhoc_kdf}

The output key materials (OKMs) are derived from the PRKs in the same way as described in Section 4.1.2 of {{RFC9528}}, with modifications in the transcript hashes THs input contraction as specified in {{messages}}.

The same OKMs, including keys, initialization vectors (IV), and salts as those shows in Section 4.1.2 of {{RFC9528}} Figure 6 are derived. To facilitate compatibility with existing {{RFC9528}} implementations, the EDHOC_KDF values are unchanged. Consequently, the numbeical order of the labels does not necessarly correspond to the order in which the associated derivations are performed. The following additional changes with respect to {{RFC9528}} are noted:

- K_3 and IV_3 are computed to provide integrity protection and confidentiality for message_3 ensuring that the Initiator's identity is protected against active attacks. However, this does not provide authentication of the Initiator's identity.
- A distinct pair K_5 and IV_5 is derived to protect message_5_KEM. These values are different from K_4 and IV_4, which are used to protect message_4_KEM.
- SALT_3e2m and SALT_4e3m are derived using the latest available transcript hash at the time of their computation, which are TH_3 and TH_4, respectively.
- PRK_out is derived using TH_5, the latest available trascript hash, as the KDF context.
- The sequence encodings used for context_2 and context_3 are updated to use TH_4 and ED_4 and TH_5 and ED_5 respectively, as specified in Sections 5.4.2 and 5.5.2.
- The transcript hash inputs are updated for the new message formats; the corresponding transcript hash computations are specified in Section 5.

The final key derivations using EDHOC_KDF is shwon in {{key-derivations-kdf}}. Further details of the key derivation and how the output keying material is used are specified in {{messages}}

{: #key-derivations-kdf title="Key Derivations Using EDHOC_KDF for the KEM-based Authentication Methods"}
~~~
KEYSTREAM_2   = EDHOC_KDF( PRK_2e,   0, TH_2,      plaintext_length )
SALT_3e2m     = EDHOC_KDF( PRK_2e,   1, TH_2,      hash_length )
MAC_2         = EDHOC_KDF( PRK_3e2m, 2, context_2, mac_length_2 )
K_3           = EDHOC_KDF( PRK_3e2m, 3, TH_3,      key_length )
IV_3          = EDHOC_KDF( PRK_3e2m, 4, TH_3,      iv_length )
SALT_4e3m     = EDHOC_KDF( PRK_3e2m, 5, TH_4,      hash_length )
MAC_3         = EDHOC_KDF( PRK_4e3m, 6, context_3, mac_length_3 )
PRK_out       = EDHOC_KDF( PRK_4e3m, 7, TH_5,      hash_length )
K_4           = EDHOC_KDF( PRK_4e3m, 8, TH_4,      key_length )
IV_4          = EDHOC_KDF( PRK_4e3m, 9, TH_4,      iv_length )
K_5           = EDHOC_KDF( PRK_4e3m, 11, TH_5,     key_length )
IV_5          = EDHOC_KDF( PRK_4e3m, 12, TH_5,     iv_length )
PRK_exporter  = EDHOC_KDF( PRK_out,  10, h'',      ash_length )
~~~

### PRK_out {#PRK_out}

The pseudorandom key PRK_out is the output session key of a completed LAKE session and is derived as follows:

~~~
PRK_out = EDHOC_KDF( PRK_4e3m, TH_4, hash_length )

~~~

## Keys for LAKE Applications {#key_app}

Keying material for the application can be derived using the same EDHOC_Exporter interface defined in Section 4.2.1 of {{RFC9528}}.

# Message Formatting and Processing {#messages}

This section outlines the message format and the procedures for composing and processing each message.

## KEM-based Authentication LAKE Message 1 {#message1}

### Formatting of Message 1 {#fmessage1}

message_1 retains the same format as defined in Section 5.2.1 of {{RFC9528}}. The same fields are used, except that G_X is replaced by the KEM ephemeral public key ( pk_eph ) computed by the Initiator.

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

- Construct SUITES_I following the Section 5.2.2 of {{RFC9528}} specifications
- Generate an ephemeral KEM Key pair (pk_eph) using the KEM algorithm from the selected cipher suit. The ephemeral key pair is computed by the Initiator using the following function:

~~~
  pk_eph, sk_eph <-  KEM.KeyGen()

~~~

- Choose a connection identifier as in Section 5.2.2 of {{RFC9528}}.
- Encode message_1 as sequence of CBOR-encoded elements, as specified in {{fmessage1}}

### Responder Processing of Message 1 {#rpmessage1}

The Responder SHALL process message_1 in the following order:

1. "Decode message_1", as specified in Section 5.2.3 of {{RFC9528}}.
2. "Process message_1", as specify in Section 5.2.3 of {{RFC9528}}.
3. "If all processing is completed successfully, and if EAD_1 is present, then make it available to the application", as specified in Section 5.2.3 of {{RFC9528}}.

## KEM-based authentication LAKE Message 2 {#message2}

### Formatting of Message 2 {#fmessage2}

message_2 keeps the same formatting as Section 5.3.1 of {{RFC9528}}. The same fields are used instead G_Y is replaced with the ephemeral KEM ciphertext ( ct_eph ) computed on the Responder.

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
- Compute the transcript hash TH_2 = H(H(message_1),ct_eph) as specified in Section 5.3.2 of {{RFC9528}}.
- Compute the PRK_2e pseudorandom key from the ephemeral KEM shared secret ( ss_eph ).
- "Choose a connection identifier C_R", as specified in Section 5.3.2 of {{RFC9528}}.
- At this point, the Responder is not yet able to authenticate itself, so MAC_2 is not computed
- CIPHERTEXT_2 is calculated, with a binary additive stream cipher as in Section 5.3.2 of {{RFC9528}}, using a keystream (KEYSTREAM_2) generated with EDHOC_Expand and the following plaintext:
  - Compute PLAINTEXT_2 as:

    ~~~
       PLAINTEXT_2 = (C_R,ID_CRED_R,?EAD_2)

    ~~~

    where C_R, ID_CRED_R and EAD_2 elements corresponds with the ones in Section 5.3.2 of {{RFC9528}}.
  - Compute KEYSTREAM_2 as in {{edhoc_kdf}}
  - Compute CIPHERTEXT_2 as in Section 5.3.2 of {{RFC9528}},

    ~~~cddl
    CIPHERTEXT_2 = PLAINTEXT_2 XOR KEYSTREAM_2

    ~~~
- Encode message_2 as a sequence of CBOR-encoded data items as specified in {{fmessage2}}

### Initiator Processing of Message 2 {#ipmessage2}

The Initiator SHALL process message_2 in the following order:

1. Decode message_2
2. "Retrieve the protocol state" as proposed in Section 5.3.3 of {{RFC9528}}.
3. Compute the ephemeral KEM shared_secret ( ss_eph ) by decapsulating the KEM ciphertext ( ct_eph ) received in message_2 using the ephemeral secret key ( sk_eph ). The ephemeral KEM shared secret is computed by the Initiator using the following function:

    ss_eph <-  KEM.Decapsulate( ct_eph, sk_eph )

4. Compute the transcript hash TH_2 = H(H(message_1),ct_eph)
5. Compute the PRK_2e pseudorandom key from the ephemeral KEM shared secret ( ss_eph )
6. Derive KEYSTREAM_2 as in {{edhoc_kdf}}
7. Decrypt CIPHERTEXT_2; see {{rcmessage2}}
8. If all processing is completed successfully, ID_CRED_R and (if present) EAD_2 SHALL be made available to the application, as specified in Section 5.3.3 of {{RFC9528}}. In this specification, the application MUST authenticate and validate the credentials associated with ID_CRED_R at this point before proceeding. The Initiator's credentials are transmitted in the subsequent message and are encrypted under a key that can only be derived by a party possessing the private key corresponding to ID_CRED_R. Prior to sending its credentials, the Initiator MUST ensure that the credentials associated with ID_CRED_R have been successfully validated and accepted according to local policy. This prevents disclosure of the Initiator's credentials to a party presenting credentials that are cryptographically valid but untrusted or unintended.
9. Obtain the authentication credential (CRED_R) from the (ID_CRED_R) as in Section 5.3.3 of {{RFC9528}}, and the static authentication key of the Responder
10. Encapsulate the retrieved static KEM authentication key of the Responder ( pk_R ) calculating the corresponding ciphertext ( ct_R ) and shared secret ( ss_R ) with the following function:

    ss_R, ct_R <-  KEM.Encapsulate(pk_R)

11. Compute the new PRK_3e2m from a chain that includes both the ephemeral KEM shared secret ( ss_eph ) and the latest KEM shared secret for the Authentication of the Responder ( ss_R ), as defined in {{prk_3e2m}}

## KEM-based authentication LAKE Message 3 {#message3}

### Formatting of Message 3 {#fmessage3}

message_3 keeps the same formatting as the using in message_2 and in Section 5.3.1 of {{RFC9528}}

~~~ cddl
message_3 = (
  ct_R_CIPHERTEXT_3 : bstr,
)

~~~

### Initiator Composition of Message 3 {#icmessage3}

The Initiator SHALL process the composition of message_3 as follows:

- Compute the transcript hash TH_3=H(ct_R,TH_2,PLAINTEXT_2,CRED_R) as specified in Section 5.4.2 of {{RFC9528}}.
- Derive the new session key K_3/IV_3 as defined in {{edhoc_kdf}}. The Initiator can use this key to compute CIPHERTEXT_3, but it cannot be used to authenticate itself.
- At this point, the Responder is not jet able to authenticate itself, so MAC_3 is not computed.
- Compute a COSE_Encrypt0 object as defined in Section 5.2 and 5.3 of {{RFC9052}}, with the LAKE AEAD algorithm of the selected cipher suite, using the encryption key K_3, the initialization vector IV_3 (if used by the AEAD algorithm), the plaintext PLAINTEXT_3, and the following parameters as input:
  - protected = h''
  - external_aad = TH_3
  - K_3 and IV_3 are defined in {{edhoc_kdf}}
  - PLAINTEXT_3 = (C_I,ID_CRED_I,?EAD_3) where C_I, ID_CRED_I and EAD_3 elements corresponds with the ones in Section 5.3.3 of {{RFC9528}}.

  CIPHERTEXT_3 is the 'ciphertext' of COSE_Encrypt0.

- Encode message_3 as a sequence of CBOR-encoded data items as specified in  {{fmessage3}}.

### Responder Processing of Message 3 {#rpmessage3}

The Responder SHALL process message_3 in the following order:

1. Decode message_3
2. "Retrieve the protocol state", as defined in Section 5.4.3 of {{RFC9528}}.
3. Compute the KEM shared_secret ( ss_R ) for the authentication of the Responder by decapsulating the KEM ciphertext ( ct_R ) received in message_3 using the Responder static KEM secret key ( sk_R ). The KEM shared secret is computed by the Responder using the following function:

    ss_R <-  KEM.Decapsulate( ct_R, sk_R )
4. Compute the new PRK_3e2m from a chain that includes both the ephemeral KEM shared secret ( ss_eph ) and the latest KEM shared secret for the Authentication of the Responder ( ss_R ), as defined in {{prk_3e2m}}
5. Compute the transcript hash TH_3=H(TH_2,PLAINTEXT_2,CRED_R, ct_R)
6. Compute K_3/IV_3 as in {{edhoc_kdf}}, where plaintext_length is the length of PLAINTEXT_3
7. Decrypt CIPHERTEXT_3; see {{icmessage3}}
8. "If all processing is completed successfully, then make ID_CRED_I and (if present) EAD_2 available to the application", as in Section 5.3.4 of {{RFC9528}}.
9. "Obtain the authentication credential (CRED_I) from the (ID_CRED_I)" as in Section 5.3.4 of {{RFC9528}} and the static authentication key of the Initiator.

## KEM-based authentication LAKE Message 4 {#message4}

### Formatting of Message 4 {#fmessage4}

message_4_KEM keeps the same formatting as the using in message_2, message_3 and in Section 5.3.1 of {{RFC9528}}.

~~~ cddl
message_4_KEM = (
  ct_I_CIPHERTEXT_4 : bstr,
)

~~~

### Responder Composition of Message 4 {#rcmessage4}

The Responder SHALL process the composition of message_4_KEM as follows:

- Encapsulate the retrieved static KEM authentication key of the Initiator ( pk_I ) calculating the corresponding ciphertext ( ct_I ) and shared secret ( ss_I ) with the following function:

  ~~~cddl
   ss_I, ct_I <-  KEM.Encapsulate(pk_I)

  ~~~

- Compute the transcript hash TH_4 = H(TH_3, PLAINTEXT_3, CRED_I, ct_I)
- Compute MAC_2 as defined in {{edhoc_kdf}}, with context_2 =<< C_R, ID_CRED_R, TH_4, CRED_R, ? EAD_4 >>
  - The Responder authenticates with a PRK_3e2m derived from the KEM ephemeral shared secret and with the shared secret computed over its static KEM key.
  - The mac_length_2 is equal to the LAKE MAC length of the selected cipher suit.
  - The C_R, ID_CRED_R and CRED_R elements corresponds with the ones in Section 5.3.2 of {{RFC9528}}.
  - The latest transcript hash TH_4 and the External Application Data included in Message 4 (EAD_4) are used.
- Compute the new PRK_4e3m from a chain that includes the ephemeral KEM shared secret ( ss_eph ), the KEM shared secret for the Authentication of the Responder ( ss_R ) , and the latest KEM shared secret for the Authentication of the Initiator ( ss_I ) as defined in {{PRK_4e3m}}
- Derive the session key K_4/IV4 as in {{edhoc_kdf}}.
- Compute a COSE_Encrypt0 object as defined in Section 5.2 and 5.3 of {{RFC9052}}, with the LAKE AEAD algorithm of the selected cipher suite, using the encryption key K_4, the initialization vector IV_4 (if used by the AEAD algorithm), the plaintext PLAINTEXT_4, and the following parameters as input:
  - protected = h''
  - external_aad = TH_4
  - K_4 and IV_4 are defined in {{edhoc_kdf}}
  - PLAINTEXT_4 = ( MAC_2, ?EAD_4 )

  CIPHERTEXT_4 is the 'ciphertext' of COSE_Encrypt0.

- Compute the transcript hash TH_5 = H(TH_4, PLAINTEXT_4)
- Encode message_4 as a sequence of CBOR-encoded data items as specified in  {{fmessage4}}.

### Initiator Processing of Message 4 {#ipmessage4}

The Initiator SHALL process message_4_KEM in the following order:

1. Decode message_4_KEM
2. "Retrieve the protocol state using available message correlation", as in Section 3.4.2 of {{RFC9528}}.
3. Compute the KEM shared secret ( ss_I ) for the authentication of the Initiator by decapsulating the KEM ciphertext ( ct_I ) received in message_4_KEM using the Responder static KEM secret key ( sk_I ). The KEM shared secret is computed by the Initiator using the following function:

    ss_I <-  KEM.Decapsulate( ct_I, sk_I )

4. Compute the new PRK_4e3m from a chain that includes the ephemeral KEM shared secret ( ss_eph ), the KEM shared secret for the Authentication of the Responder ( ss_R ), and the latest KEM shared secret for the Authentication of the Initiator ( ss_I ) as defined in {{PRK_4e3m}}
5. Derive the session key K_4/IV4 as in {{edhoc_kdf}}.
6. Decrypt and verify the COSE_Encrypt0 (CIPHERTEXT_4) as defined {{RFC9052}}, Section 5.2 and 5.3, with the LAKE AEAD algorithm in the selected cipher suite and the parameters defined in {{rcmessage4}}.
7. Verify MAC_2 as defined in {{rcmessage4}}, and make the result of the verification available to the application.

## KEM-based authentication LAKE Message 5 {#message5}

### Formatting of Message 5 {#fmessage5}

message_5_KEM SHALL be a CBOR Sequence as defined below:

~~~ cddl
message_3 = (
  CIPHERTEXT_5 : bstr,
)
~~~

### Initiator Composition of Message 5 {#icmessage5}

The Initiator SHALL process the composition of message_5_KEM as follows:

- Compute the transcript hash TH_5 = H(TH_4, PLAINTEXT_4)
- Compute MAC_3 as defined in {{edhoc_kdf}}, with context_3 =<< C_I, ID_CRED_I, TH_5, CRED_I, ? EAD_5 >>
  - The Initiator authenticates with a PRK_4e3m derived from the three shared secrets, including the shared secret computed over its static KEM key ( ss_I ).
  - The mac_length_3 is equal to the LAKE MAC length of the selected cipher suit.
  - The C_I, ID_CRED_I and CRED_I elements corresponds with the ones in Section 5.4.2 of {{RFC9528}}.
  - The latest transcript hash TH_5 and the External Application Data included on Message 5 (EAD_5) are used.
- Compute a COSE_Encrypt0 object as defined in Section 5.2 and 5.3 of {{RFC9052}}, with the LAKE AEAD algorithm of the selected cipher suite, using the encryption key K_5, the initialization vector IV_5 (if used by the AEAD algorithm), the plaintext PLAINTEXT_5, and the following parameters as input:
  - protected = h''
  - external_aad = TH_5
  - K_5 and IV_5 are defined in {{edhoc_kdf}}
  - PLAINTEXT_5 = ( MAC_3, ? EAD_5 )

  CIPHERTEXT_5 is the 'ciphertext' of COSE_Encrypt0.

- Calculate PRK_out as defined in {{PRK_out}}. The Initiator can now derive application keys using the EDHOC_Exporter interface; see {{key_app}}
- Encode message_5_KEM as a CBOR data item as specified in {{fmessage5}}
- "Make the connection identifiers (C_I and C_R) and the application algorithms in the selected cipher suite available to the application" as in Section 5.4.2 of {{RFC9528}}.

After creating message_5_KEM, the Initiator can compute PRK_out and derive application keys using the EDHOC_Exporter interface. The Initiator SHOULD now persistently store PRK_out or application keys and send protected application data, since it has already verified message_4_KEM, which is protected with a derived application key by the Responder, and the application has authenticated the Responder.

### Responder Processing of Message 5 {#rpmessage5}

The Responder SHALL process message_5_KEM in the following order:

1. Decode message_5_KEM
2. "Retrieve the protocol state using available message correlation" as in Section 3.4.2 of {{RFC9528}}.
3. Decrypt and verify the COSE_Encrypt0 (CIPHERTEXT_5) as defined in Section 5.2 and 5.3 of {{RFC9052}}, with the LAKE AEAD algorithm in the selected cipher suite and the parameters defined in {{icmessage5}}.
4. Verify MAC_3 as defined in {{icmessage5}}, and make the result of the verification available to the application.
5. Calculate PRK_out as defined in {{PRK_out}}. The Initiator can now derive application keys using the EDHOC_Exporter interface; see {{key_app}}

After verifying message_5_KEM, the Responder can compute PRK_out and derive application keys using the EDHOC_Exporter interface. The Responder SHOULD now persistently store PRK_out or application keys and send protected application data, since it has already verified message_5_KEM, which is protected with a derived application key by the Initiator, and the application has authenticated the Initiator.


# Security Considerations {#Security}

## Security Properties {#SecurityP}

LAKE protocol with static DH keys enables the Initiator and Responder to generate an ephemeral-static shared secret using the other party's ephemeral public keys and their own credentials. This shared secret is then used to derive a session key for authentication. Messages 2 and 3 provide explicit authentication through MACs, which also bind the exchanged credentials to prevent misbinding attacks, as is described in Section 9.1 of {{RFC9528}}.

In contrast, the KEM-based authentication mechanism requires an initial action from the other party. The Responder must first receive ct_R, generated by the Initiator using the Responder's static public key, and decapsulate it to obtain ss_R before it can authenticate itself. To perform this encapsulation, the Initiator must retrieve the static KEM public key of the Responder from the ID_CRED_R sent in Message 2. As a result, the Responder cannot authenticate itself until Message 3 is processed, which contains the ct_R ciphertext necessary to derive the ss_R shared secret. Until then, it cannot generate MAC_2 or authenticate itself. Similarly, the Initiator cannot generate MAC_3 or authenticate itself before sending Message 3. This highlights the main challenge in integrating KEM-based authentication method within the LAKE handshake.

To address this issue and maintain the same level of identity protection than LAKE, against active attacks on the Initiator and passive attacks on the Responder, the credentials continue to be encrypted in Messages 2 and 3. In message_2, the Responder's credentials are included in a plaintext that is XORed with a key derived from the ephemeral shared secrets, as defined in Section 5.3 of {{RFC9528}}. By employing the same construction, this specification provides equivalent identity protection for the Responder against passive attackers. The credentials of the Initiator ( ID_CRED_I ) are encrypted using an AEAD algorithm to provide integrity protection and confidentiality, but not authentication, because the Initiator's shared secret is not yet available to prove its identity. The encryption key is derived from a combination of both ephemeral KEM shared secret (ss_eph) and the Responder static KEM shared secret ( ss_R ), used to authenticate the Responder. At this stage in the protocol, the specific encryption provided a form of weak forward secrecy, as the Initiator has not yet verify the static KEM public key of the Responder. However, the Initiator's credentials are still protected against active attacks, as only the legitimate Responder, who possesses the corresponding private key ( sk_R ) is capable of deriving the session key and decrypting message_3.

Furthermore, the protocol is extended with two additional messages (Messages 4 and 5) to enable both parties to:

- Prove possession of the final session key, ensuring key confirmation to the other party
- Ensure mutual authentication by explicitly authenticating themselves using the final session key, which incorporates all three shared secrets: the ephemeral KEM shared secret ( ss_eph ) and the KEM shared secrets ss_I and ss_R used to authenticate the Initiator and Responder, respectively.
- Provide credential binding by including MAC_2 and MAC_3, ensuring the integrity and authenticity of the credentials exchanged in messages 2 and 3.

In {{RFC9528}}, the transcript hashes (THs) are constructed as an accumulative hash, combining previous TH values with the current plain-text message. Each new plain-text message in the handshake is concatenated with the previous TH value, and the resulting hash forms the new TH. This process links each message in the sequence to all prior messages, creating a verifiable and continuous chain. As a result, any changes to the message content are detected during subsequent integrity verification using the transcript hashes. The KEM-based authentication method described in this document extends this approach. Both parties only need to verify the integrity and authenticity of the latest TH_4 and TH_5, which encompass all previous messages in the handshake. To facilitate this, the MAC-protected data in Messages 4 and 5 is modified to include TH_4 and TH_5 respectively. At the end of the handshake, both the Initiator and Responder can verify the integrity and authenticity of the entire handshake by checking the received MACs.

The payload security properties for the static DH authentication method and the KEM-based authentication method differ during the handshake. Unlike the static DH authentication method, the KEM-based method exhibits no authentication until the final two messages. It provides the same level of destination confidentiality for the first two and the last two messages, while message_3 offers weaker forward secrecy. The Initiator's credentials are encrypted within message_3 using a key derived from the Responder's static public key and the ephemeral key, ensuring that only the intended Responder can decrypt the credential, and protect them against active attacks.

Strong forward secrecy is achieved once the KEM-based method handshake is completed, similar to the static-DH method handshake (described in Section 9.1 of {{RFC9528}}). The final session key is derived from ss_eph, ss_I, and ss_R, combining a fresh ephemeral KEM contribution with shared secrets established using the Initiator's and Responder's static KEM public keys. Provided that the ephemeral private key material remains uncompromised and is erased after use, subsequent compromise of the parties' long-term static KEM private keys does not enable recovery of past session keys. Furthermore, successful decapsulation demonstrates possession of the corresponding static KEM private key and thus provides implicit authentication of the parties. Explicit mutual authentication is then provided through verification of MAC_2 and MAC_3, respectively.

K_4, IV_4, K_5, and IV_5, used for the symetrical ecrypted part of message_4_KEM and message_5_KEM, are also derived from key material that depends on all three shared secrets. Consequently, message_4_KEM and message_5_KEM are protected by both the fresh ephemeral contribution and the static KEM contributions associated with the intended parties. Successful processing of these messages confirms possession of the key material required to derive the corresponding protection keys. Therefore, assuming that the ephemeral private key material remains uncompromised and is erased after use,the protected parts of previously recorded message_4_KEM and message_5_KEM remain confidential even after subsequent compromise of the long-term static KEM private keys.

The authentication method defined in this document is intended to retain resistance to classical Key-Compromise Impersonation (KCI) attacks. In particular, compromise of one party's static KEM private key alone should not enable an attacker to impersonate the peer to that party. Fresh KEM encapsulations using the static KEM public keys MUST be generated for each protocol session. ss_I, ss_R, and their corresponding ciphertexts MUST NOT be reused across sessions. Consequently, disclosure of values from one session does not by itself enable impersonation in subsequent sessions. Within a given session, both the authentication keys used to compute and verify MAC_2 and MAC_3 and the subsequently derived session keying material providing implicit authentication are derived through the LAKE key schedule from a combination of ss_eph, ss_I, and ss_R; therefore, disclosure of ss_I or ss_R, alone should not be sufficient to impersonate a peer.

The KEM-based authentication method also differs from static-DH LAKE with respect to leakage of ephemeral key material. In static-DH authentication, the authentication secret is derived using the local ephemeral private key and the peer's static public key; disclosure of the ephemeral private key therefore allows that secret to be recomputed from otherwise public information. By contrast, in the KEM-based construction, disclosure of the ephemeral KEM secret does not allow the static KEM-derived authentication shared secrets to be recomputed, since each depends only on the corresponding static KEM private key. The authentication and session keying material is subsequently derived through the LAKE key schedule from the combination of ss_eph, ss_I, and ss_R. Consequently, leakage of the ephemeral KEM secret alone is not sufficient to derive the complete authentication keying material or to impersonate the peer.

A potential misbinding attack will not be detected until the handshake concludes, specifically when the Initiator verifies Message 4 and the Responder verifies Message 5. Therefore, EAD data should be treated as unprotected, and keying materials should not be persistently stored until the protocol is complete, as with the static-DH method (described in Section 9.1 of {{RFC9528}}). The final Application Session Key should only be derived at the end of the handshake, after ensuring mutual authentication, message handshake integrity, credentials authenticity, and proof of key possession.

The KEM-based authentication method does not provide non-repudiation, but only implicit proof of participation, similar to LAKE with static DH keys. It also maintains an equivalent level of downgrade protection, as the negotiation base of the protocol is unchanged.

## KEM Security Considerations {#KEMsec}

{{KEMBinding-CCS24}} demonstrates that IND-CCA2 security alone does not preclude re-encapsulation attacks in KEM-based key exchange protocols. Such attacks can lead to unknown key-share conditions, in which two honest parties derive the same shared secret while associating it with different peer identities. Therefore, any KEM used in this specification MUST achieve IND-CCA2 security and MUST ensure that the derived shared secret is cryptographically bound to the recipient's public key. This requirement prevents re-encapsulation and related key-substitution attacks.

Fresh KEM encapsulations using the static KEM public keys MUST be generated for each protocol session. ss_I, ss_R, and their corresponding ciphertexts MUST NOT be reused across sessions. Consequently, disclosure of values from one protocol session does not by itself enable impersonation in subsequent sessions.


# IANA Considerations {#IANA}

## LAKE  Method Types Registry {#edhoc-method-types-registry}

The "EDHOC Method Types" Registry from group "Ephemeral Diffie-Hellman Over COSE (EDHOC)" SHOULD be extended with a new value that identifies the KEM-based authentication method. The extension value from the "Standards Action with Expert Review" range, is proposed in {{method-table}}

Registry Name: EDHOC Method Types

Reference: draft-ietf-lake-authkem-edhoc

The columns of the registry are Value, Initiator Authentication Key, Responder Authentication Key and Reference, where Value is an integer and the other columns are text strings. The new value proposed is:


| Value         | Initiator Authentication Key | Responder Authentication Key | Reference                       |
| ------------- | ---------------------------- | ---------------------------- | ------------------------------- |
| 5 (suggested) | Static KEM Key               | Static KEM Key               | 'draft-ietf-lake-authkem-edhoc' |
{: #method-table title="EDHOC Method Types"}

--- back

