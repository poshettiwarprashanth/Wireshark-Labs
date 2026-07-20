# Wireshark Lab 7 - DHCP (Dynamic Host Configuration Protocol) Analysis

## Objective

Learn how DHCP automatically assigns IP addresses and network configuration to clients by capturing and analyzing the complete DHCP DORA process using Wireshark.

---

# Prerequisites

- Wireshark installed
- Administrator privileges
- Active network connection
- Basic understanding of IP addressing

---

# What is DHCP?

DHCP (Dynamic Host Configuration Protocol) automatically assigns network configuration to devices joining a network.

Instead of manually configuring:

- IP Address
- Subnet Mask
- Default Gateway
- DNS Server

the DHCP server provides these automatically.

---

# Why DHCP?

Without DHCP:

- Every device must be configured manually.
- Difficult to manage large networks.
- High chance of IP conflicts.
- Time consuming.

With DHCP:

- Automatic IP allocation
- Centralized management
- Prevents duplicate IP addresses
- Easy network administration

---

# DHCP Uses UDP

| Client | Server |
|---------|---------|
| UDP Port 68 | UDP Port 67 |

DHCP uses UDP because the client initially does not have an IP address and cannot establish a TCP connection.

---

# DHCP DORA Process

DORA stands for:

1. Discover
2. Offer
3. Request
4. Acknowledge

---

# DHCP Packet Flow

```
Client
(No IP)

        │
        │ DHCP Discover
        ▼
255.255.255.255

        ▲
        │ DHCP Offer
        │
DHCP Server

        │
        │ DHCP Request
        ▼

DHCP Server

        ▲
        │ DHCP ACK
        │

Client receives:

• IP Address
• Subnet Mask
• Default Gateway
• DNS Server
• Lease Time
```

---

# Practical Performed

Started Wireshark capture on the Wi-Fi interface.

Executed:

```cmd
ipconfig /release
```

Released the currently assigned IP address.

Executed:

```cmd
ipconfig /renew
```

Requested a new IP address from the DHCP server.

Applied Wireshark display filter:

```
dhcp
```

Captured the DHCP communication successfully.

---

# Packets Captured

| Packet | Description |
|---------|-------------|
| DHCP Release | Client returned previous leased IP |
| DHCP Discover | Client searched for DHCP server |
| DHCP Offer | Server offered available IP |
| DHCP Request | Client requested offered IP |
| DHCP ACK | Server confirmed lease |

Duplicate Request and ACK packets were also observed due to DHCP application-level retry mechanisms.

---

# DHCP Release

Purpose:

Client informs the DHCP server that it is releasing its leased IP address.

Example:

```
Client (192.168.0.16)

↓

DHCP Release

↓

Server (192.168.0.1)
```

Characteristics:

- Unicast communication
- Source IP = Client IP
- Destination IP = DHCP Server
- UDP 68 → 67

---

# DHCP Discover

Purpose:

Client searches for any available DHCP server.

Characteristics:

| Field | Value |
|--------|-------|
| Source IP | 0.0.0.0 |
| Destination IP | 255.255.255.255 |
| Source Port | 68 |
| Destination Port | 67 |
| Ethernet Destination | FF:FF:FF:FF:FF:FF |

Reason:

The client has no IP address and does not know the DHCP server.

Therefore it broadcasts the request.

---

# DHCP Offer

Purpose:

DHCP server offers an available IP address.

Contains:

- Offered IP Address
- DHCP Server Identifier
- Lease Time
- Subnet Mask
- Router
- DNS Server

Offer may be:

- Broadcast
- Unicast

depending on the client capabilities and DHCP server implementation.

---

# DHCP Request

Purpose:

Client accepts one DHCP server's offer.

Initial DORA:

Usually Broadcast

Reason:

If multiple DHCP servers replied, broadcasting informs all servers which offer was accepted.

Lease Renewal:

Usually Unicast

Reason:

Client already knows the DHCP server.

---

# DHCP ACK

Purpose:

Server officially assigns the IP address.

After ACK:

The client configures:

- IP Address
- Subnet Mask
- Default Gateway
- DNS Server

Communication may be:

- Broadcast
- Unicast

depending on the client's broadcast flag and server behavior.

