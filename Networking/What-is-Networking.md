# _What is Networking?_ 

Begin learning the fundamentals of computer networking in this bite-sized and interactive module.


## Task 1: What is Networking?

**A network is an interconnection between two or more devices that share data, services, and resources**. 

**Definition:** **Networking is the practice of interconnecting computers, servers, and other devices to share data/communicate with each other**, while implementing strict controls to protect that data from unauthorized access or attacks. 
It forms the backbone of security because you cannot protect a digital asset unless you secure the network pathway leading to it.

This is why networking underpins cybersecurity as a whole: you cannot protect a digital asset unless you also secure the network pathway leading to it. A perfectly locked-down server is still exposed if the network carrying traffic to and from it isn't controlled.

### Core concepts of secure networking

| Concept | What it does |
|---|---|
| **Network Segmentation** | Divides a network into smaller, isolated sub-networks so a breach in one segment doesn't automatically spread to the rest |
| **Firewalls** | Sit between a device/network and the internet; monitor and filter incoming/outgoing traffic based on predefined rules |
| **Access Control** | Manages *who* and *what devices* are allowed to connect, via authentication and authorization |
| **Encryption** | Protects data in transit (VPNs, HTTPS) so intercepted traffic can't be read |
| **IDS/IPS** (Intrusion Detection/Prevention Systems) | Monitors network traffic for malicious activity and either alerts (IDS) or actively blocks it (IPS) |

**Q&A**
- What is the key term for devices that are connected together? **Network**


## Task 2: What is the Internet?

**Definition:** The **Internet is a massive, global network of interconnected networks** that allows billions of devices worldwide to communicate and share data. It's not one network, it's countless smaller networks(sub-networks) joined together.

The **Internet is made up of many small, interconnected networks**.  
**These small networks are called private networks**, where **networks connecting these small networks are called public networks, or the Internet!** 

<img width="852" height="579" alt="image" src="https://github.com/user-attachments/assets/ec0f41f6-7333-46a5-8a7b-0ae7b9a83e09" />


**A little history:** The earliest version of what became the Internet was **ARPANET**, a project in the late 1960s funded by the U.S. Department of Defense. The Internet as most people experience it today, however, traces back to **1989–1991**, when **Tim Berners-Lee** proposed and then built the **World Wide Web (WWW)**, the system of linked, browsable documents that turned the Internet into the information-sharing tool it is now.

### How the Internet impacts cybersecurity

- **Expanded attack surface**: connecting to the Internet exposes an organization's internal systems to threats from anywhere in the world, not just local ones.
- **Public routing**: data crossing the Internet passes through multiple third-party routers you don't control, which is exactly why encryption (HTTPS, VPNs) matters so much for anything sensitive.
- **Edge defense**: Organizations must deploy robust defense mechanisms—like firewalls and proxy servers—at the boundary where their private network meets the public Internet.
- **Cloud security**: since most modern applications and data live on the Internet rather than in a physical building, security has shifted heavily toward identity and access management (IAM) - controlling *who* can reach data, not just *where* the server sits.

### NAT (Network Address Translation)

Network Address Translation (NAT) is a **network service typically running on a router or firewall. It's used to translate private IP addresses into public IP addresses and vice versa to access the internet**.

With only about 4.3 billion possible IPv4 addresses (see Task 3) and far more devices than that connected worldwide, not every device can have its own public IP. **NAT** solves this: **a router translates the private IP addresses of devices on a local network into a single shared public IP address (and back again) when they communicate with the Internet**. This is why dozens of devices on a home network can all browse the web using just one public IP address issued by the ISP. NAT is a major reason IPv4 has lasted as long as it has despite address exhaustion.

### Types of NAT

- Static NAT: One private IP is mapped permanently to one public IP, always the same pairing. Used when a specific internal device (like a mail server) needs to be consistently reachable from the outside.
- Dynamic NAT: Private IPs are mapped to public IPs from a shared pool, but on a first-come, first-served basis. You don't get the same public IP every time, and the pool has a limited number of addresses to hand out.
- PAT (Port Address Translation) — also called NAT Overload: The version almost everyone actually uses. Many private IPs share a single public IP, distinguished by port number rather than address. This is how an entire household can be online simultaneously through one ISP-assigned public IP, the router tracks which internal device owns which port and rewrites traffic accordingly on the way out and back in.


