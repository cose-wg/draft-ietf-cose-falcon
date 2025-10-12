---
title: "FN-DSA for JOSE and COSE"
category: std

docname: draft-ietf-cose-falcon-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "CBOR Object Signing and Encryption"
keyword:
 - JOSE
 - COSE
 - PQC
 - FN-DSA

venue:
  group: "CBOR Object Signing and Encryption"
  type: "Working Group"
  mail: "cose@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/cose/"
  github: "cose-wg/draft-ietf-cose-falcon"
  latest: "https://cose-wg.github.io/draft-ietf-cose-falcon/draft-ietf-cose-falcon.html"

author:
 -
    fullname: "Michael Prorock"
    organization: mesur.io
    email: "mprorock@mesur.io"
 -
    fullname: "Orie Steele"
    organization: Tradeverifyd
    email: "orie@or13.io"
 -
    fullname: "Hannes Tschofenig"
    organization: University of Applied Sciences Bonn-Rhein-Sieg
    abbrev: "H-BRS"
    country: Germany
    email: "hannes.tschofenig@gmx.net"

contributor:
 -
    fullname: "Rafael Misoczki"
    organization: Google
    email: "rafaelmisoczki@google.com"
 -
    fullname: "Michael Osborne"
    organization: IBM
    email: "osb@zurich.ibm.com"
 -
    fullname: "Christine Cloostermans"
    organization: NXP
    email: "christine.cloostermans@nxp.com"

normative:
  RFC7515:
  RFC7517:
  RFC9053:
  RFC9054:
  RFC7518:
  I-D.draft-ietf-cose-dilithium: ML-DSA
  USNIST.FIPS.206:
    title: "Fast Fourier Transform over NTRU-Lattice-Based Digital Signature Algorithm"
    target: https://www.nist.gov/news-events/news/2024/08/nist-releases-first-3-finalized-post-quantum-encryption-standards

informative:
  IANA.jose: IANA.jose
  IANA.cose: IANA.cose
  I-D.draft-ietf-pquip-pqc-engineers:
  USNIST.FIPS.204:
    title: "Module-Lattice-Based Digital Signature Standard"
    target: https://doi.org/10.6028/NIST.FIPS.204
  USNIST.FIPS.205:
    title: "Stateless Hash-Based Digital Signature Standard"
    target: https://doi.org/10.6028/NIST.FIPS.205
  GPV08:
    title: "Trapdoors for Hard Lattices and New Cryptographic Constructions"
    author:
      - ins: C. Gentry
        name: Craig Gentry
      - ins: C. Peikert
        name: Chris Peikert
      - ins: V. Vaikuntanathan
        name: Vinod Vaikuntanathan
    date: 2008
    seriesinfo: "Proceedings of the 40th Annual ACM Symposium on Theory of Computing (STOC '08), pp. 197–206"
    target: https://doi.org/10.1145/1374376.1374407
    doi: 10.1145/1374376.1374407
    organization: "Association for Computing Machinery (ACM)"
    address: "New York, NY, USA"
  DP16:
    title: "Fast Fourier Orthogonalization"
    author:
      - ins: L. Ducas
        name: Léo Ducas
      - ins: T. Prest
        name: Thomas Prest
    date: 2016
    seriesinfo: "Proceedings of the 2016 ACM International Symposium on Symbolic and Algebraic Computation (ISSAC '16), pp. 191–198"
    target: https://doi.org/10.1145/2930889.2930923
    doi: 10.1145/2930889.2930923
    organization: "Association for Computing Machinery (ACM)"
    address: "New York, NY, USA"

--- abstract

This document specifies JSON Object Signing and Encryption (JOSE) and CBOR Object Signing and Encryption (COSE) serializations for FFT (fast-Fourier transform) over NTRU-Lattice-Based Digital Signature Algorithm (FN-DSA), a Post-Quantum Cryptography (PQC) digital signature scheme defined in US NIST FIPS 206 (expected to be published in late 2026 early 2027).

--- middle

# Introduction

