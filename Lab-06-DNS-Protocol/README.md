# Wireshark Lab 6 – DNS (Domain Name System) Analysis

## Objective

The objective of this lab is to understand how the Domain Name System (DNS) works by capturing and analyzing real DNS packets using Wireshark. In this lab, you will learn how a domain name is converted into an IP address, understand the DNS packet structure, analyze DNS Query and DNS Response packets, learn various DNS record types, understand recursive resolution, caching, TTL, and DNS security concepts. By the end of this lab, you will be able to confidently analyze DNS traffic during troubleshooting, incident response, and penetration testing.

---

# Prerequisites

* Wireshark installed
* Internet connection
* Basic understanding of IP Addressing
* Basic understanding of UDP
* Completed previous Wireshark Labs (Introduction, Filters, ARP, ICMP and TCP)

---

# What is DNS?

DNS stands for **Domain Name System**.

It is known as the **Phonebook of the Internet**.

Humans remember names such as:

* google.com
* github.com
* openai.com
* microsoft.com

Computers communicate using IP addresses such as:

* 142.251.221.206
* 20.207.73.82
* 104.18.x.x

DNS converts a human-readable domain name into an IP address that computers can understand.

Without DNS, users would have to remember the IP address of every website they wanted to visit.

Example:

google.com

↓

142.251.221.206

---

# Why DNS is Required

Suppose you open your browser and type:

google.com

Your computer does not know where Google's server is located.

The operating system first asks a DNS server:

"What is the IP address of google.com?"

The DNS server replies:

google.com → 142.251.221.206

After receiving the IP address, your browser establishes a TCP connection to Google's server.

DNS is therefore always performed before HTTP, HTTPS, FTP, SSH, or almost any Internet communication using domain names.

---

# DNS Resolution Process

The complete DNS resolution process is:

User Types Domain Name

↓

Browser

↓

Operating System

↓

Configured DNS Resolver

↓

Root DNS Server (if required)

↓

Top Level Domain (TLD) Server

↓

Authoritative DNS Server

↓

IP Address Found

↓

Response Returned

↓

Browser Connects to Website

---

# Types of DNS Queries

## Recursive Query

In a recursive query, the client requests the DNS resolver to find the complete answer.

Example:

Your PC asks Cloudflare DNS:

"What is the IP address of google.com?"

Cloudflare performs all necessary lookups and returns the final answer.

The client simply waits.

Recursive queries are the most common type used by end-user devices.

---

## Iterative Query

In an iterative query, the DNS server does not perform the complete lookup.

Instead, it replies with the next server that should be contacted.

Example:

Client

↓

Root Server

↓

"I don't know, ask the .com server."

↓

Client contacts .com server

↓

".com server says ask Google's authoritative server."

↓

Client contacts Google's authoritative server

↓

Final answer received

Iterative queries are generally used between DNS servers.

---

# DNS Hierarchy

DNS follows a hierarchical structure.

Root (.)

↓

Top Level Domain (.com)

↓

Second Level Domain (google.com)

↓

Subdomain (mail.google.com)

Every DNS lookup follows this hierarchy until the requested record is found.

---

# DNS Uses Which Protocol?

DNS primarily uses:

UDP Port 53

UDP is chosen because:

* Connectionless
* No three-way handshake
* Lower overhead
* Faster communication
* Small packet size
* Suitable for quick request-response communication

---

# When Does DNS Use TCP?

Although DNS usually uses UDP, TCP is required in certain situations.

DNS uses TCP when:

* Zone Transfers (AXFR)
* DNSSEC responses
* Large DNS responses
* Response size exceeds UDP limit
* UDP packet is truncated

Default Ports:

UDP 53 → Normal DNS Queries

TCP 53 → Large Responses and Zone Transfers

---

# Common DNS Record Types

## A Record

Maps a domain name to an IPv4 address.

Example:

google.com

↓

142.251.221.206

---

## AAAA Record