**Q&A**
- Who invented the World Wide Web? **Tim Berners-Lee**


## Task 3: Identifying Devices on a Network

Every device needs two forms of identity on a network, similar to how a person has both a name (which can change) and fingerprints (which can't):

| Identifier | Networking equivalent | Can it change? |
|---|---|---|
| Name | **IP Address** | Yes. Dynamic, can be reassigned |
| Fingerprint | **MAC Address** | No (in principle), burned into the hardware at manufacture, though it *can* be spoofed at the software level (see below) |

### IP Addresses

Internet protocols are a set of rules that govern how data is formatted, addressed, transmitted, and routed across a network.

Core Internet Layer Protocols: **Internet Protocol (IP), ICMP, ARP**

An **IP (Internet Protocol) address** is a **32-bit unique address used to identify devices on the internet** written as four decimal numbers ("octets") each ranging 0–255, e.g. `192.168.1.1`.
The first part of the address usually represents the network the device is on (192.168.0.x), and the last part of the address represents the host device (192.168.0.1)

<img width="1140" height="487" alt="image" src="https://github.com/user-attachments/assets/a42d6cd6-6662-4eeb-b56c-de86666a708f" />


**A note on IP addressing & subnetting**: exactly where the line falls between "network portion" and "host portion" is determined by a **subnet mask** (e.g. `255.255.255.0`) or its shorthand, **CIDR notation** (e.g. `/24`). For example, `192.168.1.0/24` means the first 24 bits (three octets) identify the network, leaving the last 8 bits (256 possible values, 254 usable) for individual hosts on that network — so devices `192.168.1.1` through `192.168.1.254` could all sit on that one network. This is a deep topic in its own right and worth its own dedicated write-up as you go further.

**Public vs. private IP addresses:**
- A private IP address **identifies a device *within* its local network** (e.g. `192.168.1.77`) and isn't reachable directly from the Internet. Private ranges are reserved by standard (RFC 1918): `10.0.0.0/8`, `172.16.0.0/12`, and `192.168.0.0/16`.
- A public IP address **identifies a device (or, more often, a whole network via NAT) *on* the Internet**, and is issued by an ISP.

Multiple devices on the same private network can share the very same public IP address when talking to the Internet, this is NAT in action (see Task 2).

**IPv4 vs. IPv6:**
- **IPv4** uses 32-bit addressing → 2³² ≈ 4.29 billion possible addresses. With billions of devices now online, this pool is effectively exhausted.
- **IPv6** uses 128-bit addressing → 2¹²⁸ ≈ 340 **undecillion** addresses (3.4 × 10³⁸). Vastly larger than the "340 trillion" figure sometimes quoted; the real number is trillions of trillions of trillions larger than IPv4's space. IPv6 also brings efficiency improvements to routing and configuration.

### **A note on protocols IP relies on**(protocols that IP address follows):

IP's job is narrow and specific: **get a packet from a source address to a destination address**. It doesn't do anything else; it doesn't guarantee delivery, doesn't check for errors, and doesn't know or care what's actually inside the packet. That's intentional; IP is designed to be a thin, universal addressing layer that everything else builds on top of. This is why it's usually written as "TCP/IP". IP essentially never operates alone.

The protocols that ride on top of IP are the ones that turn "a packet arrived somewhere" into "an actual conversation happened":

- **TCP (Transmission Control Protocol)**: adds reliability on top of IP's best-effort delivery. It establishes a connection (the three-way handshake), numbers packets so they can be reassembled in the correct order, detects loss, and retransmits missing data. This is why file transfers, web pages, and emails use TCP — losing or scrambling part of the data would break the result.
- **UDP (User Datagram Protocol)**: skips all of that overhead. No handshake, no guaranteed delivery, no ordering; packets are just fired off. This trade-off is worth it when speed matters more than perfection, e.g. DNS lookups, video calls, or online gaming, where a dropped packet is better resent (or ignored) than waited on.
- **ICMP (Internet Control Message Protocol)**: doesn't carry user data at all, it's for the network to talk about itself. Things like "this destination is unreachable," "TTL exceeded," or the echo request/reply pair that ping uses to check reachability.

### MAC Addresses

A **MAC (Media Access Control) address** is a physical address. It is a unique identifier burned into a device's network interface at the factory. It's a 12-character hexadecimal number, split into pairs separated by colons, e.g. `a4:c3:f0:85:ac:2d`.
- The first 6 characters (24 bits) identify the manufacturer of the network interface.
- The last 6 characters are a unique identifier for that specific interface.

  <img width="1140" height="669" alt="image" src="https://github.com/user-attachments/assets/36f9364f-01f1-4850-a563-b01971ba881a" />


Unlike an IP address, which is typically **dynamic** (can be reassigned by a network, e.g. via DHCP), a MAC address is intended to be **static**: permanently tied to that one piece of hardware.

**MAC spoofing**: 

MAC spoofing is the **technique of changing or disguising a device's unique Media Access Control (MAC) address at the software level to impersonate another device or hide its true identity**.
By using MAC spoofing, attackers can trick a network into thinking their unauthorized device is actually a trusted one. This allows them to bypass security controls, steal data, or remain hidden.

Examples: 
* Network Access (access to a restricted corporate network, free internet)
* Eavesdropping and Data Theft (Man-in-the-Middle Attacks) (Intercept traffic, Steal credentials)
* Disruption (Denial of Service)
* Anonymity and Hiding Footprints (Evade forensic tracking, Blame others)

This matters for security because some network setups (like paid hotel/cafe Wi-Fi) authorize devices based on MAC address alone. **In the practical lab**, Bob's packets were being blocked because his MAC address wasn't recognized as "paid," while Alice's were let through because hers was. Spoofing your device's MAC address to match Alice's made the router believe the traffic was coming from her already-authorized device — which is why the packets started reaching TryHackMe successfully afterward. This illustrates a real weakness: MAC-based trust assumes a device's stated identity is honest, which spoofing directly breaks.

**Q&A**
- What does "IP" stand for? **Internet Protocol**
- What is each section of an IP address called? **Octet**
- How many sections does an IPv4 address have? **4**
- What does "MAC" stand for? **Media Access Control**
- Flag from spoofing your MAC address to Alice's in the lab: **THM{YOU_GOT_ON_TRYHACKME}**



## Task 4: Ping (ICMP)

**Ping** is a basic **diagnostic tool used to test whether a connection to another device exists and how reliable/fast it is**. It works by sending an **ICMP echo request** packet to a target and timing how long it takes to receive an **ICMP echo reply** back.

### A note on ICMP
**ICMP (Internet Control Message Protocol)** is a **network-layer protocol used for diagnostics and error reporting between devices**. It doesn't carry actual application data the way TCP or UDP do. **Its job is purely operational: telling devices things like "this destination is unreachable," "this packet took too long," or, in ping's case, simply confirming "yes, I'm here and reachable."** Because it operates below the application layer and isn't meant to transport user data, ICMP is comparatively lightweight, but it's also often restricted or rate-limited by firewalls, since it can be abused for reconnaissance or denial-of-service-style attacks (e.g. ICMP floods).

**Basic syntax**: `ping <IP address or URL>` 

e.g. `ping 192.168.1.254` or `ping google.com`. Ping is built into both Linux and Windows by default.

**Q&A**
- What protocol does ping use? **ICMP**
- Syntax to ping `10.10.10.10`? **ping 10.10.10.10**
- Flag from pinging `8.8.8.8`: **THM{I_PINGED_THE_SERVER}**


## Task 5: Continue Your Learning

Follow-on room: [Intro to LAN](https://tryhackme.com/room/introtolan)


## Key Takeaways
- Networking = devices connected to share resources; securing the network is inseparable from securing what's on it.
- The Internet = a public network made of countless smaller private networks joined together.
- Devices need two identifiers: an **IP address** (dynamic, "name") and a **MAC address** (static in principle, "fingerprint" — though spoofable in practice).
- IPv4's 32-bit space (~4.3B addresses) is running out; NAT stretches it further, and IPv6's 128-bit space is the long-term fix.
- ICMP (used by ping) is a diagnostic protocol, not a data-carrying one — it just answers "is this device reachable, and how fast?"
