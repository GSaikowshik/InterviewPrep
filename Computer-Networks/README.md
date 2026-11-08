# Computer Networks Master Reference & Systems Guide

---

## 1. Network Models: OSI 7-Layer vs. TCP/IP 4-Layer

```mermaid
graph LR
    subgraph OSI["OSI 7-Layer Reference Model"]
        L7["7. Application (HTTP, DNS, SSH)"]
        L6["6. Presentation (SSL/TLS, JPEG, ASCII)"]
        L5["5. Session (Sockets, RPC, NetBIOS)"]
        L4["4. Transport (TCP, UDP)"]
        L3["3. Network (IP, ICMP, OSPF, BGP)"]
        L2["2. Data Link (Ethernet, MAC, ARP, VLAN)"]
        L1["1. Physical (Cables, Fiber, Radio/Wi-Fi)"]
    end

    subgraph TCPIP["TCP/IP 4-Layer Model"]
        T4["Application Layer (Data)"]
        T3["Transport Layer (Segments / Datagrams)"]
        T2["Internet Layer (Packets)"]
        T1["Network Access / Link Layer (Frames & Bits)"]
    end

    L7 --> T4
    L6 --> T4
    L5 --> T4
    L4 --> T3
    L3 --> T2
    L2 --> T1
    L1 --> T1
```

### Protocol Data Units (PDU) & Network Devices

| Layer # | OSI Layer Name | Protocol Data Unit (PDU) | Core Protocols | Hardware Devices | Primary Function |
|---|---|---|---|---|---|
| **7** | Application | Data | HTTP, HTTPS, DNS, SMTP, SSH, FTP, WebSocket | Gateway, Application Proxy | Network process to application interface |
| **6** | Presentation | Data | TLS/SSL, JSON, XML, JPEG, gzip | Gateway | Data formatting, encryption, and compression |
| **5** | Session | Data | Sockets, NetBIOS, RPC | Gateway | Session establishment, maintenance, and teardown |
| **4** | Transport | Segment (TCP) / Datagram (UDP)| TCP, UDP, QUIC, SCTP | Layer 4 Load Balancer, Firewall | End-to-end communication, flow/congestion control |
| **3** | Network | Packet | IPv4, IPv6, ICMP, ARP, OSPF, BGP | Router, Layer 3 Switch | Logical addressing, routing across internetworks |
| **2** | Data Link | Frame | Ethernet (802.3), Wi-Fi (802.11), PPP, VLAN | Switch, Bridge, NIC | Node-to-node frame transfer via physical MAC |
| **1** | Physical | Bits | 1000BASE-T, V.90, Fiber Optics | Hub, Repeater, Modem, Cables | Raw bitstream transmission over physical media |

---

## 2. Transport Layer: TCP vs. UDP Deep Dive

```mermaid
graph TD
    TL["Transport Layer Protocols"] --> TCP["Transmission Control Protocol (TCP)<br>- Connection-oriented (Handshake)<br>- Reliable (Guaranteed delivery & ordering)<br>- Flow & Congestion controlled<br>- Overhead: 20-60 byte header"]
    TL --> UDP["User Datagram Protocol (UDP)<br>- Connectionless (Fire-and-forget)<br>- Unreliable (Best-effort, no ACK/order)<br>- No flow/congestion overhead<br>- Minimal Overhead: 8 byte header"]

    TCP --> TCPApps["Web (HTTP/HTTPS), Email (SMTP), File Transfer (FTP), Shell (SSH)"]
    UDP --> UDPApps["DNS Queries, Live Video Streaming (WebRTC), VoIP, Online Gaming, HTTP/3 (QUIC)"]
```

### TCP 3-Way Handshake (Connection Establishment)