Maps a domain name to an IPv6 address.

Example:

google.com

↓

2404:6800:4007:831::200e

---

## CNAME Record

Canonical Name.

Used to create an alias for another hostname.

Example:

settings-win.data.microsoft.com

↓

atm-settingsfe-prod-geo2.trafficmanager.net

↓

settings-prod-scus-1-tagged.southcentralus.cloudapp.azure.com

↓

172.215.188.232

This chain is called a CNAME Chain.

Large companies use CNAMEs for:

* Load Balancing
* High Availability
* Geo Routing
* Easier Infrastructure Management

---

## MX Record

Mail Exchange Record.

Specifies the mail server responsible for receiving emails.

Example:

gmail.com

↓

Mail Servers

---

## NS Record

Name Server Record.

Specifies which DNS server is authoritative for a domain.

---

## PTR Record

Pointer Record.

Used for Reverse DNS Lookup.

Instead of:

Domain → IP

PTR performs:

IP → Domain

Example:

142.251.221.206

↓

pnmaaa-ba-in-f14.1e100.net

---

## SOA Record

Start of Authority.

Contains administrative information about the DNS zone.

SOA records are often returned when the requested record type does not exist.

---

## TXT Record

Stores arbitrary text information.

Commonly used for:

* SPF
* DKIM
* Domain Verification
* Email Authentication

---

# DNS Packet Structure

A DNS packet contains:

DNS Header

Questions

Answers

Authority

Additional

---

# DNS Header Fields

The DNS Header contains:

* Transaction ID
* Flags
* Question Count
* Answer Count
* Authority Count
* Additional Count

These fields allow DNS clients and servers to communicate efficiently.

---

# DNS Flags

## QR (Query/Response)

0 = Query

1 = Response

---

## Opcode

Usually:

0 = Standard Query

---

## AA (Authoritative Answer)

Indicates whether the response came from an authoritative DNS server.

---

## TC (Truncated)

0 = Complete Response

1 = Response Too Large

If TC = 1, the client usually retries using TCP.

---

## RD (Recursion Desired)

Indicates whether the client requests recursive resolution.

Usually:

RD = 1

Meaning:

"Please find the answer for me."

---

## RA (Recursion Available)

Returned by the DNS server.

RA = 1 means the server supports recursive resolution.

---

## RCODE

DNS Response Code.

Common values:

0 = No Error

2 = Server Failure

3 = NXDOMAIN (Domain Does Not Exist)

---

# DNS Cache

DNS servers cache responses to reduce lookup time.

Example:

First Lookup

google.com

↓

Cloudflare performs lookup

↓

Stores result in cache

↓

Second Lookup

↓

Returned immediately

↓

Faster response

Caching reduces Internet traffic and improves browsing speed.

---

# TTL (Time To Live)

TTL defines how long a DNS record can remain in cache.

Example:

TTL = 300 seconds

The record can be reused for 5 minutes.

After the TTL expires, a new DNS query must be performed.

---

# Practical Lab

## Step 1

Open Wireshark.

Select your active network interface.

Example:

Wi-Fi

Start packet capture.

---

## Step 2

Clear previous packets.

Stop capture.

Start capture again.

This makes analysis easier.

---

## Step 3

Open Command Prompt.

Flush the DNS cache.

Command:

ipconfig /flushdns

Why?

If the record is already cached, Windows will not generate a new DNS query.

Flushing the cache forces Windows to perform a fresh DNS lookup.

---

## Step 4

Generate DNS traffic.

Examples:

nslookup google.com

or

nslookup github.com

or

nslookup openai.com

You may also open these websites in your browser after flushing the cache.

---

## Step 5

Apply Display Filter.

dns

Only DNS packets will be displayed.

---

# Display Filters

Display all DNS packets:

dns

Display only Queries:

dns.flags.response == 0

Display only Responses:

dns.flags.response == 1

Display A Records:

dns.a

Display AAAA Records:

dns.aaaa

