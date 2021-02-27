---
title: "OpenSSL Pocket Reference"
toc: true
toc_label: "Contents"
tags:
  - OpenSSL
  - Security
  - Cryptography

date: February 27, 2021
layout: single
classes: wide
excerpt: "OpenSSL Pocket Reference - using the most common commands."
---

## Introduction
As per the Github [page](https://github.com/openssl/openssl) of the OpenSSL library, the `openssl` command line tool can be used for:
- creation of keys and key parameters
- creation of X.509 certificates, Certificate Signing Requests (CSRs) and Certificate Revocation Lists (CRLs)
- calculation of message digests
- encryption and decryption with the supported ciphers.
- SSL/TLS client and server tests
- many others...

The OpenSSL package can also be used as a C library and linked against.

## Glossary of terms used later in this article
- **SSL/TLS: Secure Sockets Layer / Transport Layer Security**: They are cryptographic protocols designed to provide communications security over a computer network. TLS replaced SSL in recent years.
- **Hash Function**: Also called CHF (Cryptographic Hash Function) or Message Digests. It maps data of arbitrary size to a bit array of a fixed size (the resulting hash value). It is a one-way function and it must be impossible to invert it. Examples: MD5, SHA1, Whirlpool. It protects only the message's **Integrity**.
- **MAC: Message Authentication Code**: Also known as a tag or a keyed hash, it is used to authenticate a message (certify that the message comes from the stated sender).  It hashes the data + a shared secret. It provides **Integrity and Authenticity** (but does not provide Non-repudiation).
- **HMAC: Hash-based Message Authentication Code (MAC)**: It is a method of turning a Hashing algorithm like MD5 or SHA256 into a MAC, calling them HMAC-MD5 or HMAC-SHA256 respectively.
- **Digital Signature**: It is created with a private key, and verified with the corresponding public key of an asymmetric key pair. Only the holder of the private key can create this signature, and normally anyone knowing the public key can verify it. So, it provides: **Integrity, Authenticity and Non-repudiation**.
- **CA: Certificate Authority**: It is an entity that issues digital certificates. A digital certificate certifies the ownership of a public key by the named subject of the certificate. This allows others to rely upon signatures or on assertions made about the private key that corresponds to the certified public key. A CA acts as a trusted third party—trusted both by the subject (owner) of the certificate and by the party relying upon the certificate.
- **PEM: Privacy-Enhanced Electronic Mail**: Base64 encoded DER certificate, enclosed between "-----BEGIN CERTIFICATE-----" and "-----END CERTIFICATE-----". PEM format can also be used to store keys.
- **X.509**: It is a standard defining the format of public key certificates. An X.509 certificate contains a public key and an identity (a hostname, or an organization, or an individual), and is either signed by a CA or self-signed.
- **CSR: Certificate Signing Request** - It is a block of encoded text that is given to a Certificate Authority (CA) when applying for an SSL Certificate. It is usually generated on the server where the certificate will be installed and contains information that will be included in the certificate such as the Organization Name, Domain Name, City, and Country.
- [**PKI: Public-Key Infrastructure**](https://en.wikipedia.org/wiki/Public_key_infrastructure): A public key infrastructure (PKI) is a set of roles, policies, hardware, software and procedures needed to create, manage, distribute, use, store and revoke digital certificates and manage public-key encryption. It binds public keys with respective identities of entities (like people and organizations).

## Obtaining basic information about OpenSSL

```bash
$ openssl version -a
OpenSSL 1.1.1f  31 Mar 2020
built on: Wed Feb 17 12:35:54 2021 UTC
platform: debian-amd64
options:  bn(64,64) rc4(16x,int) des(int) blowfish(ptr) 
compiler: gcc -fPIC -pthread -m64 -Wa,--noexecstack -Wall -Wa,--noexecstack -g -O2 -fdebug-prefix-map=/build/openssl-g4Ey0B/openssl-1.1.1f=. -fstack-protector-strong -Wformat -Werror=format-security -DOPENSSL_TLS_SECURITY_LEVEL=2 -DOPENSSL_USE_NODELETE -DL_ENDIAN -DOPENSSL_PIC -DOPENSSL_CPUID_OBJ -DOPENSSL_IA32_SSE2 -DOPENSSL_BN_ASM_MONT -DOPENSSL_BN_ASM_MONT5 -DOPENSSL_BN_ASM_GF2m -DSHA1_ASM -DSHA256_ASM -DSHA512_ASM -DKECCAK1600_ASM -DRC4_ASM -DMD5_ASM -DAESNI_ASM -DVPAES_ASM -DGHASH_ASM -DECP_NISTZ256_ASM -DX25519_ASM -DPOLY1305_ASM -DNDEBUG -Wdate-time -D_FORTIFY_SOURCE=2
OPENSSLDIR: "/usr/lib/ssl"
ENGINESDIR: "/usr/lib/x86_64-linux-gnu/engines-1.1"
Seeding source: os-specific
```
The most interesting configs here are the build date and the folder where OpenSSL keeps its configuration: OPENSSLDIR.

Getting the available commands:
```bash
$ openssl help
Standard commands
asn1parse         ca                ciphers           cms               
crl               crl2pkcs7         dgst              dhparam           
dsa               dsaparam          ec                ecparam           
enc               engine            errstr            gendsa            
genpkey           genrsa            help              list              
nseq              ocsp              passwd            pkcs12            
pkcs7             pkcs8             pkey              pkeyparam         
pkeyutl           prime             rand              rehash            
req               rsa               rsautl            s_client          
s_server          s_time            sess_id           smime             
speed             spkac             srp               storeutl          
ts                verify            version           x509              

Message Digest commands (see the 'dgst' command for more details)
blake2b512        blake2s256        gost              md4               
md5               rmd160            sha1              sha224            
sha256            sha3-224          sha3-256          sha3-384          
sha3-512          sha384            sha512            sha512-224        
sha512-256        shake128          shake256          sm3               

Cipher commands (see the 'enc' command for more details)
aes-128-cbc       aes-128-ecb       aes-192-cbc       aes-192-ecb       
aes-256-cbc       aes-256-ecb       aria-128-cbc      aria-128-cfb      
aria-128-cfb1     aria-128-cfb8     aria-128-ctr      aria-128-ecb      
aria-128-ofb      aria-192-cbc      aria-192-cfb      aria-192-cfb1     
aria-192-cfb8     aria-192-ctr      aria-192-ecb      aria-192-ofb      
aria-256-cbc      aria-256-cfb      aria-256-cfb1     aria-256-cfb8     
aria-256-ctr      aria-256-ecb      aria-256-ofb      base64            
bf                bf-cbc            bf-cfb            bf-ecb            
bf-ofb            camellia-128-cbc  camellia-128-ecb  camellia-192-cbc  
camellia-192-ecb  camellia-256-cbc  camellia-256-ecb  cast              
cast-cbc          cast5-cbc         cast5-cfb         cast5-ecb         
cast5-ofb         des               des-cbc           des-cfb           
des-ecb           des-ede           des-ede-cbc       des-ede-cfb       
des-ede-ofb       des-ede3          des-ede3-cbc      des-ede3-cfb      
des-ede3-ofb      des-ofb           des3              desx              
rc2               rc2-40-cbc        rc2-64-cbc        rc2-cbc           
rc2-cfb           rc2-ecb           rc2-ofb           rc4               
rc4-40            seed              seed-cbc          seed-cfb          
seed-ecb          seed-ofb          sm4-cbc           sm4-cfb           
sm4-ctr           sm4-ecb           sm4-ofb
# Your output may differ depending on the OpenSSL version.

# Getting available digests
$ openssl dgst -list
$ openssl dgst -help

# Getting available ciphers
$ openssl enc -list
$ openssl enc -help
```

## Generating pseudo-random numbers with OpenSSL
```bash
$ openssl rand -help
Valid options are:
 -help               Display this summary
 -out outfile        Output file
 -rand val           Load the file(s) into the random number generator
 -writerand outfile  Write random data to the specified file
 -base64             Base64 encode output
 -hex                Hex encode output
 -engine val         Use engine, possibly a hardware device

# Generate 32 bytes of random data in hex format (64 hex characters) - write to stdout
$ openssl rand -hex 32
e332d8bde28abd882a879f89824a315d31c8f8e6c440ed010c306ea4be82857b

# Generate 32 bytes of random data in base64 format - write to stdout
$ openssl rand -base64 32
g8mPm313yRS9Klwo73cKgDbjggS7m4/uTabddrFsufU=

$ openssl rand -base64 32 | base64 --decode
```

## Generating keys with OpenSSL
```bash
# Generate a 2048-bit RSA private key encoded with aes128 in PEM format
$ openssl genrsa -aes128 -out my.key 2048

# Output RSA key details
$ openssl rsa -text -in my.key

# Extract public key from the RSA key
$ openssl rsa -in fd.key -pubout -out fd-public.key

# Extract just the private key from the RSA key
$ openssl rsa -in fd.key -pubout -out fd-public.key

# Generate a 2048-bit DSA key encrypted with aes128
$ openssl dsaparam -genkey 2048 | openssl dsa -out dsa.key -aes128

# Generate ECDSA (Elliptic Curve DSA) key with a fixed key size (controlled by the chosen elliptic curve)
$ openssl ecparam -genkey -name secp256r1 | openssl ec -out ec.key -aes128

# Get list of ECs
$ openssl ecparam -list_curves
```

## Generating a CSR (Certificate Signing Request) with OpenSSL
This is a formal request asking a CA to sign a certificate, and it contains the public key of the entity requesting the certificate and some information about the entity. This data will all be part of the certificate. A CSR is always signed with the private key corresponding to the public key it carries.
```bash
$ openssl req -help
$ openssl genrsa -aes128 -out specflare.key 2048
$ openssl req -new -key specflare.key -out specflare.csr

# Check if the CSR is correct:
$ openssl req -text -in specflare.csr -noout
Certificate Request:
    Data:
        Version: 1 (0x0)
        Subject: C = RO, ST = Romania, L = Bucharest, O = Specflare, OU = Software, CN = specflare.com, emailAddress = liviu@specflare.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus: (... omitted bulk of data ...)
                Exponent: 65537 (0x10001)
        Attributes:
            a0:00
    Signature Algorithm: sha256WithRSAEncryption
         (... omitted bulk of data ...)
```

### Generating CSRs from existing certificates - for certificate renewal
If you’re renewing an old certificate and don’t want to make any changes to the information present in it, you can just do:
```bash
$ openssl x509 -x509toreq -in specflare.crt -out specflare.csr -signkey specflare.key
```
You can use either the old key or generate a new one and use it (better).

### Generating a self-signed certificate from your own CSR
If you don't want to pay for a CA-signed certificate, you can generate your own self-signed certificate. Please note that using it in your web server for instance, will warn your clients that your site uses a self-signed certificate.
```bash
# Generate a self-signed certificate valid for 3 years (1095 days) using given key and CSR
$ openssl x509 -req -days 1095 -in specflare.csr -signkey specflare.key -out specflare.crt

# Generating a self-signed certificate in a single step (CSR + Cert)
$ openssl req -new -x509 -days 1095 -key specflare.key -out specflare.crt

# Generate a self-signed certificate in one command (without being asked about entity info)
$ openssl req -new -x509 -days 365 -key specflare.key -out specflare.crt -subj "/C=RO/L=Bucharest/O=Specflare/CN=www.specflare.com"

# See the generated certificate
$ openssl x509 -text -in specflare.crt -noout
```

### Converting a certificate (or key) from one form to another
The most common formats are DER (raw binary formatm using DER ASN.1 encoding) and PEM (base64 encoded DER).
```bash
# Convert certificates from PEM to DER and vice-versa
$ openssl x509 -inform PEM -in specflare.pem -outform DER -out specflare.der
$ openssl x509 -inform DER -in specflare.der -outform PEM -out specflare.pem

# Convert RSA or DSA keys from PEM to DER and vice-versa
$ openssl rsa -inform PEM -in specflare.key -outform DER -out specflare.key.der
$ openssl dsa -inform DER -in specflare.key.der -outform PEM -out specflare.key.pem
```

## Understanding cipher suites
In SSL and TLS, cipher suites define how secure communication takes place. TLS clients submit a list of cipher suites that they support, and servers choose one suite from the list to use for the connection. Both client and server must support the selected suite.
```bash
$ openssl ciphers -v
RSA-PSK-AES256-CBC-SHA384 TLSv1 Kx=RSAPSK   Au=RSA  Enc=AES(256)  Mac=SHA384
# (... multiple lines omitted ...)
```
Each cipher suite line contains the following information:
- Cipher suite name, in our case `RSA-PSK-AES256-CBC-SHA384`
- Minimum protocol version that supports this cipher suite: `TLSv1`
- Key exchange algorithm: `Kx=RSAPSK`
- Authentication algorithm: `Au=RSA`
- Cipher algorithm and block size: `Enc=AES(256)`
- MAC (Message Authentication Code) algorithm: `Mac=SHA384`

You can use different selection strings for the cipher suite to tune your selection:
```bash
# Returns only cipher suites containing AES256
$ openssl ciphers -v 'AES256'
$ openssl ciphers -v 'ALL'

# All OpenSSL ciphers including NULL ciphers:
$ openssl ciphers -v 'ALL:eNULL'

# All RC4 ciphers skipping those without authentication:
$ openssl ciphers -v 'RC4:!COMPLEMENTOFDEFAULT'
```
The cipher suite selection string is well explained [here](https://www.openssl.org/docs/man1.1.0/man1/ciphers.html).

## Connecting to a TLS-secured host with OpenSSL
```bash
$ openssl s_client -help
$ openssl s_client -connect www.specflare.com:443

# Connecting by specifying the CA certificates (this path may be different on other OSes: /etc/ssl/certs/)
$ openssl s_client -connect www.specflare.com:443 -CAfile /etc/ssl/certs/ca-certificates.crt
```
You can try to connect to a TLS-secured host using only a small list of supported ciphers (from the client-side), by providing a cipher configuration string. You can even get the list of supported ciphers by the other party if you send successive requests with every cipher from the client side, and see which are supported and which are not.
```bash
$ openssl s_client -connect www.specflare.com:443 -cipher SHA1
```