```mermaid
sequenceDiagram
    autonumber
    actor Client as TCP Client
    actor Server as TCP Server

    Note over Client,Server: State: LISTEN
    Client->>Server: SYN (Seq = X)
    Note over Client: State: SYN-SENT
    Note over Server: State: SYN-RECEIVED
    Server->>Client: SYN-ACK (Seq = Y, Ack = X + 1)
    Note over Client: State: ESTABLISHED
    Client->>Server: ACK (Seq = X + 1, Ack = Y + 1)
    Note over Server: State: ESTABLISHED
    Note over Client,Server: Bi-directional reliable data stream active
```

### TCP 4-Way Handshake (Connection Teardown)

```mermaid
sequenceDiagram
    autonumber
    actor Client as TCP Initiator
    actor Server as TCP Receiver

    Client->>Server: FIN (Seq = u)
    Note over Client: State: FIN-WAIT-1
    Server->>Client: ACK (Ack = u + 1)
    Note over Server: State: CLOSE-WAIT
    Note over Client: State: FIN-WAIT-2
    Server->>Client: FIN (Seq = w)
    Note over Server: State: LAST-ACK
    Client->>Server: ACK (Ack = w + 1)
    Note over Client: State: TIME-WAIT (2 * MSL)
    Note over Server: State: CLOSED
    Note over Client: State: CLOSED (after timeout)
```

> **Why is `TIME-WAIT` (2MSL) necessary?** It ensures that the final ACK was received by the server (retransmitting if lost) and guarantees all lingering duplicate packets from this connection expire in the network before port reuse.

### TCP Congestion Control Mechanisms
1. **Slow Start:** Congestion window ($CWND$) starts at 1 MSS (Maximum Segment Size) and doubles every Round Trip Time (RTT) ($CWND = CWND \times 2$) exponentially until reaching the Slow-Start Threshold ($SSTHRESH$).
2. **Congestion Avoidance:** When $CWND \ge SSTHRESH$, growth becomes linear ($CWND = CWND + 1$ per RTT) to probe capacity cautiously.
3. **Fast Retransmit & Fast Recovery:** When 3 duplicate ACKs are received, TCP infers a packet was dropped without waiting for a retransmission timer, retransmits immediately, sets $SSTHRESH = CWND / 2$, and continues linear growth.

---

## 3. Network Layer, IP Addressing & Subnetting

### IPv4 vs. IPv6

| Attribute | IPv4 | IPv6 |
|---|---|---|
| **Address Length** | 32 bits (4 bytes) | 128 bits (16 bytes) |
| **Address Space** | $2^{32} \approx 4.3 \times 10^9$ addresses | $2^{128} \approx 3.4 \times 10^{38}$ addresses |
| **Format** | Dotted-decimal: `192.168.1.1` | Hexadecimal with colons: `2001:0db8:85a3::8a2e:0370:7334` |
| **Header Size** | Variable: 20 to 60 bytes | Fixed: 40 bytes (faster routing) |
| **Checksum in Header** | Yes (must be recomputed at every router) | No (eliminated; handled by L2 and L4) |
| **Configuration** | Manual DHCP / Static | Stateless Address Autoconfiguration (SLAAC) / DHCPv6 |
| **Security** | Optional IPSec | Mandatory native IPSec support |

### Subnetting & CIDR Mechanics
In Classless Inter-Domain Routing (CIDR) notation `192.168.10.0/24`:
* `/24` means the first 24 bits represent the **Network ID**, and the remaining $32 - 24 = 8$ bits represent the **Host ID**.
* **Subnet Mask:** `255.255.255.0`
* **Total IP Addresses:** $2^8 = 256$
* **Usable Host Addresses:** $2^8 - 2 = 254$
  * First address (`192.168.10.0`): **Network Address**
  * Last address (`192.168.10.255`): **Broadcast Address**

---

## 4. Application Layer Workflows

### The Complete DNS Resolution Pipeline

