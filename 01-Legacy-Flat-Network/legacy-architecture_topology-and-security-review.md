# Explanation of the Legacy Architecture Diagram
**This document goes into detail of the diagram showing how my network is currently configured, including some of the security concerns (shown in red)**
**To see the diagram described in this document, [click here](https://github.com/Herman-Samuel/homelab-infrastructure/blob/main/01-Legacy-Flat-Network/legacy-architecture-diagram.png)

- The title, "Legacy Architecture: Flat Network Model", gives clear emphasis that this diagram is about the old network architecture, which is just a flat network.
- Under the title is the subtitle, "Categorized by Physical Layer (L1) / Non-Segmented". This explains that the end devices shown in the diagram are categorized by the OSI Layer 1, the Physical Layer. The four colored boxes at the bottom of the diagram the way each device accesses the internet (Wi-Fi 5GHz, Wired 2.5GbE, etc.), not any sort of segmentation.
- The start of the diagram path is an internet symbol, representing the untrusted public web.
- An arrow goes from the internet symbol to a dark gray box labeled "Edge Gateway / Firewall". Inside this box is a picture of my ISP router and a symbol of a firewall, which acts as a barrier between the internet and my home network. The primary role of this ISP router is Network Address Translation (NAT) and a basic stateful firewall.
- An arrow then leads to a large light gray box labeled "Single Broadcast Domain (192.168.1.0/24)", showing that all of the devices are on one single flat network. All devices on the network are in the same "logical room", despite how it is connected.
- Within the large light gray box are four boxes of different colors. These four boxes contain all of the end devices connected to the network, separated by how they are connected.
	- Wi-Fi 5GHz: High-bandwidth wireless for most Wi-Fi devices (purple box)
	- Wi-Fi 2.4GHz: Used for long-range and IoT devices (blue box)
	- Ring Chime Bridge: An intermediary "hop" for the security cameras (green box)
	- Wired 2.5GbE: High-speed wired connection for the desktops (yellow box)

#### Security Concerns in Diagram
- **"Unrestricted Lateral Movement!"** is in red three times, between all of the 'Connection Type' categories
	- Above each instance of the phrase is a red arrow that points both directions, indicating that East-West traffic is currently unrestricted within the network.
	- This emphasizes my main security concern of lateral movement in the case of a device being compromised. Currently IoT and untrusted devices on the network can see and talk to my trusted devices, which is not necessary and a security/ privacy risk.
		- If a security camera is compromised, the attacker can "walk" directly across the network to my main PC because there are no internal firewalls (VLANs) stopping them
- **"Shared DHCP Pool!"** is in red under the 'Single Broadcast Domain' box.
	- This is to show the risk of the router handing out IP addresses from one single pool of addresses. This can be a problem because the router cannot distinguish between a high-security desktop and a low-security smart appliance. It makes it impossible to apply different firewall rules to different types of devices because the router can't tell them apart by IP address alone.
- **"Consumer-Grade Perimeter Security!"** is in red to the right of the edge gateway/firewall.
	- This is to show concern of the limited capabilities of the current router/firewall. Besides the limitations of security features between the internet and the home network, the router's firewall has no way of dealing with traffic within the network. It can't be configured to filter traffic between certain types of devices.
- **"In-Band Management!"** is in red between "Sam PC" and the gateway to show the current lack of a management plane
	- Currently my administrative traffic (logging into the routers web GUI) is happening on the same "wire" as my regular internet traffic. This is a security concern because it increases the risk of credential sniffing and unauthorized access to the gateway.
	- The inclusion of the Ring Chime and various IoT devices on the same subnet as the management workstation significantly increases the internal attack surface.

#### This legacy model demonstrates a high degree of internal risk due to a lack of segmentation. Phase 2 will involve the deployment of an OPNsense firewall to transition from this Flat Model to a Hardened, Zero-Trust Architecture utilizing 802.1Q VLAN tagging.