---
layout: post
title: "Cryptography"
---

# Symmetric Cryptography

Symmetric cryptography uses the same key for both encryption and decryption of data.
Symmetric cryptography algorithms are typically fast and are suitable for processing large streams of data.
The requirement that both parties have access to the secret key is one of the main drawbacks of symmetric-key encryption.

# Asymmetric, or Public Key, Cryptography

Public-key cryptography, or asymmetric cryptography, is a cryptographic system that uses pairs of keys. 
Each pair consists of a public key (which may be known to others) and a private key (which may not be known by anyone except the owner).
The public key is used for encryption, while the private key is used for decryption.
Both the public key and the private key are mathematically linked.
Asymmetric encryption is way slower than symmetric encryption.

# Hash Function
Hash algorithms are one-way mathematical algorithms that take any length input and produce a fixed length output string.
Hash functions produce a hashed output that cannot be used to retrieve the original input data.
It is unlikely to find two messages that produce the same hash (or 'collide').

Hash functions can be used to uniquely identify large data item, store passwords, 
as checksums to detect accidental data corruption, in digital signatures.

# Digital Signature

A digital signature is a mathematical technique used to validate the authenticity and integrity of a digital messages or documents.
Digital signatures are based on asymmetric cryptography.
The individual who creates the digital signature uses a private key to encrypt a document.
To validate the digital signature the recipient should decrypt the digital signature using signer's public key 
and compare the decrypted signature with original data.

Signature-related documents can be of a large size hence signature is usually applied not to a document but to its hash. 

# Public Key Certificate

One important issue in asymmetric encryption is proof that a particular public key
is correct and belongs to the person or entity claimed, and has not been tampered with or replaced by some third party.
Public key certificate solves this problem.

Public key certificate, also known as a digital certificate, is an electronic document used to prove the ownership of a public key.
The certificate includes information about the public key, information about the identity of its owner (called the subject),
information about a certificate authority (CA) and the digital signature of the CA.
Certificate authority is an entity that issues the digital certificates.
The CA acts as a trusted third party - trusted both by the subject (owner) of the certificate and by the party relying upon the certificate.
If a client trusts the CA, and the signature is valid, 
then it can use that key to communicate securely with the certificate's subject.

# Transport Layer Security

Transport Layer Security (TLS), the successor of the now-deprecated Secure Sockets Layer (SSL), is a cryptographic protocol
that is used as the Security layer in HTTPS.
TLS uses asymmetric encryption to securely exchange a secret key which is then used for faster symmetric encryption.

When a client connects to a TLS-enabled server, the server provides its digital certificate.
The client confirms the validity of the certificate, and uses server's public key
to securely exchange a secret key with the server. 
The secret is used for subsequent encryption and decryption of data during the session.
