## **Executive Summary:**
This Gap Analysis identifies critical security deficiencies in the current ISP-managed "Flat" network. By transitioning to a hardened, segmented architecture, we are closing a significant visibility gap and mitigating the risk of lateral movement across internal assets.
## Network Topology (Flat)
**Current State:** Single Broadcast Domain / Non-Segmented
1. **Gateway & Perimeter Details**
	- **Hardware Model:** AT&T BGW320-500 (Integrated ONT/Router).
	- **Primary Function:** Acts as the Edge Gateway, performing NAT (Network Address Translation) and basic stateful inspection.
	- **Management Plane:** In-Band Management. Administrative access to the WebGUI (192.168.1.254) is conducted over the same logical path as general user traffic, increasing the risk of credential sniffing.
2. **Logical Addressing & Services**
	- **IPv4 Scheme:** 192.168.1.0/24
	- **DHCP Configuration:** Shared Pool (Full Octet). The gateway does not distinguish between device types (e.g., a workstation vs. an IoT camera), preventing the application of granular security policies.
	- **DNS:** ISP-provided. No internal filtering or protective DNS is currently implemented at the gateway level.
	- **SSID (Wireless):** **Name:** Saki
	    - **Encryption:** WPA2-PSK
	    - **Frequency:** 2.4GHz & 5GHz (Converged/Same SSID).
## Edge Services & Perimeter Configuration
**Current Status:** Closed Perimeter / Internal Blindness
1. **Connectivity & NAT Protocol**
	- **UPnP (Universal Plug and Play):** **DISABLED/UNSUPPORTED.** The BGW320-500 does not support UPnP, preventing internal devices from autonomously creating dynamic port mappings. This reduces the risk of "shadow" services being exposed to the WAN.
	- **NAT Type:** Symmetric/Standard NAT. All internal hosts are translated to a single public IP address.
2. **Inbound Gateway Rules**
	- **Port Forwarding:** **NONE.** No manual Pinpoint rules are configured. All unsolicited inbound traffic from the WAN is dropped by the default stateful inspection policy.
	- **DMZ / IP Passthrough:** **DISABLED.** The gateway is currently operating in **Full Router Mode**. It is performing all DHCP, DNS, and Routing functions for the 192.168.1.0/24 segment.
3. **Security Features (ISP Standard)**
	- **Packet Inspection:** Basic Stateful Packet Inspection (SPI) is active.
	- **SIP ALG:** **ENABLED**. This often interferes with VoIP/Security camera signaling and will be disabled in the hardened OPNsense configuration.
	- **Firewall Level:** Set to "Default." Provides standard protection against common DoS attacks but lacks IDS/IPS or NGFW capabilities.
## Management & Access Baseline
**Current Status:** Vulnerable / In-Band Management
1. **Access Vectors**
	- **Primary Interface:** Web-Based Graphical User Interface (WebGUI).
	- **Access Method:** Accessible via any browser at 'http:// 192.168.1.254' from any device (Wired or Wireless) currently on the home network.
	- **Protocol:** Unencrypted HTTP (Port 80). Internal administrative traffic is sent in cleartext, susceptible to local packet capture.
2. **Credential Posture**
	- **Authentication Status:** **FACTORY DEFAULT.** The gateway utilizes the unique Device Access Code printed on the physical hardware label.
	- **Risk Level:** **HIGH.** While unique per device, physical access to the router or a simple photo of the label grants full administrative control. There is no multi-factor authentication (MFA) or integration with a centralized identity provider.
3. **Remote & External Access**
	- **Remote Management:** **DISABLED.** Access to the management interface from the WAN (Public Internet) is prohibited by the current firewall policy.
	- **Physical Console:** No dedicated Serial/Console port is utilized or available for out-of-band recovery.
## Firmware & Vulnerability Baseline
**Current Status:** Unpatched Risk
1. **Core Infrastructure Firmware**
	- **Gateway Hardware:** AT&T BGW320-500 (Revision 02001F00460050)
	- **Current Software:** Version 6.34.7
	- **Vulnerability Note:** Consumer-grade ISP firmware is often managed and pushed by the provider. In this baseline, the gateway represents a "Black Box" security risk where the user has no control over the patching cycle or the underlying OS vulnerabilities.
