# Wireshark Lab 2 - Capture Filters and Display Filters

## Objective

Learn the difference between Capture Filters and Display Filters, understand when to use each, and practice filtering network traffic efficiently in Wireshark.

---

## Prerequisites

- Wireshark installed
- Completed Lab 1
- Active network connection
- Basic knowledge of packet capture

---

## Key Concepts

### Capture Filters
- Applied before packets are captured.
- Save only matching packets.
- Use libpcap syntax.
- Reduce storage usage and improve performance.

### Display Filters
- Applied after packets are captured.
- Hide or show packets without modifying the capture file.
- Use Wireshark display filter syntax.

---

## Capture Filter Examples

```text
tcp
udp
icmp
port 80
port 443
port 53
host 192.168.1.10
src host 192.168.1.10
dst host 192.168.1.20
net 192.168.1.0/24
not arp
port 80 or port 443
```

## Display Filter Examples

```text
tcp
udp
icmp
arp
dns
http
tls
bootp
ip.addr == 192.168.1.10
ip.src == 192.168.1.10
ip.dst == 192.168.1.20
tcp.port == 80
udp.port == 53
http.request.method == "GET"
tcp.analysis.flags
frame contains "login"
```

---

## Practical Exercises

- Capture only ICMP traffic.
- Capture only DNS traffic.
- Capture all traffic and filter ICMP.
- Filter DNS traffic.
- Filter TCP traffic.
- Filter packets by IP address.

---

## Verification Checklist

- [ ] Used a Capture Filter
- [ ] Used a Display Filter
- [ ] Compared both filter types
- [ ] Filtered traffic by protocol
- [ ] Filtered traffic by IP address
- [ ] Removed Display Filter successfully

---

## Learning Outcome

After completing this lab, you can:

- Explain the difference between Capture Filters and Display Filters.
- Use common capture filters in Wireshark.
- Analyze traffic using display filters.
- Filter packets by protocol, port, and IP address.
- Troubleshoot packet captures more efficiently.

---

**Next Lab:** Lab 3 – Packet Structure and Packet Dissection