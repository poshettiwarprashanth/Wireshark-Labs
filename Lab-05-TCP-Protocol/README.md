# Wireshark Lab 5 - TCP & TLS Analysis

## TCP Connection Analysis, TLS 1.3 Handshake and HTTPS Encryption

---

# Objective

The objective of this lab is to understand how a secure HTTPS connection is created and terminated by analyzing real network packets using Wireshark.

In this lab we will analyze:

- TCP connection establishment
- TCP 3-way handshake
- TLS 1.3 handshake after TCP connection
- Digital certificate verification
- Public key and private key usage
- Hashing and digital signatures
- Asymmetric and symmetric encryption
- Session key creation
- HTTPS encrypted communication
- TCP 4-way termination
- Graceful and forced TCP termination

---

# Prerequisites

Before starting this lab, the following concepts should be understood:

- Basic networking concepts
- OSI Model
- IP addressing
- Ports and protocols
- TCP and UDP basics
- Previous Wireshark Labs

Required tools:

- Wireshark
- Active internet connection
- Browser (Chrome/Firefox)
- Network interface (WiFi/Ethernet)

---

# Lab Environment

## Network Topology

```
Client (Browser)

IP Address:
192.168.0.2

Port:
24986


        |
        |
        | HTTPS Connection
        |
        |


Server

IP Address:
20.207.73.82

Port:
443
```

---

# Introduction to TCP

TCP (Transmission Control Protocol) is a connection-oriented transport layer protocol.

TCP works at:

```
OSI Layer 4
Transport Layer
```

TCP provides:

- Reliable communication
- Ordered packet delivery
- Error detection
- Flow control
- Congestion control
- Connection management

Unlike UDP, TCP creates a connection before sending application data.

---

# TCP Connection Establishment

Before sending HTTPS data, the client and server must create a TCP connection.

This process is called:

```
TCP 3-Way Handshake
```

The three packets are:

1. SYN
2. SYN + ACK
3. ACK

---

# TCP 3-Way Handshake

## Step 1: SYN Packet

The client starts communication.

Example:

```
Client                         Server


SYN
----------------------------->

```

Meaning:

```
Client:
"I want to establish a TCP connection."
```

The client sends:

- Initial sequence number
- TCP options
- Supported features

TCP State:

```
Client:
CLOSED → SYN_SENT
```

---

# Step 2: SYN + ACK Packet

The server receives the SYN request and responds.

```
Client                         Server


        SYN + ACK
<-----------------------------

```

Meaning:

```
Server:
"I received your request and I accept the connection."
```

The server sends:

- Its own sequence number
- Acknowledgement of client's sequence number

TCP State:

```
Server:
LISTEN → SYN_RECEIVED
```

---

# Step 3: ACK Packet

The client confirms the server response.

```
Client                         Server


ACK
----------------------------->

```

Meaning:

```
Client:
"I received your response. Connection established."
```

TCP State:

```
Client:
SYN_SENT → ESTABLISHED


Server:
SYN_RECEIVED → ESTABLISHED
```

---

# After TCP Handshake

Important:

TCP only creates a reliable communication channel.

At this stage:

```
TCP Connection Established
```

but:

```
Data is NOT encrypted yet
```

For HTTPS communication, TLS starts now.

The flow becomes:

```
Application Layer
        |
        |
        v

TLS Encryption Layer
        |
        |
        v

TCP Transport Layer
        |
        |
        v

IP Network Layer

```

---

# TLS Introduction

TLS (Transport Layer Security) provides security for HTTPS communication.

TLS provides:

## 1. Authentication

Verifies that the server is genuine.

Example:

```
Is this really github.com?
```

---

## 2. Encryption

Protects data from attackers.

Example:

Without TLS:

```
GET /login
username=password
```

can be read.

With TLS:

```
Encrypted Application Data
```

is unreadable.

---

## 3. Integrity

Ensures data is not modified during transmission.

Example:

Original:

```
Amount = 1000
```

Attacker changes:

```
Amount = 9000
```

TLS detects modification.

---

# TLS 1.3 Handshake Overview

After TCP handshake:

```
TCP 3-Way Handshake Complete

             |
             v

        Client Hello

             |
             v

        Server Hello

             |
             v

    Certificate Verification

             |
             v

       Key Exchange

             |
             v

       Session Keys Created

             |
             v

    Encrypted Application Data

```

---

# TLS Step 1: Client Hello

The browser starts the TLS handshake.

Example:

```
Client Browser

192.168.0.2

        |
        |
        v

HTTPS Server

20.207.73.82

```

The client sends:

```
TLS Client Hello
```

---

# Client Hello Contains

## 1. Supported TLS Versions

The client tells the server which TLS versions it supports.

Example:

```
TLS 1.3
TLS 1.2
```

Meaning:

```
Browser:
"I can communicate using these TLS versions."
```

---

