# Firewall Implementation Logic: Zero-Trust VLAN Segmentation
#### This document outlines the stateful inspection rules for the OPNsense firewall.

- **Global Policy:** Default Deny (All traffic is blocked unless a specific 'Allow' rule exists). To facilitate monitoring, specific "Block & Log" rules are implemented for high-risk segments (Server, Sandbox and IoT) to provide telemetry to the Splunk instance for incident detection.
- Makes use of aliases to make firewall rules clean, professional, and scalable

#### Alias Definitions
- **Main_Admin_Host:** 10.0.10.10
	- Static IP Address for my PC
- **Rasp_Pi:** 10.0.30.10
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
- **Private_Networks:** 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
	- IP Address ranges for the all possible private networks, including the VLANs I created
- **SIEM_Alert_Ports:** 25, 587, 443, 8088, 8089, 465, 9997
	- Facilitates outbound communication from the security stack to authorized administrative endpoints by consolidating protocols for secure email notifications (SMTP/S) and encrypted webhooks (HTTPS/HEC).
- **Server_Admin_Ports:** 22, 80, 443, 445, 2049, 5000, 5001, 137:139, 1900, 3702, 8000, 9997, 9100, 631, 515, 5353
	- Facilitates comprehensive management of the server segment by consolidating protocols for secure remote access (SSH), web-based administration (HTTP/S), enterprise file sharing (SMB/NFS), automated network discovery (WSD/mDNS/NetBIOS), and multi-protocol printing and scanning services (IPP/RAW/LPR).
- **IoT_Services:** 80, 443, 123, 53, 1194, 8008, 8009, 3478, 3479, 161, 162, 500, 3000, 3001, 3074, 3544, 4500, 5061, 5353, 8557, 9100, 9955, 9998, 9999
	- Implements a zero-trust posture for the IoT segment by explicitly blocking all inter-VLAN lateral movement and restricting WAN egress to a verified whitelist of multimedia and security devices via standard web and synchronization ports (HTTP/S, DNS, NTP).
- **Ring_Cameras:** 10.0.40.50, 10.0.40.51, 10.0.40.52, 10.0.40.53, 10.0.40.54, 10.0.40.55, 10.0.40.56
	- Static IP Addresses for my 7 security cameras (Will leave .57-59 unassigned just in case more cameras are introduced to the network).
- **Ring_Video:** 16500:65000
	- Implements a granular egress filter for security hardware by permitting high-range UDP traffic (RTP/SRTP) exclusively for verified camera hardware.
- **Sandbox_Services:** 53, 3128, 3129
	- Utilizes a Protocol-Break strategy. By restricting egress to a dedicated Proxy (3128) and internal DNS (53), we strip malware of its ability to use standard 'Direct-to-WAN' communication. This architecture forces all external requests through a centralized inspection point where traffic can be killed instantly if anomalous exfiltration patterns are detected in the SIEM.
- **Nintendo_Ports:** 45000:65535
	- The large range of ports that the Nintendo Switch requires for online gaming
- **Nintendo_Switch:** 10.0.40.30
	- Static IP Address for the Nintendo Switch so that I can allow only the Switch access to Nintendo_Ports.
- UniFi_Ports
	- Ports needed for the Access Point
- **Web_GUI_Ports:** 443, 80
	- Allows access just to HTTP and HTTPS
- **Web_Ports:** 53, 80, 123, 443
	- Ports needed for a simple web connection (DNS, HTTP(S), NTP)
## Inter-VLAN Routing Matrix