## Risk Table
| **Risk Category**             | **Current State (BGW320-500)**                                                                                                                 | **Identified Gap**                                                                                                                                                      | **Future State (OPNsense / HP T620)**                                                                                                      |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| **Segmentation**              | **Flat Network:** All devices (PC, IoT, Cameras) share a single broadcast domain (192.168.1.0/24)                                              | **Unrestricted Lateral Movement:** A compromised IoT device can directly scan and attack the Main Admin PC                                                              | **802.1Q VLAN Isolation:** Distinct security zones (Main, Guest, Server, IoT, Sandbox, Management) to prevent East-West traffic            |
| **Identity & Access**         | **Shared Management:** WebGUI is accessible via unencrypted HTTP from any device on the network                                                | **Unauthorized Access:** Any guest or compromised device on Wi-Fi can reach the router's login page via HTTP                                                            | **Out-of-Band (OOB)/ Encrypted Mgmt:** Management is restricted to a dedicated VLAN using **HTTPS/SSH** exclusively                        |
| **Credentials**               | **Default Credential Posture:** Reliance on a factory-printed, static access code on the physical hardware                                     | **Credential Interception:** Admin credentials can be sniffed via local packet capture (HTTP) by any device. Also, physical access or a photo grants full admin control | **Hardened IAM:** Rotated, complex credentials with optional MFA/Key-based auth for SSH                                                    |
| **Egress Filtering**          | **Implicit Allow:** No ability to restrict or monitor outbound traffic from specific internal devices. Devices can "phone home" to any IP/Port | **Unmonitored Exfiltration:** IoT or compromised hosts could communicate with Command & Control (C2) servers via any port undetected                                    | **Mediated Egress:** Sandbox traffic via Protocol-Break Proxy (Kill-Switch); IoT limited by Granular Egress ACLs to verified services only |
| **Visibility**                | **Black Box:** No firewall logging, NetFlow, system telemetry, or admin logs available to the user                                             | **Blind Incident Response:** No way to detect an ongoing attack or perform forensics after a breach                                                                     | **SIEM Integration:** Full log forwarding to **Splunk** via the Raspberry Pi collector for real-time alerting                              |
| **Vulnerability Mgmt**        | **ISP-Managed:** Firmware updates (v6.34.7) are controlled by the ISP with no user oversight                                                   | **Unpatched Exposure:** High risk of "forgotten" vulnerable assets; no control over critical gateway security patches                                                   | **Centralized Patching:** User-controlled FreeBSD/OPNsense updates; systematic dashboard for network asset health                          |
| **Network Services**          | **Basic DNS:** Uses ISP DNS servers with no content or malicious domain filtering                                                              | **DNS Hijacking/Privacy:** Susceptibility to tracking and redirection to malicious phishing sites                                                                       | **Unbound DNS:** Secure DNS resolution with automated blocking of known malicious domains                                                  |
| **Physical Link Reliability** | **Wireless Critical Asset:** Raspberry Pi server (192.168.1.191) connected via 5GHz Wi-Fi                                                      | **Telemetry Instability:** Wi-Fi interference or congestion can cause "drops" in security logs, leading to blind spots during an incident                               | **Hardwired Server Segment:** Raspberry Pi moved to a shielded Cat6 wired connection on a dedicated Server VLAN (30)                       |
| **DNS Integrity**             | **ISP-Managed DNS:** Standard resolution with no filtering of malicious domains.                                                               | **Content & Phishing Risk:** No "Safety Net" at the gateway; users can inadvertently visit known malware or phishing sites.                                             | **Hardened Unbound DNS:** Local resolution with automated "Blocklists" to stop malware and tracking at the source.                         |
| **Infrastructure Redundancy** | **Single Point of Failure:** No secondary way to manage the network if the main LAN port or Wi-Fi fails.                                       | **Management Lockout:** A single configuration error can lead to a "Total Blackout" requiring a full factory reset.                                                     | **Out-of-Band (OOB) Access:** Dedicated "Backdoor" port (Realtek) for emergency recovery without affecting the main network.               |
| **Protocol Security**         | **SIP ALG Enabled:** Hidden ISP setting that often breaks security camera signals and VoIP.                                                    | **Service Degradation:** Unnecessary "helpers" in the ISP firmware cause dropped connections for Ring/security devices.                                                 | **Clean-Pipe Optimization:** SIP ALG and unnecessary ISP "helpers" disabled to ensure stable, encrypted camera streams.                    |
## Hardware Status