Display MX Records:

dns.mx

Display CNAME Records:

dns.cname

Display TXT Records:

dns.txt

Display DNS Errors:

dns.flags.rcode != 0

Display NXDOMAIN Responses:

dns.flags.rcode == 3

Display DNS over UDP:

udp.port == 53

Display DNS over TCP:

tcp.port == 53

---

# Commands Used During Lab

Flush DNS Cache

ipconfig /flushdns

Display DNS Cache

ipconfig /displaydns

Lookup Domain

nslookup google.com

Lookup Using Specific DNS Server

nslookup google.com 1.1.1.1

Reverse DNS Lookup

nslookup 142.251.221.206

---

# Real Packet Capture Overview

During this lab, the following DNS traffic was captured:

Windows Background Services:

* settings-win.data.microsoft.com
* login.live.com
* displaycatalog.mp.microsoft.com
* tas02.sls.update.microsoft.com

Manual DNS Queries:

* google.com
* github.com

DNS Server Used:

Cloudflare Public DNS

1.1.1.1

The captured packets included:

* A Record Queries
* AAAA Record Queries
* CNAME Chains
* PTR Queries
* Reverse DNS Responses
* SOA Records
* NXDOMAIN Responses

These packets will be analyzed in detail in the next section.

# Deep Packet Analysis

The following packet analysis is based on the real DNS traffic captured during this lab.

Example Query Packet:

192.168.0.10

↓

1.1.1.1

Standard Query

A google.com

Example Response Packet:

1.1.1.1

↓

192.168.0.10

Standard Query Response

google.com

↓

142.251.221.206

---

# Packet Flow

Laptop

↓

Wi-Fi

↓

Router (Default Gateway)

↓

ISP

↓

Cloudflare DNS (1.1.1.1)

↓

Cloudflare Performs DNS Resolution

↓

Response Returned

↓

Laptop

Even though the destination IP address is 1.1.1.1, the Ethernet destination MAC address belongs to the router because MAC addresses are only valid within the local network.

The router forwards the packet towards the Internet by replacing the Layer 2 Ethernet header at every hop while keeping the destination IP address unchanged.

---

# Layer 1 - Frame

The first section displayed by Wireshark is called **Frame**.

This is **not** an OSI layer.

It is metadata generated by Wireshark after capturing the packet.

The Frame section contains information such as:

* Frame Number
* Arrival Time
* Epoch Time
* Interface ID
* Packet Length
* Capture Length
* Time Since Previous Packet
* Encapsulation Type

These values are **not transmitted over the network**.

They only exist inside Wireshark.

---

# Layer 2 - Ethernet II

Ethernet II is the actual Data Link Layer header.

It contains:

* Destination MAC Address
* Source MAC Address
* EtherType

Example:

Destination MAC

↓

Router MAC Address

Source MAC

↓

Laptop MAC Address

EtherType

↓

IPv4 (0x0800)

Since Cloudflare is located on another network, the laptop cannot directly communicate with Cloudflare's MAC address.

Instead, the laptop sends the Ethernet frame to the Default Gateway (Router).

The router then forwards the packet towards Cloudflare.

If the destination device is located inside the same LAN, the destination MAC address belongs to the destination device instead of the router.

Example:

192.168.0.10

↓

192.168.0.20

Destination MAC

↓

192.168.0.20 MAC Address

Router is not involved.

---

# Layer 3 - IPv4

The IPv4 header contains:

* Version
* Header Length
* DSCP
* Total Length
* Identification
* Flags
* Fragment Offset
* TTL
* Protocol
* Header Checksum
* Source Address
* Destination Address

Important fields:

Source Address

↓

192.168.0.10

Destination Address

↓

1.1.1.1

Protocol

↓

17 (UDP)

TTL

↓

Limits how many routers the packet can traverse before being discarded.

The IP header remains mostly unchanged throughout the journey except for TTL, which decreases by one at every router.

---

