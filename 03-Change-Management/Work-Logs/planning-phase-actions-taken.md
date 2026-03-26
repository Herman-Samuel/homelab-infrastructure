#### 7 March 2026
- Took note of all devices I currently own that could be part of my network, even the Wi-Fi enabled water heater that is not currently connected.
- Separated devices into six categories based on what VLAN I plan to have them in
- Made a note of software I could think of from the top of my head to possibly use on my network; including a VPN, Ad Blocker, SIEM, and Packet Sniffer.
- Researched different ways to setup and manage the Management VLAN
- Researched how I would introduce a Synology NAS into the network and how it would be set up securely
- Researched how I would separate an old laptop onto its own VLAN that can't access any of the other VLANs, including setting up a kill switch or a DNS Sinkhole
- Researched pros and cons of managing my Raspberry Pi physically (separate keyboard/monitor) or remote (SSH/VNC from my main PC). This Pi will be on the Server VLAN monitoring the network and will be used for other cyber/ networking tools
- Researched exactly how modern ISP routers, dedicated access points, and managed switches worked. Also how they would be used together to create and access separate VLANs. Before I only had a general understanding of each.
- Researched how to use a mini-pc running OPNsense as my router and firewall, and the different plugins that would be helpful for it
- After today's research, here is the current plan to implement the new network architecture for my home lab
	- I will set my current AT&T modem to "IP Passthrough" so that traffic is only passed through the current router. That AT&T modem will connect to a mini-pc running OPNsense, which will act as my router and firewall for the network. On the OPNsense router, I will define subnets for six distinct VLANs: Main, Server, Sandbox, Management, IoT, and Guest. This architecture uses network segmentation to enforce the principle of least privilege, which reduces my attack surface and contains the blast radius of any potential compromise in my Lab or IoT environments. The router will act as the "doorway" between the networks, inspecting all inter-VLAN traffic. To ensure OPNsense has full visibility and control over incoming traffic, I will configure the AT&T BGW320 to disable its internal firewall features. This eliminates 'Double NAT' issues and ensures that my OPNsense firewall is the sole authority for packet inspection and threat mitigation. The OPNsense router then will connect to a managed switch via a trunk port. I will then configure specific access ports on the switch to provide direct ethernet access to certain VLANs. I will also have a dedicated VLAN-Aware Access Point connected into the managed switch for Wi-Fi access. I will map specific SSIDs to their respective VLAN IDs so that Main, IoT, and Guest have Wi-Fi access, while keeping the high-risk Sandbox and sensitive Management segments strictly ethernet only.
#### 8 March 2026
- Heavily researched hardware I would need for setting up this network, especially the computer I will use for the OPNsense router. Looked at many different options, considering the type of NIC, RAM, Storage, Processer, power consumption, and price.
	- Purchased HP Thin Client t620 Plus, Sandisk SSD 128GB, 65W HP Power Adapter, and an Intel-derived NIC. Total cost for OPNsense router: $116.43
