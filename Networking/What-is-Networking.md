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

**A little history:** The earliest version of what became the Internet was **ARPANET**, a project in the late 1960s funded by the U.S. Department of Defense. The Internet as most people experience it today, however, traces back to **1989–1991**, when **Tim Berners-Lee** proposed and then built the **World Wide Web (WWW)**, the system of linked, browsable documents that turned the Internet into the information-sharing tool it is now.

### How the Internet impacts cybersecurity

- **Expanded attack surface**: connecting to the Internet exposes an organization's internal systems to threats from anywhere in the world, not just local ones.
- **Public routing**: data crossing the Internet passes through multiple third-party routers you don't control, which is exactly why encryption (HTTPS, VPNs) matters so much for anything sensitive.
- **Edge defense**: Organizations must deploy robust defense mechanisms—like firewalls and proxy servers—at the boundary where their private network meets the public Internet.
- **Cloud security**: since most modern applications and data live on the Internet rather than in a physical building, security has shifted heavily toward identity and access management (IAM) - controlling *who* can reach data, not just *where* the server sits.

### NAT (Network Address Translation)

Network Address Translation (NAT) is a **network service typically running on a router or firewall. It's used to translate private IP addresses into public IP addresses and vice versa to access the internet**.

With only about 4.3 billion possible IPv4 addresses (see Task 3) and far more devices than that connected worldwide, not every device can have its own public IP. **NAT** solves this: **a router translates the private IP addresses of devices on a local network into a single shared public IP address (and back again) when they communicate with the Internet**. This is why dozens of devices on a home network can all browse the web using just one public IP address issued by the ISP. NAT is a major reason IPv4 has lasted as long as it has despite address exhaustion.

**Q&A**
- Who invented the World Wide Web? **Tim Berners-Lee**


## Task 3 — Identifying Devices on a Network