| **Device Name**       | **Device Type** | **Make/ Model**                   | **Hostname**       | **MAC Address**   | **IP Address**         | **Software Stack (OS/ FW)**                      | **Connection Type** |
| --------------------- | --------------- | --------------------------------- | ------------------ | ----------------- | ---------------------- | ------------------------------------------------ | ------------------- |
| AT&T Router           | Router          | HUMAX BGW320-500                  | BGW320-500         | 90:d0:92:33:xx:xx | 192.168.1.254 (Static) | Proprietary/ 6.34.7                              | Fiber (WAN)         |
| Sam PC                | Desktop         | MSI MAG B650 Tomahawk             | Data-Golem         | d8:43:ae:43:xx:xx | 192.168.1.64           | Win 11/ AMI 1.B0                                 | Wired 2.5 GbE       |
| Kodi PC               | Desktop         | MSI MAG B650 Tomahawk             | Kodi               | d8:43:ae:43:xx:xx | 192.168.1.91           | Win 11/ AMI 1.B0                                 | Wired 2.5 GbE       |
| Sam Phone             | Smart Phone     | Samsung Galaxy S25 Ultra          | Samuel-s-S25-Ultra | de:37:06:30:xx:xx | 192.168.1.65           | Android 16                                       | Wi-Fi 5 GHz         |
| Kodi Phone            | Smart Phone     | Samsung Galaxy S25 Ultra          | dykota-s-S25-Ultra | de:b9:15:20:xx:xx | 192.168.1.67           | Android 16                                       | Wi-Fi 5 GHz         |
| Sam Watch             | Smart Watch     | Samsung Galaxy Watch 6            | SM-R930            | ae:8f:4b:ed:xx:xx | 192.168.1.68           | Wear OS 6                                        | Wi-Fi 2.4 GHz       |
| Xbox One              | Gaming Console  | Microsoft Xbox One S              | gaming-zone        | bc:83:85:4c:xx:xx | 192.168.1.77           | Xbox OS/ 10.0.26100.7010                         | Wi-Fi 5 GHz         |
| Nintendo Switch       | Gaming Console  | Nintendo Switch (Base model)      | Nintendo Switch    | 70:2c:09:66:xx:xx | 192.168.1.146          | Proprietary/ 22.0.0                              | Wi-Fi 5 GHz         |
| Raspberry Pi          | Mini-Server     | Raspberry Pi 5                    | raspberrypi        | 2c:cf:67:5e:xx:xx | 192.168.1.191          | Raspberry Pi OS (Debian GNU/Linux 12 (bookworm)) | Wi-Fi 5 GHz         |
| Camera Wi-Fi Extender | IoT Gateway     | Ring Chime Pro (2nd Gen)          | ChimePro-42        | 34:3e:a4:0d:xx:xx | 192.168.1.223          | Proprietary FW                                   | Wi-Fi 2.4 GHz       |
| Kitchen Camera        | IP Camera       | Ring Indoor Cam                   | Cam-Kitchen        | 9c:76:13:ea:xx:xx | Chime Internal         | Proprietary FW                                   | Wi-Fi (via Chime)   |
| Doorbell Camera       | IP Camera       | Ring Video Doorbell Wired         | Cam-Doorbell       | 54:e0:19:ce:xx:xx | Chime Internal         | Proprietary FW                                   | Wi-Fi (via Chime)   |
| Living-room Camera    | IP Camera       | Ring Indoor Cam                   | Cam-Livingroom     | 9c:76:13:f4:xx:xx | Chime Internal         | Proprietary FW                                   | Wi-Fi (via Chime)   |
| Garage Camera         | IP Camera       | Ring Indoor Cam                   | Cam-Garage         | 9c:76:13:f2:xx:xx | Chime Internal         | Proprietary FW                                   | Wi-Fi (via Chime)   |
| Driveway Camera       | IP Camera       | Ring Floodlight Cam Wired Plus    | Cam-Driveway       | 54:e0:19:ba:xx:xx | Chime Internal         | Proprietary FW                                   | Wi-Fi (via Chime)   |
| Front Camera          | IP Camera       | Ring Floodlight Cam Wired Plus    | Cam-Front          | 54:e0:19:be:xx:xx | Chime Internal         | Proprietary FW                                   | Wi-Fi (via Chime)   |
| Rear Camera           | IP Camera       | Ring Indoor Cam                   | Cam-Rear           | 9c:76:13:f4:xx:xx | Chime Internal         | Proprietary FW                                   | Wi-Fi (via Chime)   |
| TV                    | Smart TV        | LG UHD 70 Series 65 inch Smart TV | LGwebOSTV          | 24:e8:53:cd:xx:xx | 192.168.1.66           | webOS/ 6.5.3-47 (kisscurl-koli)                  | Wi-Fi 5 GHz         |