This document specifies JSON Object Signing and Encryption (JOSE) and CBOR Object Signing and Encryption (COSE) serializations for FFT (fast-Fourier transform) over NTRU-Lattice-Based Digital Signature Algorithm (FN-DSA), a Post-Quantum Cryptography (PQC) digital signature scheme defined in US NIST FIPS 206 (expected to be published in late 2026 early 2027).

FN-DSA (formerly known as Falcon) is a lattice-based digital signature scheme based on the GPV hash-and-sign framework {{GPV08}}, instantiated over NTRU lattices with fast Fourier sampling techniques {{DP16}}. The core hard problem underlying FN-DSA is the SIS (Short Integer Solution) problem over NTRU lattices.

FN-DSA (formerly known as Falcon) is a digital signature algorithm based on lattice mathematics.
It follows the hash-and-sign design introduced by Gentry, Peikert, and Vaikuntanathan {{GPV08}}.
FN-DSA operates on NTRU lattices and uses fast Fourier techniques {{DP16}} to make signature generation compact and efficient.
The security of the scheme relies on the hardness of solving certain lattice problems, in particular the Short Integer Solution (SIS) problem.

FN-DSA offers:

- Post-quantum security under the assumption that NTRU-SIS remains hard.
- Compactness in key and signature size.  
- Efficient operations (roughly O(n log n)).
- A requirement for careful implementation to avoid side-channel leakage (notably Gaussian sampling must be constant-time where applicable).

The sizes of public key, private key, and signature for the parameter sets are the same as in the original Falcon specification:

| Parameter Set | Signature size (bytes) | Public Key size (bytes) | Private Key size (bytes) |
|---------------|-------------------------|---------------------------|----------------------------|
| FN-DSA-512    | 666                     | 897                       | 1281                       |
| FN-DSA-1024   | 1280                    | 1793                      | 2305                       |

For a detailed comparison of FN-DSA with ML-DSA {{USNIST.FIPS.204}} and SLH-DSA {{USNIST.FIPS.205}} see {{Section 11.3 of I-D.draft-ietf-pquip-pqc-engineers}}.

This document defines how FN-DSA is used with JSON Object Signing and Encryption (JOSE) {{RFC7515}} and CBOR Object Signing and Encryption (COSE) {{RFC8812}}.

# Terminology

{::boilerplate bcp14-tagged}

# The FN-DSA Algorithm Family

The FN-DSA Signature Scheme is parameterized to support different security levels.

This document introduces the registration of the following algorithms in {{-IANA.jose}}:

| Name       | alg | Description |
|-------------|------|-------------|
| FALCON512  | FALCON512     | Falcon with parameter set 512 |
| FALCON1024  | FALCON1024     | Falcon with parameter set 1024 |
{: #jose-algorithms align="left" title="JOSE Algorithms for FN-DSA"}

This document introduces the registration of the following algorithms in {{-IANA.cose}}:

| Name       | alg | Description |
|-------------|------|-------------|
| FALCON512  | TBD1 (-54) | CBOR Object Signing Algorithm for FALCON512 |
| FALCON1024  | TBD2 (-55) | CBOR Object Signing Algorithm for FALCON1024 |
{: #cose-algorithms align="left" title="COSE Algorithms for FN-DSA"}

# FN-DSA Keys

The FN-DSA Algorithm Family uses the Algorithm Key Pair (AKP) key type, as defined in {{-ML-DSA}}.

The specific algorithms for FN-DSA, such as FALCON512 and FALCON1024, are defined in this document and are used in the `alg` value of an AKP key representation to specify the corresponding algorithm.

Thumbprints for FN-DSA keys are computed according to the process described in {{-ML-DSA}}.

# Security Considerations

The security considerations of {{RFC7515}}, {{RFC7517}} and {{RFC9053}} apply to this specification as well.

A detailed security analysis of FN-DSA is beyond the scope of this specification; see {{USNIST.FIPS.206}} for additional details.

## Validating Public Keys

TODO

## Side-Channel Attacks

TODO

## Randomness

TODO

# IANA Considerations

## New COSE Algorithms

IANA is requested to add the following entries to the COSE Algorithms Registry.
The following completed registration templates are provided as described in {{RFC9053}} and {{RFC9054}}.

### FALCON512

* Name: FALCON512
* Value: TBD1 (requested assignment -54)
* Description: CBOR Object Signing Algorithm for FALCON512
* Capabilities: `[kty]`
* Change Controller: IETF
* Reference: RFC XXXX
* Recommended: Yes

### FALCON1024

* Name: FALCON1024
* Value: TBD2 (requested assignment -55)
* Description: CBOR Object Signing Algorithm for FALCON1024
* Capabilities: `[kty]`
* Change Controller: IETF
* Reference: RFC XXXX
* Recommended: Yes

## New JOSE Algorithms

IANA is requested to add the following entries to the JSON Web Signature and Encryption Algorithms Registry.
The following completed registration templates are provided as described in {{RFC7518}}.

### FALCON512

* Algorithm Name: FALCON512
* Algorithm Description: FALCON512 as described in US NIST FIPS 206.
* Algorithm Usage Location(s): alg
* JOSE Implementation Requirements: Optional
* Change Controller: IETF
* Specification Document(s): RFC XXXX
* Algorithm Analysis Documents(s): {{USNIST.FIPS.206}}

### FALCON1024

* Algorithm Name: FALCON1024
* Algorithm Description: FALCON1024 as described in US NIST FIPS 206.
* Algorithm Usage Location(s): alg
* JOSE Implementation Requirements: Optional
* Change Controller: IETF
* Specification Document(s): RFC XXXX
* Algorithm Analysis Documents(s): {{USNIST.FIPS.206}}

--- back

# Examples

## JOSE

### Key Pair

~~~json
{
  "kty": "AKP",
  "alg": "FALCON512",
  "pub": "V53SIdVF...uvw2nuCQ",
  "priv": "V53SIdVF...cDKLbsBY"
}
~~~
{: #FALCON512-private-jwk title="Example FALCON512 Private JSON Web Key"}

~~~json
{
  "kty": "AKP",
  "alg": "FALCON512",
  "pub": "V53SIdVF...uvw2nuCQ"
}
~~~
{: #FALCON512-public-jwk title="Example FALCON512 Public JSON Web Key"}


### JSON Web Signature

~~~
{
  "kid: "clpwZ...RWYU9CUF",
  "alg": "FALCON512",
  "typ": "JWT"
}
~~~
{: #FALCON512-jose-jws title="Example FALCON512 Decoded Protected Header for a JSON Web Signature"}

## COSE

### Key Pair

~~~~ cbor-diag
{
  / kty AKP       / 1: 7,
  / alg FALCON512 / 3: -54,
  / public key    / -1: h'7803c0f9...3f6e2c70',
  / private key   / -2: h'7803c0f9...3bba7abd'
}
~~~~
{: #FALCON512-private-cose-key title="Example FALCON512 Private COSE Key"}

~~~~ cbor-diag
{
  / kty AKP       / 1: 7,
  / alg FALCON512 / 3: -54,
  / public key    / -1: h'7803c0f9...3f6e2c70',
}
~~~~
{: #FALCON512-public-cose-key title="Example FALCON512 Public COSE Key"}

### COSE Sign1

~~~~ cbor-diag
18([
  <<{
    / alg FALCON512 / 1: -54,
  }>>,
  / unprotected / {},
  / payload / h'66616b65',
  / signature / h'53e855e8...0f263549'
])
~~~~
{: #FALCON512-cose-sign-1-diagnostic title="Example FALCON512 COSE Sign1"}



# Document History

-02

  *  Converted to markdown
  *  Applied feedback from IESG Evaluation on ML-DSA
  *  Revised references
  *  Revised abstract

-01

  *  Added Acknowledgements
  *  Added Document History
  *  Updated test vectors


# Acknowledgments
{:numbered="false"}

We would like to especially thank David Balenson for careful review of approaches taken in this document. We would also like to thank Michael B. Jones for guidance in authoring.


