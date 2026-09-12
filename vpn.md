# VPN (Virtual Private Network)

**First, the most important thing:** a VPN does not make you anonymous. Your VPN provider can still see your traffic, you're just shifting who you trust from your ISP to the VPN company. Choose one with a genuine no-logs policy if privacy matters to you.

**VPN is a network service that encrypts network traffic between your device and a VPN server, forming a secure tunnel over the internet. It masks your true IP address by replacing it with the VPN server's IP address for the websites and services you connect to.**

**Why people use it: privacy, security on public networks, and accessing geo-restricted content.**

## How it works
Normally: `device → ISP → website`
With a VPN: `device → encrypted tunnel → VPN server → website`

<table>
 <tr>
<td> <img width="100%" height="599" alt="vpn1" src="https://github.com/user-attachments/assets/7cd6bfa7-73d7-433d-bff4-ca0ad69edc7e" />
<td> <img width="100%" height="833" alt="vpn2" src="https://github.com/user-attachments/assets/8a3aa9ce-20c9-418f-9722-ed051afd9777" />
</tr>
</table>

The VPN encrypts your traffic and routes it through its own server before it reaches the destination. 

This:
- **Hides your traffic content from your ISP**: they can see you're connected to a VPN, but not what's inside the tunnel (unless the VPN uses obfuscation to hide even that fact, which matters in countries that block VPNs)
- **Masks your IP**: the website sees the VPN server's IP and location, not yours
- **Protects you on public wifi**: where anyone else on the network could otherwise snoop your traffic

## Common uses
Privacy, security on untrusted networks, bypassing geo-restrictions, accessing region-locked content.

## Gotchas worth knowing (especially for SOC/network work)
- **DNS leaks**: a misconfigured VPN can encrypt your traffic but still leak DNS lookups straight to your ISP, defeating part of the purpose. Good VPN clients route DNS through the tunnel too.
- **Kill switch**: cuts your internet entirely if the VPN connection drops, so you don't silently fall back to unencrypted traffic.
- **VPN types**:

**Remote-access VPN** - An individual device connects to a VPN service over the internet.

This is the consumer/personal use case- privacy apps, accessing geo-restricted content, and also the standard way employees connect to a company network remotely (e.g., working from home).

**Site-to-site VPN** - Connects two entire networks to each other, rather than a single device to a network. Typically used to link a branch office's network to headquarters, so devices on either side can talk to each other as if on the same LAN. 
No individual client software needed; the connection is established at the router/firewall level between the two sites. This is the type that comes up more in CCNA and enterprise network security contexts, since it's about network architecture rather than personal device configuration.

<table>
  <tr>
    <td><img width="100%" alt="vpntype1" src="https://github.com/user-attachments/assets/e63ac3e9-8981-49b8-80ff-40070c176939" /></td>
    <td><img width="100%" alt="vpntype2" src="https://github.com/user-attachments/assets/7329e0a9-e2ef-4c40-bb54-46d46332f47e" /></td>
  </tr>
</table>

- **Protocols**: VPN protocols are **sets of rules and algorithms** that **determine how data is encrypted, authenticated, and transmitted between your device and a VPN server**. (Not related - Internet Protocol (IP): Internet protocols are a set of rules that govern how data is formatted, addressed, transmitted, and routed across a network).
 — OpenVPN, WireGuard, IPSec are the main ones. They trade off differently on speed vs. security, worth knowing by name.








