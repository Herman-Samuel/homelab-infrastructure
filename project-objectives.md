# Project Objectives

## **Primary Mission**
To transition from a flat home network (192.168.1.0/24) to a hardened, segmented infrastructure that balances daily productivity with a monitored, isolated environment for learning IT and cybersecurity through controlled experimentation.

## **Core Technical Objectives**
- **Logical Segmentation (Least Privilege):** Eliminate the single broadcast domain using 802.1Q VLAN tagging. Devices are separated by purpose into six segments (Main, Guest, Server, IoT, Sandbox, and Management; VLANs 10 through 60), with "Default Deny" inter-VLAN routing and explicit allow rules. This applies Zero Trust principles (no implicit trust between segments, access granted only when explicitly allowed) to limit lateral movement.
- **Management Plane Isolation:** Establish a dedicated Management VLAN (VLAN 60) for the switch and access point administrative interfaces, reachable only from a single administrative workstation. Limit the OPNsense web interface to the Main VLAN, and keep a separate physical emergency admin port on the router so that a misconfiguration cannot lock out management access.
- **Wireless Segmentation:** Deploy a dedicated VLAN-aware access point with SSIDs mapped to the Main, IoT, and Guest VLANs, with client isolation on Guest. The Sandbox and Management segments remain wired-only.
- **Monitoring & Telemetry:** Build a log-monitoring stack on a Raspberry Pi (VLAN 30) using Grafana, Loki, and Gradana Alloy in Docker, collecting syslog from the network infrastructure and displaying it on a dedicated dashboard for continuous visibility. This architecture prioritizes log and metadata analysis over full packet capture to conserve the Pi's CPU and storage, while keeping the option of on-demand Deep Packet Inspection (DPI) through temporary port mirroring for troubleshooting.
- **Research Containment:** Create a dedicated Sandbox VLAN (VLAN 50) with strict firewall rules to prevent lateral movement, proxied and limited WAN access through a Squid web proxy, and a "Strict Kill-Switch" to keep experiments and any future malware analysis contained.
- **Hardware-Accelerated Gateway:** Deploy a custom-built OPNsense appliance on an HP t620 Plus with an Intel i350-AM2 dual-port gigabit NIC. This leverages AES-NI for encrypted traffic and the stable Intel NIC driver for reliable routing, and the components were selected to balance security, reliability, speed, and cost. The ISP gateway runs in IP passthrough with its own firewall disabled, so OPNsense is the sole authority for routing and packet filtering and double NAT is avoided. The platform provides comprehensive control of the routing and firewall planes, granular filtering of East-West traffic between VLANs, and room to add IPS and NGFW features later.

## **Professional Governance**
- **Change Management:** Document every infrastructure modification through detailed work logs ("Actions Taken") and a change log, supported by a formal Request for Change (RFC) process. This includes a baseline gap analysis, technical justification, and a rollback plan, to simulate enterprise IT operations.
- **Technical Documentation:** Maintain before-and-after logical topologies, a Network Addressing Plan, firewall rule documentation, and a plain-language outage manual so household users can restore connectivity without the administrator.
- **Validation:** Produce visual evidence of security controls, including screen-recorded "Ping Audits" that verify the firewall rules against live firewall logs, and a recorded RFC presentation to a simulated Change Advisory Board.

## **Revisions to the Original Plan**
- **Logging platform:** The original plan used Splunk with Universal Forwarders. Splunk Enterprise does not run on the Raspberry Pi's ARM64 processor and is resource-heavy, so the plan changed to Grafana, Loki, and Gradana Alloy.
- **Management VLAN:** Originally described as "out-of-band." Because it shares physical links with other VLANs, it is more accurately a dedicated management VLAN; the separate physical admin port serves as the emergency path.
- **Printer placement:** Moved from the Server VLAN to the IoT VLAN during implementation.
- **Promtail End-of-Life:** Promtail reached end of life in March 2026; I plan to migrate to Grafana Alloy.
