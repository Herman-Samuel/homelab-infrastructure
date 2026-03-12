# **Project Objectives**
The primary goal of this lab is to move away from a "Flat Network" and implement a hardened infrastructure that balances daily productivity with a high-risk research environment.

- **Network Isolation:** Utilize 802.1Q VLAN tagging to physically and logically separate trusted traffic from untrusted IoT devices and high-risk research segments.
- **Security Posture:** Implementation of a "Default Deny" firewall policy between VLANs to prevent lateral movement (Zero Trust).
- **Packet Analysis:** Establish a mirror port to facilitate Deep Packet Inspection (DPI) via Wireshark and forward telemetry to a centralized SIEM (Splunk) for long-term behavioral analysis and alerting.
- **Hardware Efficiency:** Repurpose x86-64 hardware into a high-performance OPNsense router and firewall. Hand selected components, considering security, reliability, speed, and price.
  - **Chassis:** HP t620 Plus (Thin Client)
  - **CPU:** AMD GX-420CA 2.0GHz Quad-Core
  - **Memory:** 8GB DDR3 RAM
  - **Storage:** 128GB SanDisk X400 M.2 SATA SSD
  - **Network:** Dual-port Intel i350-AM2 PCIe NIC
- **Architecture Diagram:** High-level Logical Topology created in Draw.io
## Planned Network Segments
| VLAN ID  | Name       | Purpose                      | Access Level                          |
|----------|------------|------------------------------|---------------------------------------|
| 10       | Main       | Personal Devices             | Full Access                           |
| 20       | Guest      | Visitors / Untrusted Mobile  | Internet Only                         |
| 30       | Server     | Network Monitoring / Storage | Restricted / Intra-VLAN Only          |
| 40       | IoT        | Smart Home / Cameras         | Isolated / No WAN                     |
| 50       | Sandbox    | Malware Lab / Research       | Strict Kill-Switch / No Local Access  |
| 60       | Management | Out-of-Band Administration   | Locked / Administrative Only (No WAN) |