# Layer 4 - User Datagram Protocol (UDP)

DNS primarily uses UDP because UDP is:

* Connectionless
* Faster
* Lower Overhead
* No Three-Way Handshake
* Suitable for Small Messages

UDP Header Fields:

* Source Port
* Destination Port
* Length
* Checksum

---

## Source Port

Example:

60920

The operating system assigns a temporary dynamic port called an **Ephemeral Port**.

This port uniquely identifies the conversation.

When the response returns, the operating system uses this destination port to deliver the packet to the correct application.

---

## Destination Port

53

DNS always listens on Port 53.

Since this port is universally reserved for DNS, it is called a **Well-Known Port**.

---

## UDP Length

The UDP Length field contains:

UDP Header

*

UDP Payload

Example:

UDP Header

8 Bytes

DNS Payload

62 Bytes

UDP Length

70 Bytes

It is **not** only the payload length.

---

## UDP Checksum

The checksum verifies packet integrity.

It helps detect accidental corruption caused by transmission errors.

The sender calculates a checksum using:

* UDP Header
* UDP Payload
* IP Pseudo Header

The checksum value is inserted into the UDP header.

When the receiver gets the packet, it performs the same calculation again.

If both values match:

Packet Accepted

If they do not match:

Packet Discarded

UDP never retransmits corrupted packets.

If a DNS response is not received because the packet was discarded, the DNS client itself usually retries after a timeout.

The checksum protects against accidental corruption.

It does **not** protect against DNS Spoofing or DNS Cache Poisoning because an attacker can modify the packet and recalculate the checksum.

---

# Layer 7 - Domain Name System (DNS)

The DNS header is the heart of the DNS protocol.

It contains:

* Transaction ID
* Flags
* Question Count
* Answer Count
* Authority Count
* Additional Count

---

# Transaction ID

Example:

0x9bc4

The Transaction ID is a randomly generated 16-bit number assigned by the DNS client.

The DNS server copies the exact same Transaction ID into the response.

Example:

Query

Transaction ID

0x9bc4

↓

Response

Transaction ID

0x9bc4

If both IDs match, the client knows that the response belongs to the original query.

If the Transaction IDs do not match, the response is ignored.

This allows multiple DNS queries to be processed simultaneously without confusion.

---

# DNS Flags

The DNS Flags field contains multiple control bits.

---

## QR (Query/Response)

0

↓

DNS Query

1

↓

DNS Response

The query packet contains:

QR = 0

The response packet contains:

QR = 1

---

## Opcode

Normally:

0

Meaning:

Standard Query

---

## AA (Authoritative Answer)

Indicates whether the answer comes directly from an authoritative DNS server.

Normally:

AA = 0

for recursive resolvers like Cloudflare.

---

## TC (Truncated)

0

↓

Complete Response

1

↓

Response Too Large

If the response is truncated, the client usually retries using TCP.

---

## RD (Recursion Desired)

Usually:

RD = 1

The client requests the DNS resolver to perform the complete lookup.

Instead of contacting Root Servers, TLD Servers and Authoritative Servers itself, the client requests the recursive resolver to perform all the work.

---

## RA (Recursion Available)

Returned by the DNS Server.

RA = 1

means:

The DNS server supports recursive resolution.

---

## RCODE

Response Code.

Common values:

0

↓

No Error

2

↓

Server Failure

3

↓

NXDOMAIN

Domain Does Not Exist

---

# Question Count

Example:

1

Meaning:

One DNS question exists inside the packet.

Example:

google.com

---

# Answer Count

Query Packet:

0

The client does not have any answers.

Response Packet:

1

The DNS server returns one answer.

Example:

google.com

↓

142.251.221.206

---

# Authority Count

Usually:

0

If required, the server may return authoritative DNS server information.

---

# Additional Count

Usually:

0

May contain additional DNS records to assist resolution.

---

# Queries Section

The Queries section specifies what the client is asking.

Example:

Name

↓

google.com

Type