#### 11 March 2026
- Changed structure of notes to look more professional
- Created a GitHub repository so that I can post my progress after I complete the planning phase.
- Made a description and README.md file for GitHub repo
#### 13 March 2026
- Planned possible future folders for Github repo
- Made Network Addressing Plan documentation and posted it to Github
	- To see this network plan, click [here](https://github.com/Herman-Samuel/homelab-infrastructure/blob/main/02-Hardened-Segmented-Design/network-plan.md)
- Researched Change Management steps
- Researched how to implement 802.1X authentication
#### 15 March 2026
- Purchased hardware from Micro Center for the network
	- Netgear Business 8-Port Gigabit Ethernet Switch (Managed Switch): $49.99
	- Ubiquiti UniFi U7 Lite (Access Point): $99.99
	- TP-Link USB 3.0 to Gigabit Ethernet Adapter: $13.99
	- inland CAT6 UTP RJ45 Ethernet Cables (3 pack): $8.99
#### 16 March 2026
- Started a baseline assessment for change management documentation to show how my network is currently set up
	- To see this baseline assessment, click [here](https://github.com/Herman-Samuel/homelab-infrastructure/blob/main/01-Legacy-Flat-Network/gap-analysis-and-risk-baseline.md)
- Listed out in a table all devices currently connected to the network, and started gathering information about each device (such as hostname, MAC & IP Addresses, Connection Type, etc.)
#### 19 March 2026
- Turned on developer mode on my android smart phone in order to learn about its advanced settings and options. Things I actually changed are:
	- Turned on 'Force Dark mode'
	- Turned on 'Sensors Off' tile
	- Selected an app to use for mocking GPS location
	- Downloaded Proton VPN to always be on, set to auto connect to the fastest location
- Updated the Githubs README to be more accurate by editing the 'Access Level' Column in the 'Planned Network Segments' table
- Updated the Githubs 'network-plan' table by: 
	- Added a 'DHCP Range' column. This column shows the planned IP address ranges that each segment will allow. The rest of the IP Addresses in each segment reserved for static use.
	- Added the 'Purpose' and 'Access Level' columns from the README.md file
- Drew up a network map of how my network setup currently is, called "legacy-architecture_flat-network-model.drawio". This diagram is useful to show the current network architecture and the networks security gaps, as well as be a resource in the future for comparison of my previous system
	- To see the network map click [here](https://github.com/Herman-Samuel/homelab-infrastructure/blob/main/01-Legacy-Flat-Network/legacy-architecture-diagram.png)
- Uploaded the current network architecture map I just made onto Github under the architecture folder
#### 21 March 2026
- Wrote out detailed documentation of the Legacy Architecture network map. This document steps through each portion of the diagram, as well as explains some of the security concerns related to the current architecture.
	- To read the network map documentation click [here](https://github.com/Herman-Samuel/homelab-infrastructure/blob/main/01-Legacy-Flat-Network/legacy-architecture_topology-and-security-review.md)
#### 22 March 2026
- Researched pros and cons of setting up a full time mirror port for the raspberry Pi. Decided against this as having a full time mirror port can quickly lead to packet drops, CPU throttling, and storage issues on a Raspberry Pi due to the sheer volume of data. Instead I will establish a pipeline for Syslogs, NetFlow, and log agents, to aggregate logs on the Pi with Splunk. I will have the capability for on-demand Deep Packet Inspection (DPI) via temporary port mirroring for advanced troubleshooting.
- Completely overhauled my previous project objectives that were in the README.md file on Github, and moved them to a dedicated file called [project-objectives](https://github.com/Herman-Samuel/homelab-infrastructure/blob/main/project-objectives.md) under the documentation folder
- Changed the [README.md](https://github.com/Herman-Samuel/homelab-infrastructure/blob/main/README.md) file on Github to be a high-level summary to quickly contextualize the technical complexity of the project for a visitor before they dive into the sub-folders
- Engineered a comprehensive firewall policy, [Firewall Rules](https://github.com/Herman-Samuel/homelab-infrastructure/blob/main/02-Hardened-Segmented-Design/firewall-rules.md), utilizing micro-segmentation and stateful inspection to enforce a Zero-Trust posture across six functional VLANs, ensuring strict isolation of high-risk segments like the Sandbox and IoT networks
- Conducted a [Pre-Migration Baseline Assessment](https://github.com/Herman-Samuel/homelab-infrastructure/blob/main/01-Legacy-Flat-Network/gap-analysis-and-risk-baseline.md) of the existing flat ISP network, documenting current IP schemes, hardware firmware, and security gaps (lack of segmentation, shared management, etc.) to serve as a benchmark for the OPNsense security implementation
- Uploaded the following documents to Github and merged the branch these edits were uploaded onto with main:
	- legacy-architecture_topology-and-security-review.md (under documentation)
	- project-objectives.md (under documentation)
	- pre-migration-baseline-assessment.md (under change-management)
	- firewall-rules.md (under opnsense)
- Sent all of the documents I've made so far to a friend who is working towards becoming a Network Engineer to get a peers opinion while still in the planning phase
#### 23 March 2026
- Drafted a comprehensive [Change Management Implementation Plan](https://github.com/Herman-Samuel/homelab-infrastructure/blob/main/03-Change-Management/change-implementation-plan.md) for the migration from a flat ISP network to a segmented OPNsense environment. Included specific technical procedures for IP Passthrough, VLAN Tagging, and a formal Rollback Plan to ensure 99.9% network availability during the transition.
- Authored a formal [Request For Change (RFC)](https://github.com/Herman-Samuel/homelab-infrastructure/blob/main/03-Change-Management/request-for-change-01.md) for the 'Hardened Perimeter' project. Translated complex technical configurations (VLANs, ACLs, PVIDs) into business-centric language focused on Risk Mitigation, Data Isolation, and Operational Visibility for presentation to a simulated Change Advisory Board (my wife).
- Developed a non-technical Troubleshooting Protocol for household residents, [Outage Emergency Manual](https://github.com/Herman-Samuel/homelab-infrastructure/blob/main/04-System-Operations/outage-emergency-manual.pdf). This ensures that household internet services remain resilient and manageable for all users, even when the primary administrator is unavailable. It transforms complex technical troubleshooting into a simple, visual process that empowers non-technical residents to restore connectivity safely and confidently.
- Made a template [Change Log](https://github.com/Herman-Samuel/homelab-infrastructure/blob/main/03-Change-Management/change-log.md) so that as soon as the first change is made I can document it. This provides the "what and when" of every change that is made to the network. If something breaks or I forget what I did, I can use this log to look back at the specific configurations that I changed.
- Changed file name to better reflect what it is: pre-migration-baseline-assessment to [gap-analysis-and-risk-baseline](https://github.com/Herman-Samuel/homelab-infrastructure/blob/main/01-Legacy-Flat-Network/gap-analysis-and-risk-baseline.md). Added an executive summary. Changed column name in "Risk Table" from "Security Impact" to "Identified Gap". Added 4 entries to the "Risk Table", expanding the identified gaps in the network. 
- Uploaded the following documents to Github branch and merged to main:
	- change-implementation-plan.md (under change-management)
	- request-for-change-01.md (under change-management)
	- outage-emergency-manual.pdf (under documentation)
	- change-log.md (under change-management)
	- gap-analysis-and-risk-baseline.md (under documentation, moved from change-management)
#### 24 March 2026
- Re-structured the GitHub repository from a mix of Categorial (Type-based) and Functional (Topic-based) directories to be solely Functional based. For example instead of having a "documentation" folder, documentation will go in whatever topic based folder makes since (e.g. gap-analysis-and-risk-baseline.md will go into "01-Legacy-Flat-Network")
- GitHub is now structured into 4 directories:
	- 01-Legacy-Flat-Network
	- 02-Hardened-Segmented-Design
	- 03-Change-Management
	- 04-System-Operations
#### 25 March 2026
- Drew a comprehensive diagram for the [hardened, segmented network architecture](https://github.com/Herman-Samuel/homelab-infrastructure/blob/main/02-Hardened-Segmented-Design/hardened-architecture-diagram.png) that is planned. This shows the new flow of traffic, security concerns that were fixed (green text), and each VLAN with what they are allowed access to.
- Drafted a comprehensive [Hardened Architecture: Topology Review](https://github.com/Herman-Samuel/homelab-infrastructure/blob/main/02-Hardened-Segmented-Design/hardened-architecture-topology-review.md) which provides a granular, step-by-step technical walkthrough of the segmented network design, explicitly detailing the transition from "Red" legacy vulnerabilities to "Green" mitigations like 802.1Q Micro-segmentation, Out-of-Band management, and Separate DHCP Pools.
- Scripted the formal Request for Change (RFC) live presentation for the primary household stakeholder.
- Uploaded the following documents to Github branch and merged to main:
	- hardened-architecture-diagram.png, under Hardened Segmented Design
	- hardened-architecture-topology-review.md, under Hardened Segmented Design
- Delivered a formal [Request for Change (RFC) Presentation](https://youtu.be/W24WhC4zqCA), modeled as a live executive briefing, which successfully secured project approval and a $290 budget by translating complex technical risks (Lateral Movement, Visibility Gaps) into clear household benefits and providing a user-friendly Emergency Operations Manual to ensure long-term system reliability. Videoed presentation to be uploaded onto GitHub.
- Edited RFC Presentation video to smoothly transition from the intro, to the GitHub screen recording, then to the closing.
- Completed the Planning Phase. Successfully consolidated architectural blueprints, risk-mitigation logic, and change implementation plans into a project roadmap. Verified all documentation, including the Gap Analysis, RFC, Topology Reviews, and Emergency Manual, for Portfolio Audience review.
- Updated my Portfolio Website's "Projects" page with a link to this projects GitHub. [Portfolio Website](https://samuelrherman.com/) 
- Made a LinkedIn Post to inform potential employers and peers about my project (Scheduled to post 26 March 2026). [LinkedIn Profile](https://www.linkedin.com/in/herman-samuel/)
- Uploaded the following documents to Github branch and merged to main:
	- request-for-change-01-presentation.mp4, under Change Management
	- planning-phase-actions-taken.md