---

# Broadcast vs Unicast Summary

| Packet | Communication |
|---------|---------------|
| Discover | Always Broadcast |
| Offer | Broadcast or Unicast |
| Request | Broadcast (Initial), Unicast (Renewal) |
| ACK | Broadcast or Unicast |

---

# Important DHCP Header Fields

## Transaction ID (XID)

Unique identifier for one DHCP conversation.

Example:

```
Discover

↓

Offer

↓

Request

↓

ACK

All share the same Transaction ID.
```

Used by the client and server to match packets belonging to the same DHCP transaction.

---

## Client MAC Address

Used to uniquely identify the client before an IP address is assigned.

---

## Your (Client) IP Address (yiaddr)

Contains the IP address offered by the DHCP server.

---

## DHCP Options

### Option 53

DHCP Message Type

Examples:

- Discover
- Offer
- Request
- ACK

---

### Option 54

DHCP Server Identifier

Identifies which DHCP server sent the Offer.

---

### Option 51

IP Address Lease Time

Specifies how long the client can use the assigned IP address.

---

### Option 1

Subnet Mask

Example:

```
255.255.255.0
```

---

### Option 3

Router

Default Gateway.

---

### Option 6

DNS Server

Used for domain name resolution.

---

### Option 55

Parameter Request List

Client requests configuration parameters such as:

- Subnet Mask
- Router
- DNS
- Lease Time
- Domain Name

---

# Duplicate DHCP Request and ACK

Observed:

```
Request

↓

Request

↓

ACK

↓

ACK
```

Reason:

UDP does not perform retransmissions.

Instead, DHCP implements its own timeout and retry mechanism at the application layer.

If the client does not receive the expected response within a timeout period, it retransmits the Request.

The DHCP server recognizes the duplicate using the Transaction ID and sends the ACK again without allocating a new IP address.

---

# Why UDP?

DHCP occurs before the client has an IP address.

TCP requires connection establishment, which is not possible at this stage.

UDP allows connectionless communication using broadcasts.

---

# Why Source IP is 0.0.0.0?

The client has not yet received an IP address.

Therefore it uses the special address:

```
0.0.0.0
```

to indicate that it currently has no valid IP.

---

# Why Destination IP is 255.255.255.255?

The client does not know:

- DHCP Server IP
- Gateway
- Network devices

Therefore it broadcasts the request to all devices on the local network.

---

# Why Destination MAC is FF:FF:FF:FF:FF:FF?

The client also does not know the DHCP server's MAC address.

Therefore the Ethernet frame is broadcast.

---

# DHCP Lease Renewal

When approximately 50% of the lease time expires, the client attempts to renew the lease by sending a unicast DHCP Request directly to the DHCP server.

If renewal fails before the lease expires, the client retries. If the lease ultimately expires without renewal, the client must restart the DHCP process beginning with DHCP Discover.

---

# Wireshark Display Filters

```
dhcp
```

or

```
bootp
```

---

# Capture Filter

No capture filter was used.

Traffic was filtered later using the display filter.

---

# Common DHCP Ports

| Protocol | Port |
|-----------|------|
| DHCP Server | UDP 67 |
| DHCP Client | UDP 68 |

---

# Interview Questions

### What does DHCP stand for?

Dynamic Host Configuration Protocol.

---

### Why does DHCP use UDP instead of TCP?

Because the client does not initially have an IP address and cannot establish a TCP connection.

---

### What is the DHCP DORA process?

Discover → Offer → Request → Acknowledge.

---

### Why is DHCP Discover broadcast?

The client does not know the DHCP server's IP address.

---

### Why is the source IP 0.0.0.0?

Because the client has not yet been assigned an IP address.

---

### Why is the destination IP 255.255.255.255?

To broadcast the request to all devices on the local network.

---

### How does the DHCP server identify the client?

Using the client's MAC address (and optionally a Client Identifier).

---

### What is a DHCP lease?

The amount of time for which a client is permitted to use an assigned IP address.

---

### Why were duplicate DHCP Request and ACK packets captured?

DHCP implements its own application-layer retry mechanism over UDP. If a response is delayed or not received within the timeout, the client retransmits the Request, and the server replies with another ACK. UDP itself does not provide retransmission.