| **Interface**  | **Source**      | **Destination**   | **Dest Port**      | **Action** | **Description**                                                                                 |
| -------------- | --------------- | ----------------- | ------------------ | ---------- | ----------------------------------------------------------------------------------------------- |
| **Main**       | Main_Admin_Host | MGMT_Net          | Any                | **ALLOW**  | Full access to MGMT VLAN from main PC                                                           |
| **Main**       | Main_Admin_Host | This Firewall     | Web_GUI_Ports      | **ALLOW**  | Explicitly allow main PC to access the OPNsense web GUI                                         |
| **Main**       | Main_Admin_Host | Rasp_Pi           | 8443               | **ALLOW**  | Explicitly allow main PC to access the Raspberry Pi at port 8443 to access the AP's controller. |
| **Main**       | Any             | MGMT_Net          | Any                | **BLOCK**  | Block all other devices from accessing the management VLAN                                      |
| **Main**       | Any             | This Firewall     | Web_GUI_Ports      | **BLOCK**  | Block any other devices from accessing the OPNsense web GUI                                     |
| **Main**       | Any             | Rasp_Pi           | 8443               | **BLOCK**  | Block other devices from accessing the Access Points controller                                 |
| **Main**       | Main_Net        | Server_Net        | Server_Admin_Ports | **ALLOW**  | Explicitly allow main PC to manage the Server VLAN so I can log it                              |
| **Main**       | Main_Net        | Any               | Any                | **ALLOW**  | Allow Main VLAN to everything (LAN & WAN)                                                       |
| **Guest**      | Guest_Net       | 10.0.20.1/24      | DOMAIN (53)        | **ALLOW**  | Allow Guest VLAN to query DNS                                                                   |
| **Guest**      | Guest_Net       | Rasp_Pi           | SIEM_Ports         | **ALLOW**  | From Guest, allow Syslog, NetFlow, and Splunk Universal Forwarder to send logs to the Pi        |
| **Guest**      | Guest_Net       | !Private_Networks | Any                | **ALLOW**  | Allow Guest traffic if it is NOT to private networks                                            |
| **Server**     | Server_Net      | Main_Admin_Host   | SIEM_Alert_Ports   | **ALLOW**  | Allow traffic from Server if going to Main PC through ports used for alerts                     |
| **Server**     | Server_Net      | Any               | Any                | **BLOCK**  | Explicitly block all traffic to log packets handled by this rule                                |
| **IoT**        | IoT_Net         | 10.0.40.1/24      | DOMAIN (53)        | **ALLOW**  | Allow IoT VLAN to query DNS                                                                     |
| **IoT**        | IoT_Net         | !Private_Networks | Any                | **ALLOW**  | Allow IoT to ping the internet                                                                  |
| **IoT**        | IoT_Net         | Rasp_Pi           | SIEM_Ports         | **ALLOW**  | From IoT, allow Syslog, NetFlow, and Splunk Universal Forwarder to send logs to the Pi          |
| **IoT**        | Ring_Cameras    | !Private_Networks | Ring_Video         | **ALLOW**  | Allow Ring cameras to use the large port range required for video streaming                     |
| **IoT**        | Nintendo_Switch | !Private_Networks | Nintendo_Ports     | **ALLOW**  | Allow Nintendo Switch to use the large port range for online gaming                             |
| **IoT**        | IoT_Net         | !Private_Networks | IoT_Services       | **ALLOW**  | Allow ports needed by services on IoT devices                                                   |
| **IoT**        | IoT_Net         | Any               | Any                | **BLOCK**  | Explicitly block all traffic to log packets handled by this rule                                |
| **Sandbox**    | Sandbox_Net     | Any               | HTTPS (443)        | **REJECT** | Force failover to TCP by rejecting QUIC                                                         |
| **Sandbox**    | Sandbox_Net     | This Firewall     | Sandbox_Services   | **ALLOW**  | Allow proxied internet access from Sandbox                                                      |
| **Sandbox**    | Server address  | Any               | Any                | **ALLOW**  | Allow Proxy to communicate back to Sandbox clients                                              |
| **Sandbox**    | Sandbox_Net     | Rasp_Pi           | SIEM_Ports         | **ALLOW**  | From Sandbox, allow Syslog, NetFlow, and Splunk Universal Forwarder to send logs to the Pi      |
| **Sandbox**    | Sandbox_Net     | Any               | Any                | **BLOCK**  | Explicitly block all traffic to log packets handled by this rule                                |
| **Management** | MGMT_Net        | Rasp_Pi           | SIEM_Ports         | **ALLOW**  | From Management, allow Syslog, NetFlow, and Splunk Universal Forwarder to send logs to the Pi   |
| **Management** | 10.0.60.20      | Rasp_Pi           | UniFi_Ports        | **ALLOW**  | Allow AP to talk to it's controller                                                             |
| **Management** | MGMT_Net        | Any               | Web_Ports          | **ALLOW**  | Allows Management to access web ports                                                           |
| **Management** | MGMT_Net        | Rasp_Pi           | 5514               | **ALLOW**  | Allow UniFi Devices to send logs to controller                                                  |
| **Management** | MGMT_Net        | Any               | Any                | **REJECT** | Explicitly reject all traffic to log packets handled by this rule                               |
Note: Rules are separated in OPNsense by interface, and within each interface they are applied in top-down priority; specific inter-VLAN restrictions are placed above general WAN egress rules. 