```mermaid
sequenceDiagram
    autonumber
    actor Client as Web Browser
    participant LocalDNS as Recursive Resolver (ISP / 8.8.8.8)
    participant RootDNS as Root Nameserver (.)
    participant TLD as TLD Nameserver (.com)
    participant AuthDNS as Authoritative Nameserver (aws.com)

    Client->>LocalDNS: "What is the IP for example.com?"
    LocalDNS->>RootDNS: "Where is .com?"
    RootDNS-->>LocalDNS: "Go to .com TLD Nameserver (IP: A.B.C.D)"
    LocalDNS->>TLD: "Where is example.com?"
    TLD-->>LocalDNS: "Go to Authoritative Nameserver (IP: W.X.Y.Z)"
    LocalDNS->>AuthDNS: "Resolve example.com A record"
    AuthDNS-->>LocalDNS: "example.com A is 93.184.216.34 (TTL: 3600)"
    LocalDNS-->>Client: Returns 93.184.216.34 (Caches record)
```

### Modern Web Protocols: HTTP/1.1 vs. HTTP/2 vs. HTTP/3

| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---|---|---|---|
| **Transport Protocol** | TCP | TCP | UDP (via QUIC) |
| **Multiplexing** | No (Head-of-Line blocking at HTTP level) | Yes (Binary framing over 1 TCP connection) | Yes (Zero Head-of-Line blocking over UDP) |
| **Header Compression** | None (Raw plaintext headers repeated) | HPACK (Static + Dynamic Huffman table) | QPACK (Optimized for out-of-order packets) |
| **Connection Handshake**| TCP (1 RTT) + TLS (1-2 RTT) = 2-3 RTT | TCP (1 RTT) + TLS 1.3 (1 RTT) = 2 RTT | QUIC Handshake = 0-RTT or 1-RTT connection |
| **Server Push** | No | Supported | Supported |

---

## 5. Network Security: HTTPS & The TLS 1.3 Handshake

```mermaid
sequenceDiagram
    autonumber
    actor Client as Browser
    actor Server as Web Server (TLS 1.3)

    Client->>Server: ClientHello (Supported Ciphers, Client Random, Key Share: g^x)
    Server->>Client: ServerHello (Selected Cipher, Server Random, Key Share: g^y)
    Server->>Client: EncryptedExtensions, Certificate (Signed by CA), CertificateVerify, Finished
    Note over Client: Validates CA Root Trust & Cert Chain
    Client->>Server: Finished
    Note over Client,Server: Symmetric AES-256-GCM Session Key Established (1-RTT)
    Client->>Server: Encrypted HTTP Application Data (GET /index.html)
```

---

## 6. Top 20 High-Yield Computer Networks Interview Questions

1. **What happens under the hood when you type `https://www.google.com` into your browser and hit enter?**
   * *Answer:*
     1. Browser checks local DNS cache (browser, OS hosts file, recursive resolver).
     2. If not cached, executes hierarchical DNS lookup (Root -> TLD -> Authoritative).
     3. Browser establishes TCP 3-Way Handshake with Google's edge IP (SYN -> SYN-ACK -> ACK).
     4. Executes TLS 1.3 cryptographic handshake to negotiate symmetric encryption keys.
     5. Client transmits HTTP `GET /` request over the secure socket.
     6. Google's Nginx/Borg reverse proxy routes request to internal services, returning HTML.
     7. Browser renders DOM/CSSOM and fires subsequent queries for assets.
2. **What is the difference between ARP and RARP?**
   * *Answer:* ARP (Address Resolution Protocol) resolves a known logical IP address to a physical hardware MAC address on a local link. RARP (Reverse ARP) resolves a known physical MAC address to an IP address (mostly superseded by DHCP).
3. **Explain the difference between a Hub, Switch, and Router.**
   * *Answer:* A **Hub** is a Layer 1 device that blindly broadcasts incoming bits to all ports (single collision domain). A **Switch** is a Layer 2 device that inspects MAC frames and forwards them only to the destination port using a MAC address table (each port is its own collision domain). A **Router** is a Layer 3 device that routes IP packets between different subnets using routing tables.
