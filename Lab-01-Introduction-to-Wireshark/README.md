# Lab 01 - Introduction to Wireshark

## Objective

The objective of this lab is to become familiar with Wireshark, understand its interface, capture live network traffic, and learn how to save and reopen packet capture files.

---

## Prerequisites

- Wireshark installed
- Windows 10/11 or Linux
- Internet connection
- Basic understanding of computer networks (recommended)

---

## Lab Overview

In this lab, you will learn:

- What Wireshark is
- Why Wireshark is used
- Understanding network interfaces
- Starting and stopping packet captures
- Exploring the Wireshark interface
- Saving packet capture files
- Opening previously saved captures

---

## Network Scenario

A computer connected to the Internet captures its own network traffic while browsing a website.

```
+-----------------+
|     Internet    |
+--------+--------+
         |
         |
+--------+--------+
| Router / Gateway|
+--------+--------+
         |
         |
+--------+--------+
|      PC         |
|  Wireshark      |
+-----------------+
```

---

## Lab Steps

### Step 1: Launch Wireshark

Open Wireshark from your system.

---

### Step 2: Select a Network Interface

Choose the active network interface.

Examples:

- Wi-Fi
- Ethernet

The active interface usually displays live traffic graphs.

---

### Step 3: Start Packet Capture

Double-click the active interface to begin capturing packets.

Observe that packets begin appearing immediately.

---

### Step 4: Generate Network Traffic

Open a web browser.

Visit:

- https://www.google.com

Wait for a few seconds while packets are captured.

---

### Step 5: Stop Capture

Click the **Stop** button (red square) in the toolbar.

The capture will stop and can now be analyzed.

---

### Step 6: Explore the Wireshark Interface

Wireshark consists of three main sections:

#### Packet List Pane

Displays all captured packets.

Includes:

- Packet Number
- Time
- Source
- Destination
- Protocol
- Length
- Info

---

#### Packet Details Pane

Displays protocol layers of the selected packet.

Examples:

- Frame
- Ethernet
- IPv4
- TCP
- UDP
- DNS

---

#### Packet Bytes Pane

Displays the raw packet data in hexadecimal and ASCII format.

---

### Step 7: Save the Capture

Navigate to:

```
File → Save As
```

Save the file as:

```
Lab1_FirstCapture.pcapng
```

---

### Step 8: Open a Saved Capture

Navigate to:

```
File → Open
```

Select:

```
Lab1_FirstCapture.pcapng
```

---

## Observations

During packet capture, various protocols may appear, including:

- ARP
- DNS
- TCP
- UDP
- TLS
- HTTP
- HTTPS
- ICMP

The exact protocols captured depend on the network activity during the lab.

---

## Expected Outcome

After completing this lab, you should be able to:

- Launch Wireshark
- Select the correct network interface
- Capture live network traffic
- Stop packet captures
- Navigate the Wireshark interface
- Save packet captures
- Open previously saved capture files

---

## Key Concepts Learned

- Network Packet
- Packet Capture
- Network Interface
- Protocol Analyzer
- Packet Inspection
- PCAPNG File Format

---

## Troubleshooting

### No packets are captured

- Verify that the correct network interface is selected.
- Ensure the system is connected to the network.
- Generate traffic by browsing a website or using the `ping` command.

---

### Too many packets appear

This is normal. Wireshark captures all traffic on the selected interface.

Filtering techniques will be covered in the next lab.

---

## Interview Questions

### 1. What is Wireshark?

A network protocol analyzer used to capture and analyze network traffic.

---

### 2. What is packet capture?

The process of recording network packets traveling across a network.

---

### 3. What file format does Wireshark use by default?

`.pcapng`

---

### 4. Name the three main panes of Wireshark.

- Packet List Pane
- Packet Details Pane
- Packet Bytes Pane

---

### 5. Can Wireshark capture encrypted HTTPS traffic?

Yes. It captures encrypted packets, but the encrypted application data cannot be read without the required decryption keys.

---

## My Lab Observations

* **Selected Network Interface:** Wi-Fi
* **Total Packets Captured:** 11,488
* **Most Frequently Observed Protocols:**

  * ARP (Address Resolution Protocol)
  * DNS (Domain Name System)
  * QUIC (Quick UDP Internet Connections)
* **Purpose of the Packet Details Pane:** To investigate and analyze a particular packet by viewing its protocol layers and detailed information.
* **Default Wireshark Capture File Format:** `.pcapng`


## Conclusion

In this lab, you gained hands-on experience with Wireshark by capturing live network traffic, exploring its interface, and learning how to save and reopen packet capture files. This foundational knowledge prepares you for analyzing network protocols and troubleshooting network communication in future labs.

---

## Next Lab

**Lab 02 – Capture Filters and Display Filters**