↓

A

Class

↓

IN

---

## Name

The requested domain name.

Example:

google.com

---

## Type

Specifies the requested DNS Record Type.

Common values:

A

IPv4 Address

AAAA

IPv6 Address

MX

Mail Server

PTR

Reverse Lookup

CNAME

Alias

TXT

Text Record

---

## Class

Almost always:

IN

Meaning:

Internet

---

# Answers Section

The Answers section exists only in DNS Responses.

Example:

google.com

↓

142.251.221.206

Information displayed:

* Name
* Type
* Class
* TTL
* Data Length
* IP Address

This section contains the final answer requested by the client.

---

# TTL (Time To Live)

TTL specifies how long the client can cache the DNS response.

Example:

TTL

300 Seconds

The operating system stores the response inside the DNS Cache.

Future requests for the same domain can be answered locally without contacting the DNS server until the TTL expires.

---

# Reverse DNS Lookup

Instead of:

Domain

↓

IP

Reverse DNS performs:

IP

↓

Domain

Example:

142.251.221.206

↓

pnmaaa-ba-in-f14.1e100.net

PTR Records are used for Reverse DNS.

---

# Understanding Google's 1e100.net

Google owns the domain:

1e100.net

The name comes from:

10^100

which is called a **Googol**.

Many Google servers return hostnames ending with:

1e100.net

during Reverse DNS lookups.

---

# Understanding CNAME Chains

Many Microsoft DNS responses contained multiple CNAME records.

Example:

settings-win.data.microsoft.com

↓

atm-settingsfe-prod-geo2.trafficmanager.net

↓

settings-prod-scus-1-tagged.southcentralus.cloudapp.azure.com

↓

172.215.188.232

This sequence is called a **CNAME Chain**.

CNAME chains allow organizations to:

* Redirect traffic
* Perform Load Balancing
* Perform Geo Routing
* Migrate infrastructure without changing the public hostname

# DNS Security and Pentesting Concepts

DNS is one of the most important protocols during penetration testing because almost every organization depends on DNS for internal and external communication.

Attackers and security professionals analyze DNS because it can reveal:

* Internal infrastructure
* Server names
* Cloud providers
* Subdomains
* Misconfigurations
* Potential attack surfaces

---

# Common DNS Attacks

## DNS Cache Poisoning

DNS Cache Poisoning is an attack where an attacker inserts fake DNS information into a DNS resolver cache.

Normal DNS:

```
User

↓

DNS Resolver

↓

google.com

↓

142.251.221.206
```

Poisoned DNS:

```
User

↓

DNS Resolver

↓

google.com

↓

6.6.6.6 (Attacker Server)
```

The user believes they are connecting to the real website but is redirected to a malicious server.

---

# How DNS Cache Poisoning Works

An attacker attempts to send a fake DNS response before the real DNS server replies.

The attacker tries to guess:

* Transaction ID
* Source Port
* DNS Request Information

If the fake response matches the expected values, the resolver may accept it.

Modern DNS implementations reduce this risk using:

* Random Transaction IDs
* Source Port Randomization
* DNSSEC

---

# DNS Spoofing

DNS Spoofing occurs when an attacker provides a fake DNS response.

Example:

Victim requests:

```
bank.com
```

Attacker responds:

```
bank.com

↓

192.168.1.50
```

The victim connects to the attacker's server.

---

# DNSSEC

DNSSEC stands for:

Domain Name System Security Extensions.

DNSSEC provides authentication for DNS responses.

It uses digital signatures to verify that DNS records have not been modified.

DNSSEC provides:

* Data Integrity
* Data Authentication
* Protection against DNS Spoofing

DNSSEC does not encrypt DNS traffic.

It only verifies authenticity.

---

# DNS Enumeration in Penetration Testing

DNS enumeration is the process of collecting DNS information about a target.

Security professionals use DNS enumeration to discover:

* Subdomains
* DNS Records
* Mail Servers
* Name Servers
* Cloud Services

