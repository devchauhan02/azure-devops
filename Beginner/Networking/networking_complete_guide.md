# 🌐 Complete Networking Guide — In Depth

> Covers: Introduction → Types → Topology → Devices → OSI → TCP/IP → MAC → IPv4 → IPv6 → Protocols → Subnetting → VLAN → DHCP → DNS → FTP → NFS → SMB → SSH → RDP → Telnet → NAT → VPN → NTA
> Each section includes **explanations**, **cross-questions**, and **answers**.

---

## Table of Contents

1. [Introduction to Networking](#1-introduction-to-networking)
2. [Types of Networks](#2-types-of-networks)
3. [Network Topology](#3-network-topology)
4. [Network Devices](#4-network-devices)
5. [OSI Model](#5-osi-model)
6. [TCP/IP Model](#6-tcpip-model)
7. [MAC Address](#7-mac-address)
8. [IPv4 Addressing](#8-ipv4-addressing)
9. [IPv6 Addressing](#9-ipv6-addressing)
10. [Protocols Overview](#10-protocols-overview)
11. [Subnetting](#11-subnetting)
12. [VLAN](#12-vlan-virtual-local-area-network)
13. [DHCP](#13-dhcp-dynamic-host-configuration-protocol)
14. [DNS](#14-dns-domain-name-system)
15. [FTP](#15-ftp-file-transfer-protocol)
16. [NFS](#16-nfs-network-file-system)
17. [SMB](#17-smb-server-message-block)
18. [SSH](#18-ssh-secure-shell)
19. [RDP](#19-rdp-remote-desktop-protocol)
20. [Telnet](#20-telnet)
21. [NAT](#21-nat-network-address-translation)
22. [VPN](#22-vpn-virtual-private-network)
23. [NTA — Network Traffic Analysis](#23-nta-network-traffic-analysis)

---

## 1. Introduction to Networking

### What is a Network?

A **computer network** is a collection of two or more computing devices connected together to share resources (files, printers, internet), communicate, and exchange data. Networking is the backbone of modern communication, enabling everything from simple file sharing between two PCs to the global internet.

### Key Concepts

| Term | Meaning |
|------|---------|
| **Node** | Any device connected to a network (PC, printer, router) |
| **Link** | The communication channel between nodes |
| **Protocol** | A set of rules governing data communication |
| **Bandwidth** | Maximum data transfer rate (measured in bps, Mbps, Gbps) |
| **Latency** | Time taken for data to travel from source to destination |
| **Throughput** | Actual data successfully transferred per unit time |
| **Jitter** | Variation in latency over time |
| **Packet** | A unit of data transmitted over a network |

### Why Networking Matters

- **Resource Sharing** — multiple users share a single printer, disk, or internet connection
- **Communication** — email, VoIP, video conferencing
- **Data Centralization** — servers store and manage data centrally
- **Reliability** — redundant paths ensure data still flows if one path fails
- **Scalability** — networks can grow by adding nodes

### Cross-Questions & Answers

**Q1: What is the difference between bandwidth and throughput?**
> **A:** Bandwidth is the theoretical maximum capacity of a link (like a highway's lane count), while throughput is the actual data successfully transferred (actual cars moving). Throughput ≤ Bandwidth due to errors, overhead, and congestion.

**Q2: What is latency, and what causes it?**
> **A:** Latency is the delay between sending and receiving data. Causes include: propagation delay (physical distance), transmission delay (packet size/bandwidth ratio), processing delay (router processing), and queuing delay (congestion).

**Q3: Can throughput exceed bandwidth?**
> **A:** No. Throughput can never exceed bandwidth. It is always lower due to protocol overhead, noise, collisions, and retransmissions.

---

## 2. Types of Networks

### Based on Scale

#### PAN — Personal Area Network
- Range: ~10 meters
- Used by one person's devices
- Examples: Bluetooth between phone & earbuds, USB connections
- Technologies: Bluetooth, Infrared, Zigbee

#### LAN — Local Area Network
- Range: Building or campus (up to ~1 km)
- High speed: 100 Mbps to 10 Gbps
- Examples: Office network, home Wi-Fi
- Technologies: Ethernet (IEEE 802.3), Wi-Fi (IEEE 802.11)

#### MAN — Metropolitan Area Network
- Range: City or town (~50 km)
- Examples: Cable TV networks, city-wide Wi-Fi
- Technologies: DOCSIS, SONET, WiMAX

#### WAN — Wide Area Network
- Range: Countries, continents, global
- Lower speed than LAN, higher latency
- Example: The Internet, MPLS enterprise networks
- Technologies: MPLS, Frame Relay, Leased lines, SD-WAN

#### WLAN — Wireless LAN
- LAN using wireless radio signals (Wi-Fi)
- IEEE 802.11 standards (a/b/g/n/ac/ax)

#### SAN — Storage Area Network
- Dedicated high-speed network for storage devices
- Uses Fibre Channel (FC) or iSCSI
- Appears as local disk to servers

#### VPN (as network type) — see Section 22

### Based on Ownership

| Type | Description |
|------|-------------|
| **Public Network** | Open access (Internet) |
| **Private Network** | Restricted access (corporate intranet) |
| **Hybrid Network** | Mix of public and private |

### Cross-Questions & Answers

**Q1: What differentiates a MAN from a WAN?**
> **A:** A MAN covers a city or metropolitan area (up to ~50 km) and is typically operated by a single organization or ISP. A WAN spans much larger areas—countries or continents—and uses leased lines or public infrastructure. MANs have less latency and higher speed than WANs.

**Q2: Why is a SAN separate from a regular LAN?**
> **A:** SANs use dedicated, high-speed, low-latency protocols (Fibre Channel) optimized for block-level storage I/O. Mixing storage traffic with regular LAN traffic would create congestion and latency issues for storage operations.

**Q3: Can a home network be considered a LAN?**
> **A:** Yes. A home network with a router, multiple PCs, phones, and smart devices connected via Wi-Fi or Ethernet is a LAN — a small private one.

---

## 3. Network Topology

Topology describes the **arrangement of nodes and links** in a network — both physical (actual layout) and logical (data flow path).

### Bus Topology

```
[PC1]---[PC2]---[PC3]---[PC4]
         (Single backbone cable)
```

- All nodes share a single cable (the bus/backbone)
- Data travels in both directions; terminators at ends prevent signal bounce
- **Pros:** Simple, cheap, easy to install
- **Cons:** Single point of failure (cable break = whole network down), performance degrades as nodes increase, difficult to troubleshoot
- Used in: Early Ethernet (10Base2, 10Base5) — now obsolete

### Star Topology

```
      [PC1]
       |
[PC2]--[Switch/Hub]--[PC3]
       |
      [PC4]
```

- All nodes connect to a **central hub or switch**
- **Pros:** Easy to add/remove nodes, single node failure doesn't affect others, easy troubleshooting
- **Cons:** Central device is single point of failure; more cable required
- Used in: **Most modern LANs** (Ethernet with switches)

### Ring Topology

```
[PC1]--[PC2]
 |           |
[PC4]--[PC3]
```

- Nodes form a closed loop; data travels in one direction (or both in dual ring)
- Uses **token passing** — only the token holder can transmit
- **Pros:** Equal access, no collisions
- **Cons:** Single break disrupts network (unless dual ring); adding nodes disrupts the ring
- Used in: Token Ring (IEEE 802.5), SONET/SDH (dual ring for fault tolerance)

### Mesh Topology

```
[PC1]--[PC2]
 |  \  / |
 |   \/  |
 |   /\  |
[PC4]--[PC3]
```

- Every node connects to **every other node** (full mesh) or some (partial mesh)
- **Full mesh connections:** n(n-1)/2 links for n nodes
- **Pros:** Highly redundant, fault tolerant, no single point of failure
- **Cons:** Expensive (many cables/ports), complex configuration
- Used in: WAN backbones, critical military/financial networks

### Tree (Hierarchical) Topology

- Root switch → secondary switches → end nodes
- Combines bus and star concepts
- **Pros:** Scalable, easy to manage hierarchically
- **Cons:** Root/backbone failure affects large portions
- Used in: Enterprise campus networks

### Hybrid Topology

- Combination of two or more topologies
- Example: Star-Bus, Star-Ring
- Most real-world enterprise networks are hybrid

### Cross-Questions & Answers

**Q1: What topology does the modern internet most resemble?**
> **A:** Partial mesh. Core routers in the internet backbone have multiple redundant links to other routers, providing fault tolerance, but not every router is connected to every other — that would be impractical at scale.

**Q2: In a star topology, if the central switch fails, what happens?**
> **A:** The entire network segment goes down. All nodes lose connectivity because every path goes through the central switch. This is the primary weakness of star topology — the switch is a single point of failure.

**Q3: Why is full mesh not used in LANs?**
> **A:** For n nodes, full mesh requires n(n-1)/2 connections. For 100 nodes that's 4,950 connections — prohibitively expensive and complex. Instead, we use switches in a star topology which provides similar benefits at far lower cost.

**Q4: What is the formula for connections in a full mesh?**
> **A:** n(n-1)/2, where n = number of nodes. For 5 nodes: 5×4/2 = 10 connections.

---

## 4. Network Devices

### Hub

- **OSI Layer:** Layer 1 (Physical)
- A "dumb" repeater that broadcasts incoming data to **all ports**
- No intelligence — can't distinguish between devices
- Creates one large collision domain (all devices compete for bandwidth)
- Half-duplex only
- **Obsolete** — replaced by switches

### Switch

- **OSI Layer:** Layer 2 (Data Link) — some are Layer 3 (Network)
- Maintains a **MAC Address Table** (also called CAM table) mapping MAC → port
- Forwards frames **only to the destination port** (unicast)
- Creates separate collision domains per port
- Full-duplex support
- **Layer 3 Switch:** Can also route between VLANs (routing + switching)

**How a Switch Learns:**
1. When a frame arrives, switch reads the **source MAC** and maps it to the incoming port
2. If destination MAC is known → forward to that port only
3. If destination MAC is unknown → **flood** to all ports (except source)
4. Over time, the MAC table fills up (learning process)

### Router

- **OSI Layer:** Layer 3 (Network)
- Routes packets between **different networks** using IP addresses
- Maintains a **routing table** (static routes or learned via routing protocols)
- Connects LAN to WAN (e.g., your home to the internet)
- Performs NAT (Network Address Translation)
- Separates broadcast domains

### Bridge

- **OSI Layer:** Layer 2
- Connects two network segments, filtering traffic based on MAC addresses
- Largely replaced by switches (a switch is a multi-port bridge)

### Repeater

- **OSI Layer:** Layer 1
- Regenerates and amplifies signals to extend cable distance
- No intelligence — forwards everything

### Access Point (AP)

- **OSI Layer:** Layer 2
- Provides wireless connectivity (Wi-Fi) to wired network
- Bridges wireless and wired segments

### Firewall

- **OSI Layer:** Layer 3–7 (depending on type)
- Filters traffic based on rules (IP, port, protocol, application)
- Types: Packet filter, Stateful, Application-layer (proxy), NGFW

### Gateway

- Connects networks using **different protocols** (e.g., TCP/IP to IBM SNA)
- Operates at all layers
- Your home router acts as a default gateway for your LAN

### Modem

- **MOdulator-DEModulator**
- Converts digital signals to analog (for transmission over phone lines) and back
- Types: DSL modem, Cable modem, Fiber ONT

### NIC — Network Interface Card

- Hardware adapter that connects a device to a network
- Has a unique **MAC address** burned in by manufacturer
- Can be wired (Ethernet) or wireless (Wi-Fi)

### Cross-Questions & Answers

**Q1: What is the difference between a hub and a switch?**
> **A:** A hub broadcasts all incoming data to every port (Layer 1, no intelligence), creating one collision domain. A switch uses MAC addresses to forward data only to the intended port (Layer 2), creating separate collision domains per port. Switches are far more efficient and support full-duplex.

**Q2: What is a collision domain vs. a broadcast domain?**
> **A:** A **collision domain** is a network segment where simultaneous transmissions collide. Each switch port is its own collision domain. A **broadcast domain** is where broadcast frames reach all nodes — separated by routers. All ports on a switch share one broadcast domain (unless VLANs are configured).

**Q3: Can a switch replace a router?**
> **A:** A basic Layer 2 switch cannot. Routers handle Layer 3 (IP routing), NAT, and inter-network communication. However, a **Layer 3 switch** can route between VLANs, but still relies on a router for internet-bound traffic and WAN connectivity.

**Q4: What happens if a switch's MAC table is full?**
> **A:** The switch enters **fail-open** mode and behaves like a hub — flooding all frames to all ports. This is exploited in **MAC flooding attacks** (e.g., using tools like macof) to turn a switch into a hub and enable sniffing.

**Q5: What is the difference between a router and a gateway?**
> **A:** A router connects networks using the same protocol family (IP). A gateway can connect networks using completely different protocols or architectures. In practice, "default gateway" typically refers to the router that handles traffic leaving your local network.

---

## 5. OSI Model

The **Open Systems Interconnection (OSI) model** is a conceptual framework that standardizes network communication into **7 layers**. It was developed by ISO in 1984. Each layer has specific responsibilities and communicates with the layers directly above and below it.

> **Mnemonic (Top to Bottom):** "All People Seem To Need Data Processing"
> **Mnemonic (Bottom to Top):** "Please Do Not Throw Sausage Pizza Away"

### Layer 7 — Application Layer

- **Role:** Interface between network and applications
- **What it does:** Provides network services directly to end-user applications
- **Protocols:** HTTP, HTTPS, FTP, SMTP, DNS, SNMP, Telnet, SSH
- **Data unit:** Data/Message
- **Examples:** Web browser, email client

### Layer 6 — Presentation Layer

- **Role:** Data translation, encryption, compression
- **What it does:** Ensures data is in a readable format; translates between formats
- **Functions:** Encoding (ASCII ↔ EBCDIC), encryption/decryption (SSL/TLS), compression
- **Protocols:** SSL/TLS (historically placed here), JPEG, MPEG, ASCII, EBCDIC
- **Data unit:** Data

### Layer 5 — Session Layer

- **Role:** Manages sessions (conversations) between applications
- **What it does:** Establishes, maintains, and terminates sessions; handles synchronization
- **Functions:** Session establishment, checkpointing, recovery
- **Protocols:** NetBIOS, RPC, PPTP
- **Data unit:** Data

### Layer 4 — Transport Layer

- **Role:** End-to-end communication, reliability
- **What it does:** Segments data, manages flow control, error correction, port addressing
- **Protocols:** **TCP** (reliable, connection-oriented), **UDP** (unreliable, connectionless)
- **Key concepts:** Port numbers, segmentation, reassembly, 3-way handshake (TCP), flow control (sliding window), congestion control
- **Data unit:** **Segment** (TCP) / **Datagram** (UDP)

### Layer 3 — Network Layer

- **Role:** Logical addressing and routing
- **What it does:** Determines best path for data; adds source/destination IP addresses
- **Protocols:** **IP (IPv4/IPv6)**, ICMP, OSPF, BGP, RIP
- **Devices:** Routers, Layer 3 switches
- **Data unit:** **Packet**

### Layer 2 — Data Link Layer

- **Role:** Node-to-node delivery within same network
- **What it does:** Frames data, adds MAC addresses, error detection (CRC), media access control
- **Sub-layers:**
  - **LLC (Logical Link Control):** Error and flow control
  - **MAC (Media Access Control):** Physical addressing, collision avoidance
- **Protocols:** Ethernet, Wi-Fi (802.11), PPP, HDLC, ARP
- **Devices:** Switches, bridges, NICs
- **Data unit:** **Frame**

### Layer 1 — Physical Layer

- **Role:** Bit transmission over physical medium
- **What it does:** Defines electrical/optical/radio signals, cable types, connector types, bit rate
- **Protocols/Standards:** Ethernet (cable specs), USB, Bluetooth, RS-232
- **Devices:** Hubs, repeaters, cables, NICs (physical part)
- **Data unit:** **Bits**

### OSI Layer Summary Table

| Layer | Name | PDU | Device | Key Protocol |
|-------|------|-----|--------|-------------|
| 7 | Application | Data | — | HTTP, DNS, FTP |
| 6 | Presentation | Data | — | TLS, JPEG |
| 5 | Session | Data | — | NetBIOS, RPC |
| 4 | Transport | Segment/Datagram | — | TCP, UDP |
| 3 | Network | Packet | Router | IP, ICMP |
| 2 | Data Link | Frame | Switch | Ethernet, ARP |
| 1 | Physical | Bits | Hub, Repeater | RS-232, 802.3 |

### Encapsulation & Decapsulation

**Encapsulation (Sender):** Data moves DOWN the stack; each layer adds its own **header** (and sometimes trailer):
```
Application:   [Data]
Transport:     [TCP Header][Data]
Network:       [IP Header][TCP Header][Data]
Data Link:     [Frame Header][IP Header][TCP Header][Data][Frame Trailer]
Physical:      10101010... (bits)
```

**Decapsulation (Receiver):** Data moves UP the stack; each layer strips its header.

### Cross-Questions & Answers

**Q1: At which OSI layer does SSL/TLS operate?**
> **A:** Technically SSL/TLS spans Layers 4–7. Historically it's associated with Layer 6 (Presentation) for encryption/decryption, but it actually operates between the Application and Transport layers — sometimes called "Layer 4.5." In modern discussions, TLS is considered to operate at Layer 4 or 5.

**Q2: What is the difference between a segment, packet, and frame?**
> **A:** These are Protocol Data Units (PDUs) at different layers. A **segment** (Layer 4) contains TCP/UDP headers + data. A **packet** (Layer 3) adds IP headers to the segment. A **frame** (Layer 2) adds MAC address headers and CRC trailer to the packet.

**Q3: ARP operates at which layer?**
> **A:** ARP operates at Layer 2 (Data Link) conceptually but uses Layer 3 (IP addresses) as input. It bridges Layer 2 and Layer 3. In the OSI model, it's often placed between Layers 2 and 3.

**Q4: Why does the OSI model have 7 layers while TCP/IP has only 4?**
> **A:** OSI is a theoretical, vendor-neutral reference model designed for standardization. TCP/IP was developed practically for the internet and collapses OSI Layers 5, 6, 7 into one Application layer, and Layers 1 and 2 into one Network Access layer. OSI provides finer-grained conceptual separation; TCP/IP is the practical implementation.

**Q5: What device operates at ALL 7 layers?**
> **A:** A **gateway** or **proxy server** or **application-layer firewall** (NGFW). These inspect and process data at every layer from Physical to Application.

---

## 6. TCP/IP Model

The **Transmission Control Protocol / Internet Protocol** model is the practical networking model used by the Internet. It was developed by DARPA in the 1970s. It has **4 layers** (some texts describe 5).

### TCP/IP vs. OSI Mapping

| TCP/IP Layer | Corresponds to OSI Layers | Protocols |
|---|---|---|
| Application | 5, 6, 7 | HTTP, FTP, DNS, SMTP, SSH |
| Transport | 4 | TCP, UDP |
| Internet | 3 | IP, ICMP, ARP, IGMP |
| Network Access (Link) | 1, 2 | Ethernet, Wi-Fi, PPP |

### Layer 1 — Network Access (Link) Layer

- Combines OSI Physical + Data Link
- Handles hardware addressing (MAC), framing, media access
- **Protocols:** Ethernet, Wi-Fi (802.11), ARP

### Layer 2 — Internet Layer

- Logical addressing and routing
- **Key Protocol: IP (IPv4/IPv6)**
- ICMP: Error messages and diagnostics (ping uses ICMP Echo Request/Reply)
- IGMP: Multicast group management

### Layer 3 — Transport Layer

#### TCP — Transmission Control Protocol
- **Connection-oriented:** 3-way handshake before data transfer
- **Reliable:** Acknowledgments (ACKs), retransmission of lost packets
- **Ordered delivery:** Sequence numbers ensure correct order
- **Flow control:** Sliding window mechanism
- **Congestion control:** Slow start, congestion avoidance
- **Use cases:** HTTP/HTTPS, FTP, SSH, email

**TCP 3-Way Handshake:**
```
Client          Server
  |--SYN-------->|   Client sends SYN (SEQ=x)
  |<--SYN-ACK----|   Server replies SYN-ACK (SEQ=y, ACK=x+1)
  |--ACK-------->|   Client confirms ACK (ACK=y+1)
  [Connection Established]
```

**TCP 4-Way Termination:**
```
Client          Server
  |--FIN-------->|
  |<----ACK------|
  |<----FIN------|
  |-----ACK----->|
  [Connection Closed]
```

#### UDP — User Datagram Protocol
- **Connectionless:** No handshake
- **Unreliable:** No acknowledgments, no retransmission
- **Fast:** Low overhead, low latency
- **Use cases:** DNS, DHCP, VoIP, video streaming, online gaming

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Yes | No |
| Reliability | Yes | No |
| Order | Yes | No |
| Speed | Slower | Faster |
| Header size | 20 bytes | 8 bytes |
| Use case | HTTP, FTP, SSH | DNS, VoIP, gaming |

### Layer 4 — Application Layer

- Combines OSI Session, Presentation, Application
- All user-facing protocols operate here

### Cross-Questions & Answers

**Q1: What is a TCP SYN flood attack?**
> **A:** An attacker sends many SYN packets with spoofed source IPs. The server sends SYN-ACK and waits for ACK (which never comes, as the source IPs are fake). The server's connection table fills up (SYN backlog exhausted), causing denial of service. **Mitigation:** SYN cookies, rate limiting, firewalls.

**Q2: Why does DNS use UDP but falls back to TCP?**
> **A:** DNS queries and small responses use UDP (port 53) for speed. If the response exceeds 512 bytes (or 4096 bytes with EDNS), DNS falls back to TCP for reliability and to handle large zone transfers.

**Q3: Can TCP guarantee delivery?**
> **A:** TCP guarantees **best-effort reliable delivery within a single connection** — it will retransmit until acknowledged. However, if the network is truly broken or the other side crashes, TCP will eventually give up (timeout) and report an error to the application. It doesn't guarantee absolute delivery.

**Q4: What is TCP sequence number used for?**
> **A:** Sequence numbers track the order of bytes. The receiver uses them to reassemble out-of-order segments and identify which data has been received, which allows selective retransmission.

---

## 7. MAC Address

### What is a MAC Address?

A **Media Access Control (MAC) address** is a unique **48-bit hardware identifier** assigned to a NIC by its manufacturer. It operates at OSI Layer 2.

**Format:** `AA:BB:CC:DD:EE:FF` (6 pairs of hex digits)

```
|  OUI (24 bits)  |  Device ID (24 bits)  |
| AA : BB : CC    |  DD : EE : FF         |
```

- **OUI (Organizationally Unique Identifier):** First 3 bytes identify the manufacturer
- **Device ID:** Last 3 bytes are assigned by the manufacturer (unique per device)

### Properties

| Property | Detail |
|----------|--------|
| Length | 48 bits (6 bytes) |
| Format | XX:XX:XX:XX:XX:XX or XX-XX-XX-XX-XX-XX |
| Scope | **Local network only** — not routable |
| Burned-in | Stored in NIC firmware (ROM) |
| Can be spoofed | Yes — software can override MAC |

### Special MAC Addresses

| MAC | Purpose |
|-----|---------|
| `FF:FF:FF:FF:FF:FF` | Broadcast — sent to all devices |
| `01:xx:xx:xx:xx:xx` | Multicast (LSB of first byte = 1) |
| Globally unique | Bit 1 of byte 1 = 0 |
| Locally administered | Bit 1 of byte 1 = 1 |

### How MAC is Used

1. **ARP (Address Resolution Protocol):** Maps IP address → MAC address within a LAN
   - Device A wants to reach 192.168.1.10
   - Sends ARP broadcast: "Who has 192.168.1.10?"
   - Device with that IP replies: "I have it, my MAC is AA:BB:CC:DD:EE:FF"
   - Device A stores this in its **ARP cache**

2. **Switch MAC Table:** Switches learn MAC-to-port mappings to forward frames intelligently

### Cross-Questions & Answers

**Q1: Can two devices have the same MAC address?**
> **A:** In theory, no — MACs are globally unique. In practice, **MAC spoofing** allows a device to impersonate another's MAC. If two devices have the same MAC on the same network, it causes a conflict (the switch gets confused, toggling between ports for that MAC).

**Q2: Is MAC address permanent?**
> **A:** The **burned-in address (BIA)** is permanent in firmware, but the **effective MAC** can be overridden in software (MAC spoofing). Modern OSes also use **MAC randomization** for Wi-Fi probing to protect privacy.

**Q3: Does a MAC address travel across routers?**
> **A:** No. MAC addresses are local to each network segment. When a packet crosses a router, the router replaces the source and destination MAC addresses with its own interfaces' MACs. The IP addresses remain unchanged across routers (that's routing's job).

**Q4: What is ARP poisoning/spoofing?**
> **A:** An attacker sends fake ARP replies, associating their MAC with a legitimate IP (e.g., the gateway). Victims update their ARP cache with the false mapping, so traffic meant for the gateway goes to the attacker — enabling a **Man-in-the-Middle (MITM)** attack.

---

## 8. IPv4 Addressing

### What is IPv4?

**IPv4 (Internet Protocol version 4)** uses **32-bit addresses**, providing ~4.3 billion unique addresses. Written in **dotted-decimal notation**: four octets separated by dots.

```
192.168.10.1
|   |   | |
Octet1.Octet2.Octet3.Octet4
Each octet = 8 bits (0–255)
```

### Address Classes (Classful)

| Class | First Bits | Range | Default Subnet Mask | Use |
|-------|-----------|-------|--------------------|----|
| **A** | 0xxxxxxx | 1.0.0.0 – 126.255.255.255 | /8 (255.0.0.0) | Large networks |
| **B** | 10xxxxxx | 128.0.0.0 – 191.255.255.255 | /16 (255.255.0.0) | Medium networks |
| **C** | 110xxxxx | 192.0.0.0 – 223.255.255.255 | /24 (255.255.255.0) | Small networks |
| **D** | 1110xxxx | 224.0.0.0 – 239.255.255.255 | N/A | Multicast |
| **E** | 1111xxxx | 240.0.0.0 – 255.255.255.255 | N/A | Reserved/Research |

### Private IP Ranges (RFC 1918)

These addresses are non-routable on the internet (used in private networks):

| Range | CIDR | Class |
|-------|------|-------|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | A |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | B |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | C |

### Special Addresses

| Address | Purpose |
|---------|---------|
| `0.0.0.0` | Default route / "this network" |
| `127.0.0.1` | Loopback (localhost) |
| `169.254.x.x` | APIPA (link-local, no DHCP found) |
| `255.255.255.255` | Limited broadcast |
| Host bits all 0 | Network address |
| Host bits all 1 | Directed broadcast |

### IPv4 Packet Header

Key fields:
- **Version (4 bits):** Always 4 for IPv4
- **IHL (4 bits):** Internet Header Length
- **DSCP/ECN (8 bits):** QoS marking
- **Total Length (16 bits):** Header + data
- **TTL (8 bits):** Time To Live — decremented at each router; packet dropped when TTL=0
- **Protocol (8 bits):** Layer 4 protocol (TCP=6, UDP=17, ICMP=1)
- **Source IP (32 bits):** Sender's IP
- **Destination IP (32 bits):** Recipient's IP

### Cross-Questions & Answers

**Q1: What is the purpose of TTL?**
> **A:** TTL (Time To Live) prevents packets from circulating infinitely in routing loops. Each router decrements TTL by 1. When TTL reaches 0, the router discards the packet and sends an ICMP "Time Exceeded" message back to the sender. The `traceroute` command exploits this behavior.

**Q2: What is APIPA and when is it assigned?**
> **A:** APIPA (Automatic Private IP Addressing) assigns a 169.254.x.x/16 address when a DHCP server is unreachable. The device picks a random address in this range and can only communicate with other APIPA hosts on the same segment (no internet access).

**Q3: Why are we running out of IPv4 addresses?**
> **A:** IPv4 has 2^32 ≈ 4.3 billion addresses. With billions of internet-connected devices (phones, IoT, PCs), this pool is exhausted. Solutions include: NAT (sharing one public IP among many private devices), IPv6 (128-bit addresses), and address reclamation.

---

## 9. IPv6 Addressing

### Why IPv6?

IPv4 address exhaustion drove IPv6's development. **IPv6 uses 128-bit addresses**, providing 2^128 ≈ **340 undecillion** addresses — enough for every grain of sand on Earth (many times over).

### IPv6 Address Format

```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
|    |    |    |    |    |    |    |
16-bit groups (hextets) separated by colons
```

**Simplification Rules:**
1. Leading zeros in a group can be omitted: `0db8` → `db8`
2. Consecutive all-zero groups can be replaced with `::` (only once)
   - `2001:0db8:0000:0000:0000:0000:0000:0001` → `2001:db8::1`

### IPv6 Address Types

| Type | Description | Example |
|------|-------------|---------|
| **Unicast** | One-to-one | `2001:db8::1` |
| **Multicast** | One-to-many | `FF00::/8` |
| **Anycast** | One-to-nearest | `2001:db8::/48` (anycast) |
| **Link-Local** | Local segment only | `FE80::/10` |
| **Loopback** | Self (= 127.0.0.1) | `::1` |
| **Global Unicast** | Publicly routable | `2000::/3` |

> IPv6 **has no broadcast** — replaced by multicast.

### IPv6 vs IPv4 Key Differences

| Feature | IPv4 | IPv6 |
|---------|------|------|
| Address size | 32 bits | 128 bits |
| Address format | Dotted-decimal | Colon-hex |
| Total addresses | ~4.3 billion | ~340 undecillion |
| Header size | 20–60 bytes | Fixed 40 bytes |
| NAT needed? | Yes | No |
| Broadcast | Yes | No (uses multicast) |
| DHCP | Optional | Optional (SLAAC available) |
| IPsec | Optional | Built-in |
| Fragmentation | Routers & hosts | Hosts only |

### SLAAC — Stateless Address Autoconfiguration

- IPv6 devices can **self-configure** without DHCP
- Process:
  1. Device generates link-local address (FE80:: + interface ID)
  2. Sends Router Solicitation (RS) to routers
  3. Router replies with Router Advertisement (RA) containing network prefix
  4. Device combines prefix + its MAC (modified EUI-64) = Global Unicast Address

### Cross-Questions & Answers

**Q1: What replaces broadcast in IPv6?**
> **A:** Multicast. IPv6 uses specific multicast groups instead of broadcast. For example, "all nodes" multicast is `FF02::1`, and "all routers" is `FF02::2`. Neighbor Discovery Protocol (NDP) replaces ARP using multicast.

**Q2: What is the IPv6 equivalent of ARP?**
> **A:** **NDP (Neighbor Discovery Protocol)**, which uses ICMPv6 messages. Specifically, **Neighbor Solicitation** and **Neighbor Advertisement** messages replace ARP Request/Reply.

**Q3: Can IPv4 and IPv6 coexist?**
> **A:** Yes, via **dual-stack** (device runs both protocols), **tunneling** (IPv6 encapsulated in IPv4), or **translation** (NAT64 converts between IPv4 and IPv6). Most modern networks are dual-stack during the transition period.

**Q4: What is EUI-64?**
> **A:** EUI-64 is a method to generate a 64-bit interface identifier from a 48-bit MAC address. The MAC is split in half, `FFFE` is inserted in the middle, and the 7th bit is flipped. This creates a unique interface ID for SLAAC.

---

## 10. Protocols Overview

A **protocol** is a set of rules defining how data is formatted, transmitted, and received. Here's a high-level map before deep dives:

| Protocol | Layer | Port | Purpose |
|----------|-------|------|---------|
| HTTP | 7 | 80 | Web browsing |
| HTTPS | 7 | 443 | Encrypted web |
| FTP | 7 | 20/21 | File transfer |
| SSH | 7 | 22 | Secure remote access |
| Telnet | 7 | 23 | Unsecure remote access |
| SMTP | 7 | 25 | Send email |
| DNS | 7 | 53 | Name resolution |
| DHCP | 7 | 67/68 | IP assignment |
| TFTP | 7 | 69 | Simple file transfer (UDP) |
| HTTP | 7 | 80 | Web |
| POP3 | 7 | 110 | Receive email |
| NTP | 7 | 123 | Time sync |
| NetBIOS | 7 | 137–139 | Windows naming |
| IMAP | 7 | 143 | Email access |
| SNMP | 7 | 161/162 | Network management |
| LDAP | 7 | 389 | Directory services |
| HTTPS | 7 | 443 | Encrypted web |
| SMB | 7 | 445 | File sharing (Windows) |
| RDP | 7 | 3389 | Remote desktop |
| ICMP | 3 | — | Diagnostics (ping, traceroute) |
| NFS | 7 | 2049 | Network file system |

---

## 11. Subnetting

### What is Subnetting?

**Subnetting** divides a large IP network into smaller **sub-networks (subnets)**. This improves:
- **Security** — isolate departments/segments
- **Performance** — reduce broadcast domain size
- **IP efficiency** — allocate only needed IPs

### Subnet Mask

A **subnet mask** tells which part of an IP is the **network portion** and which is the **host portion**.

```
IP:      192.168.1.0
Mask:    255.255.255.0  →  /24 in CIDR
Binary:  11111111.11111111.11111111.00000000
         |--- Network (24 bits) ---|Host|
```

- `/24` means 24 bits for network → 8 bits for hosts → 2^8 - 2 = **254 usable hosts**
- Subtract 2: network address (all host bits 0) + broadcast address (all host bits 1)

### CIDR — Classless Inter-Domain Routing

CIDR replaced classful addressing, allowing flexible prefix lengths.

**Key formula:**
- **Number of subnets (if borrowing n bits):** 2^n
- **Hosts per subnet:** 2^h - 2 (h = host bits remaining)
- **Total hosts in address space:** 2^h

### Subnet Mask Reference Table

| CIDR | Subnet Mask | # Hosts | # Usable Hosts |
|------|------------|---------|----------------|
| /8 | 255.0.0.0 | 16,777,216 | 16,777,214 |
| /16 | 255.255.0.0 | 65,536 | 65,534 |
| /24 | 255.255.255.0 | 256 | 254 |
| /25 | 255.255.255.128 | 128 | 126 |
| /26 | 255.255.255.192 | 64 | 62 |
| /27 | 255.255.255.224 | 32 | 30 |
| /28 | 255.255.255.240 | 16 | 14 |
| /29 | 255.255.255.248 | 8 | 6 |
| /30 | 255.255.255.252 | 4 | 2 |
| /31 | 255.255.255.254 | 2 | 0 (point-to-point use) |
| /32 | 255.255.255.255 | 1 | Host route only |

### Subnetting Example — Step by Step

**Problem:** Network 192.168.10.0/24. Create 4 subnets.

**Step 1:** We need 4 subnets → need 2^n ≥ 4 → n=2 (borrow 2 bits)

**Step 2:** New prefix = 24 + 2 = **/26**

**Step 3:** Host bits remaining = 32 - 26 = 6 → 2^6 - 2 = **62 usable hosts per subnet**

**Step 4:** Block size = 256 - 192 = 64 (the 4th octet increments by 64)

| Subnet | Network | First Host | Last Host | Broadcast |
|--------|---------|------------|-----------|-----------|
| 1 | 192.168.10.0/26 | 192.168.10.1 | 192.168.10.62 | 192.168.10.63 |
| 2 | 192.168.10.64/26 | 192.168.10.65 | 192.168.10.126 | 192.168.10.127 |
| 3 | 192.168.10.128/26 | 192.168.10.129 | 192.168.10.190 | 192.168.10.191 |
| 4 | 192.168.10.192/26 | 192.168.10.193 | 192.168.10.254 | 192.168.10.255 |

### VLSM — Variable Length Subnet Masking

VLSM allows subnets of **different sizes** within the same network — more efficient IP allocation.

**Example:** 3 departments needing 50, 25, and 10 hosts respectively.
- Dept A (50 hosts) → /26 (62 usable)
- Dept B (25 hosts) → /27 (30 usable)
- Dept C (10 hosts) → /28 (14 usable)

### Cross-Questions & Answers

**Q1: Given IP 172.16.45.14/20, find network address, broadcast, first & last host.**

> **A:**
> - /20 → 20 network bits, 12 host bits
> - In the 3rd octet: 20-16=4 bits used → 2^4=16 block size
> - 45 / 16 = 2 remainder 13 → network = 2×16 = 32
> - Network: **172.16.32.0**
> - Broadcast: **172.16.47.255** (32+16-1=47, last octet 255)
> - First host: **172.16.32.1**
> - Last host: **172.16.47.254**

**Q2: How many subnets and hosts does /27 give from a /24?**
> **A:** Borrowing 3 bits (27-24=3): 2^3 = **8 subnets**. Host bits remaining = 5: 2^5-2 = **30 usable hosts per subnet**.

**Q3: What is a supernet?**
> **A:** The opposite of subnetting — aggregating multiple networks into a larger one. Example: 192.168.0.0/24 and 192.168.1.0/24 can be summarized as 192.168.0.0/23. Used in **route summarization** to reduce routing table size.

---

## 12. VLAN — Virtual Local Area Network

### What is a VLAN?

A **VLAN** logically segments a physical network into multiple isolated **broadcast domains** without requiring separate physical switches. VLANs are configured on managed switches using **IEEE 802.1Q** tagging.

### Why VLANs?

- **Security:** Isolate sensitive departments (Finance, HR)
- **Performance:** Limit broadcast traffic to relevant devices only
- **Flexibility:** Users in different physical locations can be in the same VLAN
- **Simplified management:** Group users logically, not physically

### How VLANs Work

Each **port** on a switch is assigned to a VLAN (Access mode) or can carry multiple VLANs (Trunk mode):

- **Access port:** Carries traffic for **one VLAN** only (end devices)
- **Trunk port:** Carries traffic for **multiple VLANs** (switch-to-switch or switch-to-router links)

### IEEE 802.1Q Tagging

When a frame crosses a trunk link, a **4-byte 802.1Q tag** is inserted into the Ethernet header:

```
| Dest MAC | Src MAC | 802.1Q Tag | EtherType | Data | FCS |
                      |TPID|PCP|DEI|VLAN ID|
                      2B    3b  1b   12 bits
```

- **TPID:** Tag Protocol ID = 0x8100
- **PCP:** Priority Code Point (QoS, 0–7)
- **DEI:** Drop Eligible Indicator
- **VLAN ID:** 12 bits → 4096 VLANs (0 and 4095 reserved → 4094 usable)

### Default VLAN

- **VLAN 1** is the default VLAN on all switches (all ports belong to VLAN 1 by default)
- Best practice: Don't use VLAN 1 for user traffic; reassign and disable it

### Native VLAN

- The VLAN whose traffic is sent **untagged** across a trunk
- Must match on both ends of a trunk (mismatch = VLAN hopping vulnerability)
- Default native VLAN = VLAN 1

### Inter-VLAN Routing

VLANs are isolated — devices in different VLANs can't communicate without routing:

1. **Router-on-a-Stick:** Single physical router interface with multiple subinterfaces, one per VLAN
2. **Layer 3 Switch (SVIs):** Switch Virtual Interfaces — each VLAN gets a virtual Layer 3 interface

### VLAN Security Threats

| Attack | Description | Mitigation |
|--------|-------------|-----------|
| **VLAN Hopping** | Attacker sends double-tagged frames to access other VLANs | Change native VLAN, disable auto-trunking |
| **Switch Spoofing** | Attacker makes port become trunk using DTP | Disable DTP, manually configure access ports |
| **Native VLAN Mismatch** | Traffic leaks between VLANs | Consistent native VLAN config |

### Cross-Questions & Answers

**Q1: Can devices in different VLANs communicate?**
> **A:** Not directly. VLANs are separate broadcast domains at Layer 2. Communication between VLANs requires a Layer 3 device (router or Layer 3 switch) to route packets between them.

**Q2: What is the difference between an access port and a trunk port?**
> **A:** An **access port** belongs to one VLAN and doesn't use 802.1Q tags — used for end devices. A **trunk port** carries multiple VLANs simultaneously using 802.1Q tags — used between switches, or between a switch and a router.

**Q3: What is VLAN hopping, and how is it performed?**
> **A:** VLAN hopping allows an attacker to access traffic from VLANs they shouldn't. Two methods: (1) **Switch Spoofing** — attacker enables DTP to negotiate a trunk with the switch; (2) **Double Tagging** — attacker sends a frame tagged with their VLAN and an inner tag for the target VLAN; the outer tag is stripped at the first switch, and the inner tag carries traffic to the target VLAN. Prevention: disable DTP, change native VLAN to unused VLAN, use PVLAN.

---

## 13. DHCP — Dynamic Host Configuration Protocol

### What is DHCP?

**DHCP** (RFC 2131) automatically assigns IP addresses and network configuration to devices, eliminating manual configuration. Uses **UDP** (port 67 server, port 68 client).

### DORA Process — 4-Step Handshake

```
Client                              Server
  |--DISCOVER (broadcast)---------->|  "I need an IP!"
  |<--OFFER (unicast/broadcast)-----|  "Here's 192.168.1.10"
  |--REQUEST (broadcast)----------->|  "I'll take 192.168.1.10"
  |<--ACK (unicast/broadcast)-------|  "Confirmed! Lease granted"
```

1. **DISCOVER:** Client broadcasts (0.0.0.0 → 255.255.255.255) looking for DHCP server
2. **OFFER:** Server offers an available IP with lease time, subnet mask, gateway, DNS
3. **REQUEST:** Client broadcasts the chosen IP (may be multiple DHCP servers)
4. **ACK:** Server acknowledges and finalizes the lease

### DHCP Lease

- IP is **leased** for a time period (lease time)
- At 50% of lease: client tries to **renew** with unicast DHCP Request
- At 87.5% of lease: client broadcasts another DHCP Request
- At 100%: lease expires; client must start DORA again

### DHCP Parameters Provided

- IP Address
- Subnet Mask
- Default Gateway
- DNS Server(s)
- Lease Duration
- Domain Name
- NTP server

### DHCP Relay Agent

In multi-subnet environments, DHCP broadcasts don't cross routers. A **DHCP relay agent** (ip helper-address on router) forwards DHCP broadcasts to the DHCP server via unicast.

### DHCP Security Issues

| Threat | Description | Mitigation |
|--------|-------------|-----------|
| **Rogue DHCP Server** | Attacker runs own DHCP server, gives wrong gateway/DNS | **DHCP Snooping** (switch only trusts DHCP from designated port) |
| **DHCP Starvation** | Attacker exhausts the IP pool with fake requests | Rate limiting, DHCP Snooping, port security |
| **DHCP Spoofing** | Attacker intercepts DHCP messages | DHCP Snooping |

### Cross-Questions & Answers

**Q1: What is DHCP snooping?**
> **A:** A Layer 2 security feature on switches that filters DHCP messages. Ports are classified as **trusted** (DHCP server port) or **untrusted** (client ports). DHCP offers from untrusted ports are dropped, preventing rogue DHCP servers.

**Q2: What happens when a DHCP lease expires?**
> **A:** The client loses its IP address and must restart the DORA process. If no server responds, the client may self-assign an APIPA address (169.254.x.x).

**Q3: What is a DHCP reservation?**
> **A:** Also called a static DHCP assignment. The server is configured to always assign a specific IP to a specific MAC address. The device still uses DHCP but always gets the same IP — combining DHCP's ease with static IP's predictability.

---

## 14. DNS — Domain Name System

### What is DNS?

**DNS** (RFC 1034/1035) translates human-readable **domain names** (www.google.com) into **IP addresses** (142.250.x.x). Called the "phone book of the internet." Uses **UDP port 53** (TCP for responses > 512 bytes and zone transfers).

### DNS Hierarchy

```
. (Root)
├── com
│   ├── google
│   │   └── www
│   └── amazon
├── org
├── net
└── in
```

From right to left:
1. **Root (.):** 13 root server clusters worldwide
2. **TLD (Top-Level Domain):** .com, .org, .net, .in, .uk
3. **Second-Level Domain:** google, amazon
4. **Subdomain/Hostname:** www, mail, ftp

### DNS Resolution — Full Process

```
User → Browser → DNS Resolver (ISP/8.8.8.8) → Root Server → TLD Server → Authoritative Server → IP
```

1. User types `www.google.com`
2. Browser checks local **DNS cache** → not found
3. OS checks `/etc/hosts` or `C:\Windows\System32\drivers\etc\hosts` → not found
4. Query sent to **Recursive Resolver** (configured via DHCP — e.g., 8.8.8.8)
5. Resolver checks its cache → not found
6. Resolver queries **Root Server** → "I don't know, ask .com TLD server at x.x.x.x"
7. Resolver queries **.com TLD server** → "Ask Google's authoritative server at x.x.x.x"
8. Resolver queries **Google's Authoritative Server** → "www.google.com = 142.250.x.x"
9. Resolver caches and returns IP to client
10. Browser connects to IP

### DNS Record Types

| Record | Purpose | Example |
|--------|---------|---------|
| **A** | IPv4 address | `google.com → 142.250.80.46` |
| **AAAA** | IPv6 address | `google.com → 2607:f8b0::` |
| **CNAME** | Canonical name (alias) | `www.google.com → google.com` |
| **MX** | Mail server | `google.com → aspmx.l.google.com` |
| **NS** | Name server | `google.com → ns1.google.com` |
| **PTR** | Reverse DNS (IP → name) | `142.250.80.46 → google.com` |
| **SOA** | Start of Authority | Zone info, serial, refresh |
| **TXT** | Text (SPF, DKIM, verification) | `"v=spf1 include:..."` |
| **SRV** | Service location | `_sip._tcp.example.com` |

### DNS Attacks

| Attack | Description | Mitigation |
|--------|-------------|-----------|
| **DNS Spoofing/Cache Poisoning** | Inject false DNS records into cache | DNSSEC, source port randomization |
| **DNS Amplification** | Use open resolvers for DDoS amplification | Disable open resolvers, BCP38 |
| **DNS Tunneling** | Encode data in DNS queries (C2 channel) | DNS traffic analysis, NTA |
| **NXDOMAIN attack** | Flood non-existent domains to exhaust resolvers | Rate limiting, RPZ |
| **Typosquatting** | Register lookalike domains | User education, DNS filtering |

### Cross-Questions & Answers

**Q1: What is the difference between a recursive resolver and an authoritative server?**
> **A:** A **recursive resolver** (like 8.8.8.8) does the legwork — it queries other servers on behalf of the client, traversing the DNS hierarchy. An **authoritative server** holds the actual DNS zone records and gives definitive answers for its domain. It doesn't recurse further.

**Q2: What is DNSSEC?**
> **A:** DNS Security Extensions adds **cryptographic signatures** to DNS records. Each DNS response is signed with a private key; the client verifies using the public key. This prevents cache poisoning because forged responses won't have valid signatures.

**Q3: What is a zone transfer?**
> **A:** Zone transfers (AXFR/IXFR) replicate DNS zone data from a **primary** to a **secondary** DNS server. If not restricted, an attacker can query AXFR to enumerate all DNS records for a domain — a major information disclosure risk. **Mitigation:** Restrict AXFR to known secondary IPs.

---

## 15. FTP — File Transfer Protocol

### What is FTP?

**FTP** (RFC 959) is an application-layer protocol for transferring files between client and server. Uses **TCP port 21** (control) and **port 20** (data in active mode).

### FTP Modes

#### Active Mode
```
Client → Server:21 (Control: "Send me the file")
Server:20 → Client:random port (Data connection)
```
- Server **initiates** the data connection back to client
- Problem: Client firewalls often block incoming connections from server port 20

#### Passive Mode (PASV)
```
Client → Server:21 (Control: "I want passive mode")
Server tells client: "Connect to me on port XXXX"
Client → Server:XXXX (Data connection)
```
- Client initiates **both** connections
- **Preferred** — works through firewalls and NAT

### FTP Variants

| Protocol | Port | Encryption | Description |
|----------|------|-----------|-------------|
| **FTP** | 21/20 | None | Plain text — insecure |
| **FTPS** | 990 (implicit) / 21 (explicit) | SSL/TLS | FTP over TLS |
| **SFTP** | 22 | SSH | File transfer over SSH (not FTP over SSH — different protocol) |
| **TFTP** | 69 | None | Trivial FTP, UDP, no auth, simple (used in PXE boot, router configs) |

### FTP Security Concerns

- Credentials and data transmitted in **plaintext** — vulnerable to sniffing
- Anonymous FTP: allows login with username "anonymous" — minimal security
- **FTP bounce attack:** Attacker uses server's PORT command to scan/attack third parties
- Use **SFTP or FTPS** in production

### Cross-Questions & Answers

**Q1: What is the difference between SFTP and FTPS?**
> **A:** **SFTP** (SSH File Transfer Protocol) is a completely separate protocol built on SSH — it uses a single port (22) and encrypts everything. **FTPS** is FTP with TLS/SSL encryption added — it uses multiple ports (21 for control, additional ports for data). SFTP is generally simpler to firewall and more widely supported.

**Q2: Why does FTP use two separate connections?**
> **A:** The **control connection** (port 21) remains open for commands (LIST, GET, PUT). The **data connection** (port 20 in active) is opened per-transfer and closed when done. Separating them allows command interaction while data transfers are in progress, and is part of FTP's original design.

**Q3: What is TFTP used for?**
> **A:** TFTP (Trivial FTP, UDP port 69) is used for lightweight transfers where simplicity matters over features — like booting diskless workstations (PXE), transferring router configurations, or delivering firmware. It has no authentication, no directory listing, and minimal error handling.

---

## 16. NFS — Network File System

### What is NFS?

**NFS** (developed by Sun Microsystems, RFC 1094/3530) allows a client to **mount a remote filesystem** and access it as if it were local. Primarily used in **Unix/Linux** environments. Uses **TCP/UDP port 2049** (NFSv3+).

### NFS Architecture

```
[NFS Client]                [NFS Server]
mount /mnt/share ← → /etc/exports: /data 192.168.1.0/24(rw,sync)
                            /data directory exposed
```

- **Server exports** directories via `/etc/exports`
- **Client mounts** exported directories using `mount` command
- Data is accessed using RPC (Remote Procedure Calls)

### NFS Versions

| Version | Features |
|---------|---------|
| **NFSv2** | UDP only, 32-bit file sizes |
| **NFSv3** | TCP/UDP, large files, async writes |
| **NFSv4** | Stateful, single TCP port 2049, integrated security (Kerberos), POSIX ACLs |

### NFS Security

- NFSv2/v3 trust IP-based authentication — inherently insecure
- **NFSv4 with Kerberos** provides strong authentication
- **Firewall exports** to specific IP ranges only
- Avoid `no_root_squash` — it allows root on client to act as root on server

### Cross-Questions & Answers

**Q1: What is the no_root_squash option and why is it dangerous?**
> **A:** By default, NFS uses **root squash** — client root is mapped to an anonymous user (nfsnobody) on the server. `no_root_squash` disables this, allowing root on the client to have root privileges on the server's exported filesystem. This is extremely dangerous because any compromised client with root can compromise the server.

**Q2: How does NFS differ from SMB?**
> **A:** NFS is primarily Unix/Linux-based; SMB is Windows-native. NFS uses RPC; SMB uses its own protocol stack. NFS mounts feel more like local filesystems; SMB provides richer features like printer sharing, named pipes, and Windows ACLs. Both can cross-platform operate (NFS client on Windows, Samba/SMB on Linux).

---

## 17. SMB — Server Message Block

### What is SMB?

**SMB** (Server Message Block) is Microsoft's file-sharing protocol for **Windows networks**. It enables file access, printer sharing, serial ports, and named pipes. Uses **TCP port 445** (direct) and **NetBIOS on 137-139** (legacy).

### SMB Versions

| Version | OS | Security |
|---------|----|----|
| **SMBv1** | Windows XP/2003 | Highly vulnerable (WannaCry!) |
| **SMBv2** | Windows Vista/2008 | Improved performance |
| **SMBv2.1** | Windows 7/2008 R2 | Opportunistic locking |
| **SMBv3** | Windows 8/2012+ | Encryption, multichannel |
| **SMBv3.1.1** | Windows 10/2016+ | Pre-auth integrity |

### Samba

**Samba** is the open-source implementation of SMB for Linux/Unix systems, allowing Linux to act as a Windows file/print server and join Active Directory domains.

### SMB Security Vulnerabilities

| Vulnerability | CVE | Description |
|--------------|-----|-------------|
| **EternalBlue** | MS17-010 | SMBv1 remote code execution (used by WannaCry) |
| **SMB Relay** | — | Attacker relays authentication to another server |
| **Pass the Hash** | — | Reuse NTLM hash without knowing password |
| **Null Sessions** | — | Anonymous connections to enumerate shares |

**Critical:** **Disable SMBv1** — it has no viable security and multiple RCE exploits.

### Cross-Questions & Answers

**Q1: What is the WannaCry ransomware, and how did SMB relate?**
> **A:** WannaCry (2017) used the **EternalBlue** exploit (NSA-developed, leaked by Shadow Brokers) targeting a buffer overflow in SMBv1 (MS17-010). It spread laterally across networks infecting millions of Windows machines without user interaction. Patch: KB4012212 / disable SMBv1.

**Q2: What is an SMB relay attack?**
> **A:** When a victim authenticates to a malicious server (e.g., via a spoofed share), the attacker captures the NTLM authentication challenge/response and **relays it** to another legitimate server, authenticating as the victim. **Mitigation:** SMB signing (makes relay fail), LDAP signing, tiered admin accounts.

---

## 18. SSH — Secure Shell

### What is SSH?

**SSH** (RFC 4251) provides **encrypted remote access** to systems over an unsecured network. It replaced Telnet, rlogin, and rsh. Uses **TCP port 22**.

### SSH Architecture

SSH has three sub-protocols:
1. **Transport Layer Protocol:** Key exchange, server authentication, encryption, compression
2. **User Authentication Protocol:** Authenticates client to server
3. **Connection Protocol:** Multiplexes the encrypted tunnel into channels (terminal, file transfer, port forwarding)

### Authentication Methods

| Method | Description | Security |
|--------|-------------|---------|
| **Password** | Username + password | Moderate (vulnerable to brute force) |
| **Public Key** | Asymmetric key pair | Strong (recommended) |
| **Keyboard-interactive** | Challenge-response | Variable |
| **Certificate** | CA-signed keys | Strongest (enterprise) |

### Public Key Authentication

```
1. Generate key pair: ssh-keygen -t ed25519
   → Private key: ~/.ssh/id_ed25519 (KEEP SECRET)
   → Public key:  ~/.ssh/id_ed25519.pub

2. Copy public key to server:
   ssh-copy-id user@server
   → Appended to ~/.ssh/authorized_keys on server

3. SSH connection:
   Client proves possession of private key → Server verifies against public key → Authenticated
```

### Key Exchange — How SSH Secures Connection

1. Client connects → Server sends its **host key** (RSA/ECDSA/Ed25519)
2. Client verifies host key (prevents MITM) — first time: user warned (TOFU — Trust on First Use)
3. **Diffie-Hellman key exchange** → both sides derive the same session key without transmitting it
4. All subsequent communication encrypted with symmetric key (AES-256)

### SSH Tunneling / Port Forwarding

- **Local forwarding:** Tunnel a remote service to local port
  `ssh -L 8080:internal-server:80 user@jump-host`
- **Remote forwarding:** Expose local service through remote server
  `ssh -R 9090:localhost:3000 user@remote`
- **Dynamic (SOCKS proxy):** Full SOCKS proxy through SSH
  `ssh -D 1080 user@remote`

### Cross-Questions & Answers

**Q1: What is the difference between SSH and SSL/TLS?**
> **A:** Both provide encrypted communication but serve different purposes. **SSH** is primarily for interactive remote access (terminal, file transfer) and uses its own protocol stack. **SSL/TLS** is a general-purpose cryptographic protocol used by HTTPS, SMTPS, etc. SSH is application-specific; TLS is used to secure many protocols.

**Q2: Why is public key authentication better than password auth for SSH?**
> **A:** Public key auth is immune to brute-force and password-spraying attacks. The private key never leaves the client. Even if the server is compromised, the private key isn't exposed. Password auth requires transmitting credentials (even encrypted) and is susceptible to weak passwords.

**Q3: What is the known_hosts file?**
> **A:** Located at `~/.ssh/known_hosts`, it stores the fingerprints of SSH servers the client has connected to. On subsequent connections, SSH verifies the server's key matches the stored fingerprint — preventing MITM attacks. A changed fingerprint triggers a warning.

**Q4: How does SSH prevent MITM during first connection?**
> **A:** It doesn't fully — SSH uses **TOFU (Trust On First Use)**. The first time you connect, you accept the server's fingerprint on faith. After that, it's stored in known_hosts and verified. For true MITM prevention on first connect, use SSH certificates with a trusted CA.

---

## 19. RDP — Remote Desktop Protocol

### What is RDP?

**RDP** (Remote Desktop Protocol, Microsoft proprietary) provides **graphical remote access** to Windows systems. Uses **TCP (and sometimes UDP) port 3389**. Transmits full desktop sessions — keyboard, mouse, screen.

### How RDP Works

1. Client sends connection request to server:3389
2. Negotiation: encryption, authentication methods, compression
3. **NLA (Network Level Authentication):** Pre-authenticates before session setup — more secure
4. Session established; client receives graphical desktop stream
5. All input (keyboard/mouse) encrypted and sent to server
6. Output (screen) sent back compressed and encrypted

### RDP Security Features

| Feature | Description |
|---------|-------------|
| **TLS Encryption** | Encrypts all RDP traffic |
| **NLA** | Requires credential before desktop loads (reduces attack surface) |
| **RD Gateway** | Tunnels RDP over HTTPS — avoids exposing port 3389 |
| **MFA** | Two-factor authentication (Azure AD, Duo) |

### RDP Security Risks

| Risk | Description | Mitigation |
|------|-------------|-----------|
| **BlueKeep (CVE-2019-0708)** | Wormable RCE in RDP (Windows 7/2008) | Patch, disable RDP if unused |
| **DejaBlue** | Similar to BlueKeep for newer Windows | Patch |
| **Brute Force** | Password attacks on port 3389 | Account lockout, NLA, MFA |
| **Exposed to internet** | Port 3389 scanned constantly | Use VPN or RD Gateway, firewall |
| **Pass-the-Hash** | Authenticate with NTLM hash | Protected Users group, credential guard |

### Cross-Questions & Answers

**Q1: What is NLA in RDP and why is it important?**
> **A:** Network Level Authentication requires the client to authenticate **before** a full RDP session is established. Without NLA, unauthenticated users reach the Windows login screen — vulnerable to exploitation (BlueKeep exploits this). NLA significantly reduces the attack surface by pre-validating credentials.

**Q2: Should RDP port 3389 be exposed to the internet?**
> **A:** No. RDP on port 3389 is a massive attack surface — constantly scanned and brute-forced. Best practices: place RDP behind a **VPN** or **RD Gateway** (which tunnels RDP over HTTPS/443), enable NLA, use MFA, and restrict access by IP.

---

## 20. Telnet

### What is Telnet?

**Telnet** (RFC 854) is a network protocol providing **command-line remote access** to systems. Uses **TCP port 23**. Completely **unencrypted** — sends everything (including passwords) in plaintext.

### Why Telnet is Dangerous

```
Telnet session (as seen by attacker on same network):
→ User types: password123
→ Network packet: "p", "a", "s", "s", "w", "o", "r", "d", "1", "2", "3"
→ Attacker captures packet → has credentials
```

- All data transmitted as **plain ASCII** — trivially sniffed with Wireshark
- No integrity protection — data can be modified in transit
- No server authentication — MITM attacks possible

### Telnet vs SSH

| Feature | Telnet | SSH |
|---------|--------|-----|
| Port | 23 | 22 |
| Encryption | None | Strong (AES-256) |
| Authentication | Password (plaintext) | Password + Public key |
| Usage today | Deprecated / limited lab use | Industry standard |
| MITM protection | None | Yes (host key verification) |

### Legitimate Uses Today

- Testing connectivity to services (e.g., `telnet mail.server.com 25` to test SMTP)
- Legacy devices (old switches, printers with no SSH support)
- Debugging protocols

### Cross-Questions & Answers

**Q1: If Telnet is so insecure, why does it still exist?**
> **A:** Legacy hardware (old routers, industrial equipment) may only support Telnet. It's also useful for manually testing/debugging services on specific ports (connecting to TCP port 25 to test SMTP). In isolated lab environments with no sniffing risk, it can still be used. But it should **never** be used over untrusted networks.

**Q2: Can Telnet be made secure?**
> **A:** Not inherently — its design lacks encryption. Workarounds include tunneling Telnet over SSH (`ssh -L 2323:device:23 user@jump`) or using a VPN to encrypt the link. But the proper solution is simply to use SSH instead.

---

## 21. NAT — Network Address Translation

### What is NAT?

**NAT** (RFC 1631) translates **private IP addresses to public IP addresses** (and vice versa) at the router level. It allows many devices with private IPs to share a single public IP address — critical for IPv4 address conservation.

### Types of NAT

#### Static NAT (1:1)
- One private IP permanently maps to one public IP
- Used for servers that need consistent external access
```
192.168.1.10 ↔ 203.0.113.10 (always)
```

#### Dynamic NAT (Many:Many)
- Pool of public IPs; private IPs mapped to public IPs from pool dynamically
- Less common

#### PAT — Port Address Translation (NAT Overload / Masquerade)
- **Most common type** (used in homes and offices)
- Many private IPs share **one public IP** — differentiated by **port numbers**

```
Inside                          Outside
192.168.1.10:52341 → 203.0.113.1:52341 → internet
192.168.1.11:52342 → 203.0.113.1:52342 → internet
192.168.1.12:52343 → 203.0.113.1:52343 → internet
```

Router maintains a **NAT table:**
| Inside Local | Inside Global | Outside |
|---|---|---|
| 192.168.1.10:12345 | 203.0.113.1:40001 | 8.8.8.8:53 |
| 192.168.1.11:54321 | 203.0.113.1:40002 | 93.184.216.34:80 |

### NAT Process (Outbound)
1. Inside device sends packet: `192.168.1.10:12345 → 8.8.8.8:53`
2. Router translates source: `203.0.113.1:40001 → 8.8.8.8:53` and records in NAT table
3. Server responds to `203.0.113.1:40001`
4. Router looks up table → translates back to `192.168.1.10:12345`

### NAT and IPv6

NAT was a **workaround** for IPv4 exhaustion. IPv6's massive address space eliminates the need for NAT — every device gets a public IPv6 address. However, NAT provides some security by hiding internal network topology.

### NAT Security Implications

- **Hides internal network** — outside cannot initiate connections to inside devices (unless port forwarding is configured)
- **Not a true security mechanism** — a firewall should always be used alongside NAT
- **Port forwarding** (DNAT) exposes internal services to the internet

### Cross-Questions & Answers

**Q1: What is the difference between NAT and PAT?**
> **A:** NAT is the general concept of translating IP addresses. PAT is a specific type of NAT that also translates port numbers, allowing many private IPs to share a single public IP (many-to-one). PAT is sometimes called NAT Overload. Most home routers use PAT.

**Q2: Does NAT provide security?**
> **A:** NAT provides **security through obscurity** — it hides internal IPs from the internet and prevents unsolicited inbound connections. But it's not a substitute for a firewall. A NAT device without a proper firewall can still be exploited via port forwarding, UPnP, or through compromised inside devices.

**Q3: What is hairpin NAT?**
> **A:** Also called NAT loopback or reflection. It allows an internal device to access an internal server using its **public IP** (rather than the private IP). Without it, internal devices may fail to connect to the server when using the public DNS name.

---

## 22. VPN — Virtual Private Network

### What is a VPN?

A **VPN** creates an **encrypted tunnel** over an untrusted network (like the internet), making remote devices appear as if they're on the local network. Used for remote access, site-to-site connectivity, and privacy.

### Types of VPN

#### Remote Access VPN
- Individual users connect to a corporate network remotely
- User's device acts as if it's on the corporate LAN
- Example: Employee working from home connecting to office

#### Site-to-Site VPN
- Connects two entire networks securely over the internet
- Routers on each side handle the tunnel — individual hosts don't need VPN clients
- Example: Branch office connected to headquarters

#### Client VPN (Consumer)
- Routes user's internet traffic through a VPN server
- Hides IP from websites; protects traffic on public Wi-Fi
- Examples: NordVPN, ExpressVPN

### VPN Protocols

| Protocol | Port | Encryption | Speed | Security |
|----------|------|-----------|-------|---------|
| **OpenVPN** | 1194 (UDP) / 443 (TCP) | AES-256 | Good | Strong, open-source |
| **WireGuard** | 51820 (UDP) | ChaCha20 | Fastest | Very strong, modern |
| **IPsec/IKEv2** | 500/4500 (UDP) | AES-256 | Good | Strong, mobile-friendly |
| **L2TP/IPsec** | 1701/500/4500 | AES-256 | Moderate | Good (L2TP alone = none) |
| **PPTP** | 1723 | RC4/MPPE | Fast | **Broken — DO NOT USE** |
| **SSL VPN** | 443 | TLS | Good | Firewall-friendly |

### IPsec — IP Security

IPsec is a suite of protocols securing IP communications. Used in many VPN implementations:

- **AH (Authentication Header):** Provides integrity and authentication (no encryption)
- **ESP (Encapsulating Security Payload):** Provides encryption + integrity + authentication
- **IKE (Internet Key Exchange):** Negotiates security associations and keys

**Modes:**
- **Transport Mode:** Encrypts only the payload (used between two hosts)
- **Tunnel Mode:** Encrypts entire packet + new IP header (used in VPNs)

### Split Tunneling

- **Full tunnel:** All traffic goes through VPN (secure but slow)
- **Split tunnel:** Only corporate traffic goes through VPN; internet traffic goes direct (faster, less secure)

### Cross-Questions & Answers

**Q1: What is the difference between a VPN and a proxy?**
> **A:** A **proxy** routes specific application traffic (usually HTTP) through an intermediary — it's not encrypted by default and doesn't handle all traffic. A **VPN** encrypts **all** traffic at the network level, creating a secure tunnel. VPNs are more comprehensive; proxies are faster for specific use cases.

**Q2: Is WireGuard better than OpenVPN?**
> **A:** WireGuard is significantly faster (less CPU overhead) and has a much smaller codebase (~4,000 lines vs OpenVPN's ~600,000) — easier to audit for security. OpenVPN is more mature and configurable. WireGuard is generally preferred for new deployments.

**Q3: What is a "No-Log" VPN, and can you trust it?**
> **A:** A no-log VPN claims not to store user activity logs. Trust is limited — you must rely on the provider's word, jurisdiction, and independent audits. VPN providers have been compelled by courts to hand over data (or proven to log despite claims). Open-source solutions (WireGuard, OpenVPN self-hosted) offer more transparency.

**Q4: What is Perfect Forward Secrecy (PFS)?**
> **A:** PFS uses **ephemeral key exchange** — a new session key is generated for each session (and even each session renegotiation). If a long-term private key is compromised, past sessions cannot be decrypted because each had a unique, temporary key that's since been discarded. Essential for modern VPN and HTTPS security.

---

## 23. NTA — Network Traffic Analysis

### What is NTA?

**Network Traffic Analysis (NTA)** is the process of **intercepting, recording, and analyzing network traffic** to understand behavior, detect anomalies, diagnose problems, and identify threats. It's a core capability in security operations (SOC) and network management.

### NTA Components

#### Capture Methods
| Method | Description |
|--------|-------------|
| **Port mirroring (SPAN)** | Switch copies all traffic from one port/VLAN to a monitoring port |
| **TAP (Test Access Point)** | Hardware device that passively copies traffic off the wire |
| **NetFlow/IPFIX** | Router/switch exports flow summaries (not full packets) |
| **sFlow** | Statistical sampling of traffic flows |
| **Packet capture (PCAP)** | Full packet capture using tools like tcpdump, Wireshark |

#### Flow Data vs. Full Packet Capture

| Aspect | Flow Data (NetFlow) | Full Packet Capture |
|--------|--------------------|--------------------|
| **What's captured** | Metadata only (src/dst IP, port, bytes, duration) | Every bit of every packet |
| **Storage** | Low | Very high |
| **Privacy** | Less sensitive | Contains full payload |
| **Use case** | Trend analysis, anomaly detection | Deep forensics, protocol analysis |

### NTA Tools

| Tool | Type | Use |
|------|------|-----|
| **Wireshark** | GUI packet analyzer | Protocol analysis, debugging |
| **tcpdump** | CLI packet capture | Command-line capture |
| **Zeek (Bro)** | Network security monitor | Log generation, threat detection |
| **Snort/Suricata** | IDS/IPS | Signature-based detection |
| **Nmap** | Port scanner | Network discovery |
| **Elastic Stack (ELK)** | SIEM/analytics | Log aggregation, visualization |
| **Darktrace/Vectra** | AI-based NTA | Behavioral anomaly detection |
| **ntopng** | Flow analyzer | Real-time traffic visualization |

### Key NTA Concepts

#### Baseline Behavior
Establish normal traffic patterns — then **anomalies** stand out:
- Normal DNS query rate: 100/min → anomaly: 10,000/min (possible DNS tunneling)
- Normal bandwidth: 100 Mbps → spike to 1 Gbps at 3am (possible data exfiltration)

#### IOCs — Indicators of Compromise
Evidence in network traffic suggesting a breach:
- Connections to known malicious IPs (C2 servers)
- DNS queries to DGA (Domain Generation Algorithm) domains
- Unusual outbound traffic on non-standard ports
- Large data transfers to external IPs
- Repeated authentication failures

#### Common Detected Threats

| Threat | Network Signature |
|--------|-----------------|
| **Port Scanning** | Many connections to sequential ports from single source |
| **DDoS** | Massive traffic from many sources to single target |
| **Lateral Movement** | Internal SMB/SSH connections to many hosts |
| **Data Exfiltration** | Large outbound transfers to unusual IPs |
| **C2 Communication** | Periodic beaconing to external IPs (regular intervals) |
| **DNS Tunneling** | Unusually long DNS queries, high frequency |
| **Ransomware** | Mass file access + encryption patterns |

### Protocol Analysis with Wireshark

```
# Capture SSH traffic:
tcp.port == 22

# Find all DNS queries:
dns.flags.response == 0

# Find large HTTP transfers:
http && tcp.len > 1000

# Follow TCP stream to see conversation:
Right-click → Follow → TCP Stream

# Filter by IP:
ip.addr == 192.168.1.100
```

### NTA in Security Operations

1. **Detection:** Alert on anomalies using IDS rules or ML models
2. **Investigation:** Drill into packet captures for forensic evidence
3. **Threat Hunting:** Proactively search for IOCs without prior alert
4. **Incident Response:** Reconstruct attack timelines from PCAP evidence
5. **Compliance:** Demonstrate audit trails for PCI-DSS, HIPAA, etc.

### Cross-Questions & Answers

**Q1: What is beaconing in network traffic, and how is it detected?**
> **A:** Beaconing is the regular, periodic "check-in" of malware with its Command & Control (C2) server — e.g., every 60 seconds. It appears in traffic as regular intervals of small outbound connections to the same external IP. Detection: statistical analysis of connection timing, jitter analysis, flow-based tools like Zeek. Anomaly: perfectly regular intervals suggest automation, not human browsing.

**Q2: What is the difference between IDS and IPS?**
> **A:** An **IDS (Intrusion Detection System)** monitors and alerts — it's passive, doesn't block traffic. An **IPS (Intrusion Prevention System)** is inline — it monitors AND can actively block malicious traffic. IDS generates logs/alerts; IPS also drops packets or resets connections. IPS requires more careful tuning to avoid false positives blocking legitimate traffic.

**Q3: How does SSL/TLS encryption affect NTA?**
> **A:** Encrypted traffic (HTTPS, SSH, TLS) hides the payload from NTA tools — you can see metadata (IP, port, size, timing) but not content. Techniques to analyze encrypted traffic: **SSL inspection** (MITM proxy — decrypts, inspects, re-encrypts — requires certificate trust), **JA3 fingerprinting** (fingerprints TLS handshake parameters to identify clients/malware), and **traffic behavior analysis** (packet size, timing patterns, certificate attributes).

**Q4: What is a PCAP file?**
> **A:** A **PCAP (Packet CAPture)** file is the standard format for storing raw network packet data. Captured by tools like Wireshark (`libpcap`) or tcpdump. Contains complete packet data including headers and payload. Used for forensic analysis, incident response, and protocol debugging. Next-gen format: **PCAPng** (better metadata, multiple interfaces).

**Q5: What is deep packet inspection (DPI)?**
> **A:** DPI examines the actual content (payload) of packets — not just headers. It can identify applications regardless of port (e.g., BitTorrent on port 80), detect malware signatures in payload, inspect for data loss prevention (DLP), and enforce application-layer policies. Used in NGFWs, ISP traffic management, and corporate content filtering.

---

## 📋 Quick Reference: Port Numbers

| Port | Protocol | Service |
|------|----------|---------|
| 20 | TCP | FTP Data |
| 21 | TCP | FTP Control |
| 22 | TCP | SSH |
| 23 | TCP | Telnet |
| 25 | TCP | SMTP |
| 53 | TCP/UDP | DNS |
| 67 | UDP | DHCP Server |
| 68 | UDP | DHCP Client |
| 69 | UDP | TFTP |
| 80 | TCP | HTTP |
| 110 | TCP | POP3 |
| 123 | UDP | NTP |
| 137-139 | TCP/UDP | NetBIOS |
| 143 | TCP | IMAP |
| 161/162 | UDP | SNMP |
| 389 | TCP | LDAP |
| 443 | TCP | HTTPS |
| 445 | TCP | SMB |
| 500 | UDP | IKE (IPsec) |
| 514 | UDP | Syslog |
| 636 | TCP | LDAPS |
| 993 | TCP | IMAPS |
| 995 | TCP | POP3S |
| 1194 | UDP | OpenVPN |
| 1433 | TCP | MSSQL |
| 2049 | TCP/UDP | NFS |
| 3306 | TCP | MySQL |
| 3389 | TCP | RDP |
| 5432 | TCP | PostgreSQL |
| 8080 | TCP | HTTP Alt |
| 51820 | UDP | WireGuard |

---

## 🔑 Key Formulas Summary

| Formula | Use |
|---------|-----|
| 2^h - 2 | Usable hosts per subnet (h = host bits) |
| 2^n | Number of subnets (n = borrowed bits) |
| 256 - subnet mask octet | Block size |
| n(n-1)/2 | Full mesh connections (n = nodes) |
| 32 - CIDR prefix | Host bits in IPv4 |
| 128 - CIDR prefix | Host bits in IPv6 |

---

## 📚 Final Cross-Topic Questions

**Q: Trace a web request from typing www.example.com to receiving the page, covering all layers.**
> **A:** 
> 1. **Application (L7):** Browser initiates HTTP/HTTPS GET request for www.example.com
> 2. **DNS Resolution:** DNS query (UDP:53) → recursive resolver → authoritative server → IP returned
> 3. **Transport (L4):** TCP 3-way handshake to server IP on port 443; TLS handshake for HTTPS
> 4. **Network (L3):** IP packet created with src (your IP) and dst (server IP); routing table consulted → sent to default gateway
> 5. **Data Link (L2):** Frame created; ARP resolves gateway IP to MAC; Ethernet frame to gateway
> 6. **Physical (L1):** Bits transmitted over cable/Wi-Fi
> 7. **NAT:** Router translates your private IP to public IP (PAT)
> 8. **Internet:** Packet travels through multiple routers (BGP routing) to reach server
> 9. **Server side:** Reverse process; HTTP response generated
> 10. **Response:** Returns through same path; NAT translates back; browser renders HTML

**Q: How would you secure a corporate network end-to-end?**
> **A:** Layered defense: Perimeter (firewall, IPS), network segmentation (VLANs), access control (802.1X), encrypted remote access (VPN/SSH), patch management, endpoint protection, NTA/SIEM for detection, DNS filtering, MFA everywhere, principle of least privilege, and regular penetration testing.

---

*Guide compiled by Claude | Covers all major networking fundamentals for education and certification prep (CompTIA Network+, CCNA, Security+)*