---

# Key Takeaways

- DHCP automates IP address assignment.
- DHCP uses UDP ports 67 and 68.
- Discover is always broadcast.
- Offer and ACK may be broadcast or unicast.
- Initial Request is usually broadcast; renewal Request is typically unicast.
- DHCP relies on the client's MAC address before an IP is assigned.
- Transaction IDs uniquely identify each DHCP exchange.
- DHCP options provide essential network configuration.
- DHCP implements its own retry mechanism because UDP does not provide reliability.
- Wireshark allows detailed analysis of every stage of the DHCP process.

```text
# Network Troubleshooting and Diagnostics Cheat Sheet

A handy reference guide for common networking commands across Windows, Linux, and Nmap.

---

## 💻 Windows Commands

### View Current IP Configuration
Displays basic network configuration for all adapters.

ipconfig

Displays:
* IP Address
* Subnet Mask
* Gateway
* DNS Server

### Detailed Configuration
Shows full TCP/IP configuration for all adapters.

ipconfig /all

Displays:
* DHCP Enabled status
* DHCP Server IP
* Lease Obtained / Lease Expires timestamps
* MAC Address (Physical Address)
* DNS Servers
* IPv4 and IPv6 details

### Release Current Lease
Sends a DHCP Release notification to the server and removes the currently assigned IP address.

ipconfig /release

### Renew IP Address
Starts a DHCP Request to renew an existing lease, or a full DORA process if no lease exists.

ipconfig /renew

### Flush DNS Cache
Clears the local DNS resolver cache, forcing Windows to request fresh DNS records on the next lookup.

ipconfig /flushdns

### Register DNS
Manually refreshes all DHCP leases and re-registers the computer's DNS names/records with the DNS server.

ipconfig /registerdns

---

## 🐧 Linux Commands

### Show IP Address
Displays IP addresses assigned to all network interfaces.

ip addr
# Alternative shorthand:
ip a

### Show Routing Table
Displays the current IP routing table, including the default gateway.

ip route

### View Network Interfaces
Shows the status and properties of all network link devices (up/down state, MAC addresses).

ip link

### Release DHCP Lease
Releases the current IP address using the DHCP client daemon. (Equivalent to ipconfig /release)

sudo dhclient -r

### Request New DHCP Lease
Requests a new IP address configuration from the DHCP server. (Equivalent to ipconfig /renew)

sudo dhclient

### Specify an Interface
Request or release a lease for a specific network interface.

# For wired ethernet:
sudo dhclient eth0

# For wireless:
sudo dhclient wlan0

### Check Current Lease File
Views saved DHCP lease details, including lease time, assigned IP, and renewal schedules.

cat /var/lib/dhcp/dhclient.leases

### Show DNS Configuration
Displays the system's currently configured DNS nameservers.

cat /etc/resolv.conf

### NetworkManager Management
Check the status of network interfaces or restart the network service.

# Check status:
nmcli device status

# Restart the service:
sudo systemctl restart NetworkManager

---

## 🔍 Nmap Commands

### Discover Active Hosts
Performs a ping scan across a subnet to find live hosts without port scanning. Useful for locating a DHCP server.

nmap -sn 192.168.0.0/24

### Scan Common Ports
Scans the most common ports on a target device (e.g., a home router).

nmap 192.168.0.1

### Detect Services
Attempts to determine the version of the services running on open ports.

nmap -sV 192.168.0.1

### UDP Scan
Performs a UDP port scan rather than the default TCP scan.

sudo nmap -sU 192.168.0.1

### Scan DHCP Server Port
Checks UDP port 67, which is used by DHCP servers to listen for client requests.

sudo nmap -sU -p 67 192.168.0.1

Note: Many DHCP servers do not respond like normal network services. Seeing the port listed as open|filtered is normal and expected behavior.

### Scan DHCP Client Port
Checks UDP port 68, used by clients to receive DHCP server responses.

sudo nmap -sU -p 68 192.168.0.10

### Aggressive Scan
An all-in-one comprehensive scan.

sudo nmap -A 192.168.0.1

Performs:
* OS Detection
* Service & Version Detection
* Default Nmap Scripting Engine (NSE) scripts
* Traceroute

```