## 2. Supported Cipher Suites

A cipher suite defines the cryptographic algorithms that will be used.

Example:

```
TLS_AES_256_GCM_SHA384
```

It contains:

```
AES-256-GCM

Encryption Algorithm


SHA384

Integrity / Hash Algorithm
```

---

## 3. Key Share

The client sends key exchange information.

Example:

```
X25519
```

Important:

The client DOES NOT send the session key.

It sends only information required to create a shared secret.

---

# Important Concept

The session key is never transmitted.

Wrong method:

```
Client:

"My secret key is ABC123"

        |
        v

Server

```

An attacker could steal it.

TLS does not work this way.

Instead:

Both sides independently create the same secret.

```
Client calculates secret

        +

Server calculates secret


        =


Same Shared Secret

```

---

# TLS Step 2: Server Hello

The server responds:

```
TLS Server Hello
```

The server selects:

- TLS version
- Cipher suite
- Key exchange parameters

Example:

```
TLS 1.3

TLS_AES_256_GCM_SHA384
```

Meaning:

```
Both sides agree:

Use AES-256-GCM
for encryption
```

---

# Wireshark Packet Example

From our capture:

```
Packet 1149

TLSv1.3 Record Layer:
Handshake Protocol:
Server Hello

```

This packet represents:

```
Server response after Client Hello
```

---

# Packet 1149 Analysis

```
Source:

20.207.73.82:443


Destination:

192.168.0.2:24986

```

Direction:

```
Server → Client
```

---

# Packet Structure

```
Ethernet II

        |

        v

IPv4

        |

        v

TCP

        |

        v

TLS

        |

        v

Server Hello

```

---

# Server Hello Fields

## Content Type

```
Handshake (22)
```

Meaning:

This TLS record contains handshake information.

---

## TLS Version Field

Wireshark may show:

```
TLS 1.2 (0x0303)
```

Even when using TLS 1.3.

Reason:

TLS 1.3 keeps the record layer version for compatibility.

The actual version is inside:

```
Supported Versions Extension
```

---


# TLS Key Exchange and Session Key Creation

After Server Hello, both client and server start creating the encryption keys.

A common misunderstanding is:

```
Client creates session key

Server creates session key

Then they send the session key to each other
```

This is incorrect.

The session key is never transmitted.

---

# How Do Client and Server Get the Same Session Keys?

TLS 1.3 uses:

```
ECDHE

(Elliptic Curve Diffie-Hellman Ephemeral)
```

for secure key exchange.

The purpose of ECDHE is:

```
Create a shared secret securely
```

without sending the secret over the network.

---

# ECDHE Key Exchange Process

## Step 1: Client Creates Key Pair

The client generates:

```
Client Private Key

        +

Client Public Key
```

The private key stays on the client.

The public key can be shared.

---

## Step 2: Server Creates Key Pair

The server generates:

```
Server Private Key

        +

Server Public Key
```

The private key stays on the server.

The public key can be shared.

---

## Step 3: Exchange Public Information

Communication:

```
Client                         Server


Client Public Key
----------------------------->


Server Public Key
<-----------------------------

```

Only public information is exchanged.

Private keys never leave the device.

---

# Step 4: Shared Secret Creation

Now both sides calculate:

Client:

```
Client Private Key

        +

Server Public Key

        |

        v

Shared Secret
```

---

Server:

```
Server Private Key

        +

Client Public Key

        |

        v

Shared Secret
```

---

Both produce:

```
Same Shared Secret
```

The attacker sees:

```
Public Keys
```

but cannot calculate the shared secret because they do not have:

```
Private Keys
```

---

# Session Key Generation

The shared secret is processed using TLS key derivation functions.

The result:

```
Session Keys
```

---

# What Are Session Keys?

Session keys are:

```
Symmetric Keys
```

They are temporary encryption keys used only for the current TLS connection.

Example:

```
Browser Session

        |
        |
        v

Session Key Created

        |
        |
        v

HTTPS Data Encrypted

        |
        |
        v

Connection Closed

        |
        |
        v

Session Key Destroyed

```

---

# Why Use Symmetric Keys for Data?

Asymmetric encryption is:

- Secure
- But slower

Symmetric encryption is:

- Very fast
- Efficient for large amounts of data

Therefore TLS uses:

```
Asymmetric Cryptography

        |
        |
        v

Create Shared Secret

        |
        |
        v

Generate Symmetric Session Keys

        |
        |
        v

Encrypt Actual Data

```

---

# Key Types Used in TLS

| Key | Type | Purpose |
|---|---|---|
| Server Private Key | Asymmetric | Creates digital signatures |
| Server Public Key | Asymmetric | Verifies signatures |
| Client Key Share | Ephemeral Asymmetric | Key exchange |
| Server Key Share | Ephemeral Asymmetric | Key exchange |
| Shared Secret | Secret Value | Used to derive keys |
| Session Keys | Symmetric | Encrypt HTTPS data |

---

# Digital Certificate

After Server Hello, the server sends its certificate.

Example:

```
TLS Certificate
```

The certificate proves:

```
This server is really github.com
```

---

# Digital Certificate Contains

Example:

```
Subject:

github.com


Issuer:

DigiCert


Validity:

Start Date
Expiry Date


Public Key:

Server Public Key


Digital Signature:

Certificate Authority Signature

```

---

# Purpose of Public Key in Certificate

A common confusion:

The certificate public key is NOT used to encrypt all HTTPS traffic.

It is mainly used for:

## 1. Authentication

The browser confirms:

```
This public key belongs to github.com
```

---

## 2. Digital Signature Verification

The server proves ownership of the private key.

Example:

Certificate contains:

```
Server Public Key
```

Server owns:

```
Matching Private Key
```

---

# Certificate Verification Process

The browser checks:

## 1. Domain Name

Example:

```
Certificate:

github.com

Website:

github.com

```

They must match.

---

## 2. Expiry Date

The browser checks:

```
Is the certificate still valid?
```

---

## 3. Certificate Authority

The browser checks:

```
Who issued this certificate?
```

Example:

```
DigiCert
```

The browser already trusts known Certificate Authorities.

---

## 4. Digital Signature Verification

The browser verifies:

```
Certificate was not modified
```

---

# Digital Signature and Hashing

Digital signatures use hashing.

Process:

```
Certificate Information

        |
        |
        v

Hash Function

        |
        |
        v

Hash Value

        |
        |
        v

Encrypted using CA Private Key

        |
        |
        v

Digital Signature

```

---

# How Client Verifies Digital Signature

The client uses:

```
CA Public Key
```

to verify the signature.

Process:

```
Digital Signature

        |
        |
        v

Decrypt using CA Public Key

        |
        |
        v

Original Hash

```

The client calculates a new hash:

```
Certificate Data

        |
        v

Hash Function

        |
        v

New Hash

```

Comparison:

```
Original Hash

        =

New Hash

```

If they match:

```
Certificate is authentic
and unchanged
```

---

# Where Hashing Happens in TLS

Hashing is used in multiple places.

---

# 1. Certificate Integrity

Purpose:

```
Check certificate was not modified
```

Example:

Attacker changes:

```
github.com

to

fakewebsite.com
```

Hash verification detects the change.

---

# 2. Digital Signature

Hashing creates a fingerprint of data before signing.

Flow:

```
Data

 |

 v

Hash

 |

 v

Private Key Signing

 |

 v

Digital Signature

```

---

# 3. TLS Handshake Integrity

TLS maintains a transcript hash.

It stores:

```
All handshake messages
```

Example:

```
Client Hello

Server Hello

Certificate

Finished

```

A hash confirms:

```
Nobody modified the handshake
```

---

# 4. Application Data Integrity

TLS 1.3 uses:

```
AEAD Encryption
```

Examples:

```
AES-GCM

ChaCha20-Poly1305
```

AEAD provides:

```
Encryption

+

Integrity Verification

```

---

# Encryption vs Hashing

| Feature | Encryption | Hashing |
|---|---|---|
| Purpose | Hide data | Verify integrity |
| Reversible | Yes | No |
| Uses Key | Yes | Usually no |
| Output | Ciphertext | Hash value |

---

# Encryption Flow

Before TLS:

```
HTTP Data

Example:

GET /index.html

```

After encryption:

```
Session Key

        |

        v

AES-GCM Encryption

        |

        v

Encrypted TLS Data

```

---

# TLS Finished Message

After keys are created, both sides send:

```
Finished
```

Purpose:

- Confirm handshake success
- Confirm same keys were generated
- Verify handshake integrity

---

# After TLS Handshake

Now the connection changes to:

```
TLS Application Data
```

Example Wireshark:

```
TLSv1.3 Application Data
```

The data is encrypted using:

```
Symmetric Session Keys
```

---

# Wireshark Encrypted Data Example

Example:

```
Encrypted Application Data:

c077485174068e6dcf2995a8...

```

This is:

```
Ciphertext
```

NOT:

```
Hash
```

NOT:

```
Digital Signature
```

---

# Difference

Hash:

```
Data

 |

 v

Hash Algorithm

 |

 v

Fingerprint

```

Cannot recover original data.

---

Encryption:

```
Data

 |

 v

Encryption Algorithm

+

Session Key

 |

 v

Ciphertext

```

Can be decrypted with the correct key.

---

# TLS Complete Flow Summary

```
TCP 3-Way Handshake

        |

        v

Client Hello

        |

        v

Server Hello

        |

        v

Certificate Verification

        |

        v

ECDHE Key Exchange

        |

        v

Shared Secret Created

        |

        v

Session Keys Generated

        |

        v

Encrypted Application Data

```

---

