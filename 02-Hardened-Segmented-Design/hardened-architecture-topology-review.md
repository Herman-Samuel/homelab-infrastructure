# Hardened Architecture: Topology Review
**Explanation of the [Hardened Architecture Diagram](https://github.com/Herman-Samuel/homelab-infrastructure/blob/main/02-Hardened-Segmented-Design/hardened-architecture-topology-review.md)**

This document provides a technical analysis of the hardened network architecture diagram. The transition from the legacy flat model to this segmented model utilizes 802.1Q VLAN tagging and firewall rules to enforce isolation between different segments. 

---
### **Diagram Flow**
- At the top is the title, "Hardened Architecture: Segmented Network Model", which clearly emphasizes that this diagram is about the more secure segmented network that I am designing and implementing.
- The start of the path is an internet symbol, representing the untrusted public web.
- An arrow goes from the internet symbol to the ISP Gateway titled "Edge Gateway". Under it is a note stating that it is set to IP Passthrough, meaning all this device is doing is acting as the "courtyard door", passing the public IP Address to my custom router.
- Next, an arrow goes from the edge gateway to a light pink box titled "VLAN 60: Management", and has 10.0.60.0/24 as its private IP Address range. "Out-of-Band Management" is written in green here to indicate a fix from the Legacy Architecture Diagram which has "In-Band Management". On the right hand side of the Management VLAN box is a green shield with a checkmark and green text that reads "Enterprise-Grade Perimeter & Inter-VLAN Security". This is a fix from the Legacy Architecture Diagrams red text "Consumer-Grade Perimeter Security".
- The arrow from the edge gateway continues to inside the light pink VLAN 60 box, pointing towards a darker pink box residing within, titled "OPNsense Router/Firewall". This is my custom built and configured router and firewall, which is able to handle inter-VLAN routing and advanced security (e.g. Intrusion Prevention System and Next Generation Firewall).
- From the OPNsense router the arrow points to a managed switch. This managed switch is what the access point and wired devices are plugged into. Each port is configured to a "trunk port" for VLAN tagging or an "access port" for direct device plug in to a specific VLAN.
- 
---
### **The Management Plane (VLAN 60)**
- **Core Infrastructure:** The OPNsense router, Managed Switch, and Wireless Access Point reside in this isolated segment.
- **Control logic:** Management interfaces are no longer accessible from general-purpose segments (like Guest or IoT). Administrative access is restricted specifically to authorized devices in the Main VLAN, protecting the "brain" of the network from credential sniffing or unauthorized configuration changes.
---
### **VLAN Segmentation & Trust Levels**
The network is partitioned into five client segments, each with a specific security policy:
- **VLAN 10: Main (Trusted):** High-trust workstations and personal devices. This segment has stateful access to all other VLANs for management and printing, but is protected from incoming unsolicited traffic from lower-trust zones.
- **VLAN 20: Guest (Untrusted):** Wireless-only segment with a strict "WAN-Only" policy. Devices can reach the Internet but have zero visibility into the internal 10.0.0.0/16 private space.
- **VLAN 30: Server (Internal Services):** The homes static infrastructure like the NAS, Printer, and the Raspberry Pi (Security Stack). Traffic is restricted to specific ports (e.g., 514 for Syslog, 9997 for Splunk) to ensure the server segment remains a secure repository for data and telemetry.
- **VLAN 40: IoT (Restricted):** Contains high-risk "Smart Home" devices and cameras. These are blocked from the LAN and limited to specific external domains (e.g., Ring/Microsoft) to prevent phone-home telemetry to unauthorized servers.
- **VLAN 50: Sandbox (High Risk):** A strictly isolated "Dirty Lab" for malware analysis and high-risk research. Traffic is "Allowed with Restrictions" to the WAN to allow for research while maintaining a total block on all internal communication to prevent lateral spread.
---
### **Security Controls Replaced (Green Text Analysis)**

This architecture directly remediates the "Red Text" vulnerabilities found in the Legacy Audit:

- **Restricted Lateral Movement:** East-West traffic is now statefully inspected by the OPNsense firewall.
    
- **Separate DHCP Pools:** The router now distinguishes between device classes by assigning IPs from dedicated scopes, allowing for VLAN-specific DNS (Pi-hole) and firewall rules.
    
- **Enterprise-Grade Perimeter:** By utilizing **IP Passthrough** on the ISP Gateway, we have bypassed consumer-grade limitations in favor of the HP t620 Plus’s superior processing and filtering capabilities.