Common tools:

```
nslookup
dig
host
dnsrecon
amass
subfinder
```

---

# Useful DNS Commands

## nslookup

Basic DNS lookup:

```
nslookup google.com
```

Output:

```
Name:
google.com

Address:
142.251.221.206
```

---

## Query Specific DNS Server

Using Google DNS:

```
nslookup google.com 8.8.8.8
```

Using Cloudflare DNS:

```
nslookup google.com 1.1.1.1
```

---

## Reverse Lookup

```
nslookup 142.251.221.206
```

Used to find PTR records.

---

## dig Command

Linux DNS analysis tool.

Example:

```
dig google.com
```

Shows:

* Query
* Response
* TTL
* DNS Flags
* Answer Records

---

## DNS Record Enumeration

Query A record:

```
dig google.com A
```

Query AAAA record:

```
dig google.com AAAA
```

Query MX record:

```
dig google.com MX
```

Query NS record:

```
dig google.com NS
```

Query TXT record:

```
dig google.com TXT
```

---

# Wireshark DNS Filters for Security Analysis

Display all DNS traffic:

```
dns
```

---

Only DNS Queries:

```
dns.flags.response == 0
```

---

Only DNS Responses:

```
dns.flags.response == 1
```

---

Find Failed DNS Queries:

```
dns.flags.rcode != 0
```

---

Find NXDOMAIN Responses:

```
dns.flags.rcode == 3
```

---

Find DNS Traffic From Specific Host:

```
ip.addr == 192.168.0.10 && dns
```

---

Find Queries For Specific Domain:

```
dns.qry.name contains "google"
```

---

Find DNS Servers:

```
dns.flags.response == 1
```

Then analyze:

```
Source IP
```

The source IP of responses is usually the DNS server.

---

# Capture Filters for DNS

Capture only DNS traffic:

```
port 53
```

Capture UDP DNS:

```
udp port 53
```

Capture TCP DNS:

```
tcp port 53
```

---

# Troubleshooting DNS Using Wireshark

## Problem: Website Does Not Open

Check:

1. Did DNS Query leave the computer?

Filter:

```
dns.flags.response == 0
```

---

2. Did DNS Response return?

Filter:

```
dns.flags.response == 1
```

---

3. Was the response successful?

Check:

```
RCODE
```

Expected:

```
NOERROR
```

---

## Problem: DNS Timeout

Possible causes:

* DNS server unavailable
* Firewall blocking port 53
* Network connectivity issue
* Wrong DNS configuration

---

# Complete DNS Packet Flow Summary

When a user opens:

```
google.com
```

The process is:

```
Application Layer

Browser requests domain resolution


↓

DNS Query

UDP Port 53


↓

Network Layer

IPv4 Packet

Source:
192.168.0.10

Destination:
1.1.1.1


↓

Data Link Layer

Ethernet Frame

Source MAC:
Laptop MAC

Destination MAC:
Router MAC


↓

Cloudflare DNS


↓

DNS Response

Transaction ID matched

Answer received


↓

Browser starts TCP/TLS connection
```

---

# Important Concepts Learned in Lab 6

## Frame

Wireshark metadata.

Not part of the transmitted packet.

---

## Ethernet II

Layer 2 protocol.

Contains:

* Source MAC
* Destination MAC
* EtherType

---

## IPv4

Layer 3 protocol.

Contains:

* Source IP
* Destination IP
* TTL
* Protocol

---

## UDP

Layer 4 protocol.

Contains:

* Source Port
* Destination Port
* Length
* Checksum

---

## DNS

Application Layer protocol.

Contains:

* Transaction ID
* Flags
* Questions
* Answers
* Records

---

# Interview Questions

## 1. Why does DNS use UDP instead of TCP?

DNS uses UDP because:

* No handshake
* Lower latency
* Smaller packets
* Less overhead

TCP is used when responses are large or during zone transfers.

---