4. **What is Head-of-Line (HoL) blocking? How did HTTP/2 and HTTP/3 solve it?**
   * *Answer:* In HTTP/1.1, a slow request blocks subsequent requests on the same TCP connection. HTTP/2 solved this at the application layer with binary frame multiplexing. However, if a single TCP packet dropped at the transport layer, all streams paused until retransmitted. HTTP/3 completely eliminates HoL blocking by running over UDP via QUIC, making individual streams fully independent.

## 7. Network Topologies & Data Link Layer Protocols

```mermaid
graph TD
    Topologies["Physical & Logical Network Topologies"]
    Topologies --> Star["Star Topology<br>- All nodes connect to central switch/hub<br>- Single cable cut only drops one node<br>- Central switch is single point of failure"]
    Topologies --> Mesh["Mesh Topology (Full vs Partial)<br>- Every node connects to multiple nodes: N*(N-1)/2 links<br>- Maximum redundancy, highest cabling cost"]
    Topologies --> Bus["Bus Topology<br>- Shared backbone cable with terminators<br>- High collision rate, hard to troubleshoot"]
    Topologies --> Ring["Ring Topology<br>- Token passing protocol (FDDI, Token Ring)<br>- Deterministic latency, break disrupts ring"]
```

### Address Resolution Protocol (ARP) & Collision Domains

```mermaid
sequenceDiagram
    autonumber
    actor HostA as Host A (192.168.1.10)
    actor Switch as Layer 2 Switch
    actor HostB as Host B (192.168.1.20)

    HostA->>Switch: Broadcast Frame: "Who has 192.168.1.20? Tell 192.168.1.10 (MAC: AA:AA:AA:AA:AA:AA)"
    Switch->>HostB: Floods broadcast to all ports (FF:FF:FF:FF:FF:FF)
    HostB->>Switch: Unicast ARP Reply: "192.168.1.20 is at BB:BB:BB:BB:BB:BB"
    Switch->>HostA: Forwards unicast ARP reply
    Note over HostA: Caches mapping in ARP Table: 192.168.1.20 -> BB:BB:BB:BB:BB:BB
```

---

## 8. Dynamic Host Configuration Protocol (DHCP DORA Process)

```mermaid
sequenceDiagram
    autonumber
    actor Client as New Device (No IP)
    actor Server as DHCP Server (Router)

    Client->>Server: 1. DHCP Discover (Broadcast: 255.255.255.255:67 from 0.0.0.0:68)
    Server->>Client: 2. DHCP Offer (Unicast/Broadcast: Reserved IP 192.168.1.105, Gateway, DNS, Lease Time)
    Client->>Server: 3. DHCP Request (Broadcast: "I accept 192.168.1.105 from Server X")
    Server->>Client: 4. DHCP ACK (Unicast: Confirms lease, assigns configuration)
    Note over Client: Configures interface with 192.168.1.105 / 24
```

---

## 9. IP Routing Algorithms & NAT/PAT

| Protocol | Category | Algorithm | Metric | Convergence Speed | Typical Use Case |
|---|---|---|---|---|---|
| **RIP** (Routing Information Protocol) | Interior Gateway (IGP) | Distance Vector (Bellman-Ford) | Hop Count (Max 15 hops) | Slow; prone to routing loops | Small, legacy networks |
| **OSPF** (Open Shortest Path First) | Interior Gateway (IGP) | Link State (Dijkstra's SPF) | Cost (Inverse bandwidth) | **Very Fast**; uses hierarchical areas | Enterprise campus & data center backbones |
| **BGP** (Border Gateway Protocol) | Exterior Gateway (EGP) | Path Vector | AS-Path & Policy attributes | Moderate; optimized for global stability | The core routing protocol of the global Internet |

### Network Address Translation (NAT) & Port Address Translation (PAT)
* **Problem:** Depletion of public IPv4 addresses ($2^{32} pprox 4.3$ billion).
* **PAT / NAT Overload:** Maps thousands of private RFC 1918 internal IP addresses (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) to a **single public IP address** by translating source TCP/UDP port numbers.