Every device needs two forms of identity on a network — similar to how a person has both a name (which can change) and fingerprints (which can't):

| Identifier | Networking equivalent | Can it change? |
|---|---|---|
| Name | **IP Address** | Yes — dynamic, can be reassigned |
| Fingerprint | **MAC Address** | No (in principle) — burned into the hardware at manufacture, though it *can* be spoofed at the software level (see below) |

### IP Addresses

An **IP (Internet Protocol) address** is a **32-bit** address (for IPv4) used to identify a device on a network, written as four decimal numbers ("octets") each ranging 0–255, e.g. `192.168.1.1`.

Within an address, the address is generally split into a **network portion** and a **host portion**:
- The earlier part of the address typically identifies which network the device belongs to (e.g. `192.168.0.x`)
- The final part identifies the specific device on that network (e.g. `.1`, `.2`, etc.)

**A note on IP addressing & subnetting**: exactly where the line falls between "network portion" and "host portion" is determined by a **subnet mask** (e.g. `255.255.255.0`) or its shorthand, **CIDR notation** (e.g. `/24`). For example, `192.168.1.0/24` means the first 24 bits (three octets) identify the network, leaving the last 8 bits (256 possible values, 254 usable) for individual hosts on that network — so devices `192.168.1.1` through `192.168.1.254` could all sit on that one network. This is a deep topic in its own right and worth its own dedicated writeup as you go further.

**Public vs. private IP addresses:**
- A **private IP address** identifies a device *within* its local network (e.g. `192.168.1.77`) and isn't reachable directly from the Internet. Private ranges are reserved by standard (RFC 1918): `10.0.0.0/8`, `172.16.0.0/12`, and `192.168.0.0/16`.
- A **public IP address** identifies a device (or, more often, a whole network via NAT) *on* the Internet, and is issued by an ISP.

Multiple devices on the same private network can share the very same public IP address when talking to the Internet — this is NAT in action (see Task 2).

**IPv4 vs. IPv6:**
- **IPv4** uses 32-bit addressing → 2³² ≈ 4.29 billion possible addresses. With billions of devices now online, this pool is effectively exhausted.
- **IPv6** uses 128-bit addressing → 2¹²⁸ ≈ 340 **undecillion** addresses (3.4 × 10³⁸) — vastly larger than the "340 trillion" figure sometimes quoted; the real number is trillions of trillions of trillions larger than IPv4's space. IPv6 also brings efficiency improvements to routing and configuration.

**A note on protocols IP relies on**: IP itself is just one layer of the broader **TCP/IP suite**. It handles addressing and routing, while other protocols handle the rest of the conversation — **TCP** and **UDP** carry the actual application data on top of IP, and **ICMP** (see Task 4) handles diagnostics and error reporting. None of these work in isolation; IP gets packets to the right device, but it's the protocols layered above/alongside it that do the rest of the job.

### MAC Addresses

A **MAC (Media Access Control) address** is a unique identifier burned into a device's network interface at the factory. It's a 12-character hexadecimal number, split into pairs separated by colons, e.g. `a4:c3:f0:85:ac:2d`.
- The first 6 characters (24 bits) identify the manufacturer of the network interface.
- The last 6 characters are a unique identifier for that specific interface.

Unlike an IP address, which is typically **dynamic** (can be reassigned by a network, e.g. via DHCP), a MAC address is intended to be **static** — permanently tied to that one piece of hardware.

**MAC spoofing**: despite being "burned in," a MAC address is just a value the operating system reports to the network — and most operating systems allow you to override what the network interface presents, without physically altering the hardware chip itself. This is exactly what changing your MAC address to another device's accomplishes: the network layer sees the spoofed value and treats your traffic as if it came from that other device.

This matters for security because some network setups (like paid hotel/cafe Wi-Fi) authorize devices based on MAC address alone. **In the practical lab**, Bob's packets were being blocked because his MAC address wasn't recognized as "paid," while Alice's were let through because hers was. Spoofing your device's MAC address to match Alice's made the router believe the traffic was coming from her already-authorized device — which is why the packets started reaching TryHackMe successfully afterward. This illustrates a real weakness: MAC-based trust assumes a device's stated identity is honest, which spoofing directly breaks.

**Q&A**
- What does "IP" stand for? **Internet Protocol**
- What is each section of an IP address called? **Octet**
- How many sections does an IPv4 address have? **4**
- What does "MAC" stand for? **Media Access Control**
- Flag from spoofing your MAC address to Alice's in the lab: **THM{YOU_GOT_ON_TRYHACKME}**

---

## Task 4 — Ping (ICMP)

**Ping** is a basic diagnostic tool used to test whether a connection to another device exists and how reliable/fast it is. It works by sending an **ICMP echo request** packet to a target and timing how long it takes to receive an **ICMP echo reply** back.

### A note on ICMP
**ICMP (Internet Control Message Protocol)** is a network-layer protocol used for diagnostics and error reporting between devices — it doesn't carry actual application data the way TCP or UDP do. Its job is purely operational: telling devices things like "this destination is unreachable," "this packet took too long," or, in ping's case, simply confirming "yes, I'm here and reachable." Because it operates below the application layer and isn't meant to transport user data, ICMP is comparatively lightweight — but it's also often restricted or rate-limited by firewalls, since it can be abused for reconnaissance or denial-of-service style attacks (e.g. ICMP floods).

**Basic syntax**: `ping <IP address or URL>` — e.g. `ping 192.168.1.254` or `ping google.com`. Ping is built into both Linux and Windows by default.

**Q&A**
- What protocol does ping use? **ICMP**
- Syntax to ping `10.10.10.10`? **ping 10.10.10.10**
- Flag from pinging `8.8.8.8`: **THM{I_PINGED_THE_SERVER}**

---

## Task 5 — Continue Your Learning

Follow-on room: [Intro to LAN](https://tryhackme.com/room/introtolan)

---

## Key Takeaways
- Networking = devices connected to share resources; securing the network is inseparable from securing what's on it.
- The Internet = a public network made of countless smaller private networks joined together.
- Devices need two identifiers: an **IP address** (dynamic, "name") and a **MAC address** (static in principle, "fingerprint" — though spoofable in practice).
- IPv4's 32-bit space (~4.3B addresses) is running out; NAT stretches it further, and IPv6's 128-bit space is the long-term fix.
- ICMP (used by ping) is a diagnostic protocol, not a data-carrying one — it just answers "is this device reachable, and how fast?"

























-----


Networks are simply things connected. For example, your friendship circle: you are all connected because of similar interests, hobbies, skills and sorts.

Networks can be found in all walks of life:

A city's public transportation system
Infrastructure such as the national power grid for electricity
Meeting and greeting your neighbours
Postal systems for sending letters and parcels
But more specifically, in computing, networking is the same idea, just dispersed to technological devices. Take your phone as an example; the reason that you have it is to access things. We'll cover how these devices communicate with each other and the rules that follow.

In computing, a network can be formed by anywhere from 2 devices to billions. These devices include everything from your laptop and phone to security cameras, traffic lights and even farming!

Networks are integrated into our everyday life. Be it gathering data for the weather, delivering electricity to homes or even determining who has the right of way at a road. Because networks are so embedded in the modern-day, networking is an essential concept to grasp in cybersecurity.

Take the diagram below as an example, Alice, Bob and Jim have formed their network! We'll come onto this a bit later on.

<img width="846" height="593" alt="image" src="https://github.com/user-attachments/assets/8eb0bb12-306b-4865-b8db-7153ceda926c" />


Networks come in all shapes and sizes, which is something that we will also come on to discuss throughout this module. 

Answer the questions below
What is the key term for devices that are connected together?
Network
Correct Answer


## Some edits I need in this topic:

### Networking definition:
Networking is the practice of connecting computers, servers, and other devices to share data, while implementing strict controls to protect that data from unauthorized access or attacks. It forms the backbone of security because you cannot protect a digital asset unless you secure the network pathway leading to it.

### Core Concepts of Secure Networking:
- Network Segmentation: Dividing a network into smaller, isolated sub-networks to contain breaches.
- Firewalls: Security devices that monitor and filter incoming and outgoing network traffic based on rules.
- Access Control: Managing who (and what devices) can connect to the network using authentication and authorization.
- Encryption: Protecting data in transit (like using VPNs or HTTPS) so attackers cannot read it if intercepted.Intrusion Detection/Prevention
- (IDS/IPS): Monitoring systems that spot and block malicious activity on the network.


# Task 2: What is the Internet?

Now that we've learnt what a network is and how one is defined in computing (just devices connected), let's explore the Internet.

The Internet is one giant network that consists of many, many small networks within itself. Using our example from the previous task, let's now imagine that Alice made some new friends named Zayn and Toby that she wants to introduce to Bob and Jim. The problem is that Alice is the only person who speaks the same language as Zayn and Toby. So Alice will have to be the messenger!

<img width="712" height="800" alt="image" src="https://github.com/user-attachments/assets/7a45194b-3168-4245-a95c-68191ab281f3" />

Because Alice can speak both languages, they can communicate to one another through Alice — forming a new network.

The first iteration of the Internet was within the ARPANET project in the late 1960s. This project was funded by the United States Defence Department and was the first documented network in action. However, it wasn't until 1989 when the Internet as we know it was invented by Tim Berners-Lee by the creation of the World Wide Web (WWW). It wasn't until this point that the Internet started to be used as a repository for storing and sharing information, just like it is today.

Let's relate Alice's network of friends to computing devices. The Internet looks like a much larger version of this sort of diagram:

<img width="852" height="579" alt="image" src="https://github.com/user-attachments/assets/15c3896a-d941-4cc9-88f1-36797a46573b" />

As previously stated, the Internet is made up of many small networks all joined together.  These small networks are called private networks, where networks connecting these small networks are called public networks -- or the Internet! So, to recap, a network can be one of two types:

A private network
A public network
Devices will use a set of labels to identify themselves on a network, which we will come onto in the task below.

Answer the questions below
Who invented the World Wide Web?
Tim Berners-Lee
Correct Answer

## Some edits I need in this topic:

### Internet

The Internet is a massive, global network of interconnected networks that allows billions of devices worldwide to communicate and share data.

How the Internet Impacts Cybersecurity
Attack Surface: The Internet drastically expands an organization's attack surface, exposing internal systems to global threats if they are not properly protected.Public Routing: Data sent over the Internet travels through multiple third-party routers, making encryption (like HTTPS or VPNs) essential to prevent data theft.Edge Defense: Organizations must deploy robust defense mechanisms—like firewalls and proxy servers—at the boundary where their private network meets the public Internet.Cloud Security: Because modern applications and data live on the Internet (the cloud), security has shifted from protecting physical buildings to securing identity and access management (IAM).


# Task 3: Identifying Devices on a Network 




<img width="1140" height="487" alt="image" src="https://github.com/user-attachments/assets/4f174a39-1fe7-4c9d-9f31-ca9732d2956e" />





<img width="546" height="145" alt="image" src="https://github.com/user-attachments/assets/a31df245-7af7-429a-a6af-6c6adc7ccaf7" />


<img width="383" height="118" alt="image" src="https://github.com/user-attachments/assets/7c00ee36-5cfb-47b9-9fd6-56e12f686a8d" />



<img width="736" height="177" alt="image" src="https://github.com/user-attachments/assets/0b5c1f9f-2823-4946-ab09-36f10e2f50d4" />

<img width="1140" height="669" alt="image" src="https://github.com/user-attachments/assets/99788cf3-de59-493c-a98e-9c0106ab8808" />



## Some edits I need in this topic:

