## 2. Why is the destination MAC address your router when the destination IP is 1.1.1.1?

Because MAC addresses work only inside the local network.

For external destinations, the packet is sent to the default gateway.

---

## 3. Does UDP provide reliability?

No.

UDP does not provide:

* Retransmission
* Ordering
* Delivery guarantee

---

## 4. What does UDP checksum do?

It detects accidental packet corruption.

It does not provide encryption or protection against attackers.

---

## 5. Can checksum prevent DNS poisoning?

No.

Attackers can modify the packet and generate a valid checksum.

---

## 6. What is Transaction ID in DNS?

A random identifier used to match DNS responses with the correct queries.

---

## 7. What happens if DNS Transaction ID does not match?

The DNS response is ignored.

---

## 8. What is TTL?

Time To Live.

It specifies how long a DNS record can remain cached.

---

## 9. Difference between A and AAAA records?

A record:

IPv4 address

AAAA record:

IPv6 address

---

## 10. What is a CNAME?

A Canonical Name record.

It creates an alias pointing to another hostname.

---

# Lab 6 Final Checklist

Completed:

✅ DNS fundamentals
✅ DNS hierarchy
✅ Recursive and iterative queries
✅ DNS record types
✅ DNS over UDP
✅ UDP header analysis
✅ UDP checksum concept
✅ DNS Query packet analysis
✅ DNS Response packet analysis
✅ Transaction ID analysis
✅ DNS flags analysis
✅ TTL analysis
✅ CNAME chain analysis
✅ Reverse DNS analysis
✅ Wireshark DNS filters
✅ DNS troubleshooting
✅ DNS security concepts
✅ Pentesting relevance

---

# Key Takeaways

DNS is one of the first protocols involved whenever a user accesses a website.

Understanding DNS packet analysis allows security professionals to:

* Identify malicious domains
* Detect suspicious DNS activity
* Investigate malware communication
* Perform reconnaissance
* Troubleshoot network problems

Wireshark allows us to see the complete DNS communication:

```
DNS Query

↓

DNS Resolver

↓

DNS Response

↓

Application Connection
```

By analyzing DNS packets, a security professional can understand how systems communicate and identify abnormal behavior during security investigations.

---

# Nmap DNS Enumeration Commands

Nmap is mainly a network scanning tool, but it also provides DNS-related capabilities that are useful during reconnaissance and penetration testing.

DNS enumeration helps security professionals discover:

- Domain information
- DNS records
- Hostnames
- Subdomains
- Name servers
- Mail servers

---

# Basic DNS Lookup Using Nmap

## Resolve Hostname

Command:

nmap -sL google.com

Explanation:

- -sL performs a DNS lookup without sending packets to the target.
- It lists the IP address associated with the hostname.

Example:

google.com

↓

142.251.221.206

---

# Reverse DNS Lookup

Command:

nmap -sL 192.168.0.0/24

Explanation:

- Performs reverse DNS lookups for hosts in the network range.
- Useful for identifying hostnames.

Example:

192.168.0.10

↓

DESKTOP-PC

---

# Disable DNS Resolution During Scanning

Command:

nmap -n 192.168.0.10

Explanation:

- Prevents Nmap from performing DNS lookups.
- Makes scans faster.
- Useful when DNS information is not required.

---

# Enable DNS Resolution

Command:

nmap -R 192.168.0.10

Explanation:

- Forces reverse DNS resolution.
- Attempts to discover the hostname of the target.

---

# Use Specific DNS Server With Nmap

Command:

nmap --dns-server 1.1.1.1 example.com

Explanation:

- Forces Nmap to use Cloudflare DNS.
- Useful when testing different DNS resolvers.

Example:

nmap --dns-server 8.8.8.8 example.com

Uses Google DNS.

---

# DNS Brute Force Subdomain Discovery

Command:

nmap --script dns-brute example.com

Explanation:

Uses the Nmap NSE script engine to discover common subdomains.

Example discoveries:
