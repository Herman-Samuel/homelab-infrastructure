# Firewall Implementation Logic: Zero-Trust VLAN Segmentation
#### This document outlines the stateful inspection rules for the OPNsense gateway.

- **Global Policy:** Default Deny (All traffic is blocked unless a specific 'Allow' rule exists). To facilitate monitoring, specific "Block & Log" rules are implemented for high-risk segments (Server, Sandbox and IoT) to provide telemetry to the Splunk instance for incident detection.
- Makes use of aliases to make firewall rules clean, professional, and scalable

#### Alias Definitions
- **Main_Admin_Host:** 10.0.10.10
	- Static IP Address for my PC
-  **SIEM_Collector:** 10.0.30.10
	- Static IP Address for the Raspberry Pi
- **SIEM_Ports:** 514, 2055, 9997
	- Facilitates secure security monitoring by consolidating protocols for centralized system logging (Syslog), network flow analysis (NetFlow), and high-performance data ingestion via the Splunk Universal Forwarder.
- **Main_Net:** 10.0.10.0/24
	- IP Address range for the Main VLAN
- **Guest_Net:** 10.0.20.0/24
	- IP Address range for the Guest VLAN
- **Server_Net:** 10.0.30.0/24
	- IP Address range for the Server VLAN
- **IoT_Net:** 10.0.40.0/24
	- IP Address range for the IoT VLAN
- **Sandbox_Net:** 10.0.50.0/24
	- IP Address range for the Sandbox VLAN
- **MGMT_Net:** 10.0.60.0/24 
	- IP Address range for the Management VLAN
- **Private_Networks:** 10.0.0.0/16
	- IP Address range for the all of the VLANS
- **SIEM_Alert_Ports:** 25, 587, 443, 8088
	- Facilitates outbound communication from the security stack to authorized administrative endpoints by consolidating protocols for secure email notifications (SMTP/S) and encrypted webhooks (HTTPS/HEC).
- **Server_Admin_Ports:** 22, 80, 443, 445, 2049, 5000, 5001, 137:139, 1900, 3702, 8000, 9997, 9100, 631, 515, 5353
	- Facilitates comprehensive management of the server segment by consolidating protocols for secure remote access (SSH), web-based administration (HTTP/S), enterprise file sharing (SMB/NFS), automated network discovery (WSD/mDNS/NetBIOS), and multi-protocol printing and scanning services (IPP/RAW/LPR).
- **IoT_Safe_Services:** 80, 443, 123, 53, 1194, 8008, 8009, 3478, 3479
	- Implements a zero-trust posture for the IoT segment by explicitly blocking all inter-VLAN lateral movement and restricting WAN egress to a verified whitelist of multimedia and security devices via standard web and synchronization ports (HTTP/S, DNS, NTP).
- **Ring_Cameras:** 10.0.40.50, 10.0.40.51, 10.0.40.52, 10.0.40.53, 10.0.40.54, 10.0.40.55, 10.0.40.56
	- Static IP Addresses for my 7 security cameras (Will leave .57-59 unassigned just in case more cameras are introduced to the network).
- **Ring_Video:** 16500:65000
	- Implements a granular egress filter for security hardware by permitting high-range UDP traffic (RTP/SRTP) exclusively for verified camera hardware.
- **OPNsense_IP:** 10.0.60.10
	- Static IP Address of OPNsense router
- **Sandbox_Services:** 53, 3128
	- Utilizes a Protocol-Break strategy. By restricting egress to a dedicated Proxy (3128) and internal DNS (53), we strip malware of its ability to use standard 'Direct-to-WAN' communication. This architecture forces all external requests through a centralized inspection point where traffic can be killed instantly if anomalous exfiltration patterns are detected in the SIEM.
## Inter-VLAN Routing Matrix

| **Source**           | **Destination**   | **Dest Port**      | **Action** | **Purpose**                                                                             |
| -------------------- | ----------------- | ------------------ | ---------- | --------------------------------------------------------------------------------------- |
| **Main_Admin_Host**  | MGMT_Net          | Any                | **ALLOW**  | The Master Key: Full access to infrastructure management                                |
| **Private_Networks** | SIEM_Collector    | SIEM_Ports         | **ALLOW**  | Log aggregation on Raspberry Pi                                                         |
| **Main_Net**         | Server_Net        | Server_Admin_Ports | **ALLOW**  | Scalable Admin Access                                                                   |
| **Main_Net**         | Any               | Any                | **ALLOW**  | Full access to all segments and WAN.                                                    |
| **Guest_Net**        | !Private_Networks | Any                | **ALLOW**  | Internet only; blocked from private subnets.                                            |
| **Server_Net**       | Main_Admin_Host   | SIEM_Alert_Ports   | **ALLOW**  | Push alerts to main workstation.                                                        |
| **Server_Net**       | Any               | Any                | **BLOCK**  | Explicit block; to log packets handled by this rule.                                    |
| **Ring_Cameras**     | !Private_Networks | Ring_Video         | **ALLOW**  | No LAN, restricted WAN; the large range of ports Ring is known to require for live view |
| **IoT_Net**          | !Private_Networks | IoT_Safe_Services  | **ALLOW**  | No LAN, restricted WAN; Only ports necessary for Ring camera and Smart TV services      |
| **IoT_Net**          | Any               | Any                | **BLOCK**  | Explicit block; to log packets handled by this rule.                                    |
| **Sandbox_Net**      | OPNsense_IP       | Sandbox_Services   | **ALLOW**  | No LAN, proxied & monitored WAN access                                                  |
| **Sandbox_Net**      | Any               | Any                | **BLOCK**  | Explicit block; to log packets handled by this rule.                                    |
| **MGMT_Net**         | Any               | Any                | **REJECT** | The Vault Lock: No Outbound/WAN                                                         |
Note: Rules are applied in top-down priority; specific inter-VLAN restrictions are placed above general WAN egress rules
