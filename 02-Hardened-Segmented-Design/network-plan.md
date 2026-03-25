# Network Addressing Plan

| **VLAN ID** | **Name**   | **Subnet**   | **Gateway** | **DHCP Range** | **Purpose**                          | **Access Level**                       |
|-------------|------------|--------------|-------------|----------------|--------------------------------------|----------------------------------------|
| 10          | Main       | 10.0.10.0/24 | 10.0.10.1   | .100 - .250    | Trusted Personal Devices             | Full Access                            |
| 20          | Guest      | 10.0.20.0/24 | 10.0.20.1   | .100 - .250    | Visitors / Untrusted                 | Internet Only                          |
| 30          | Server     | 10.0.30.0/24 | 10.0.30.1   | Static Only    | Shared Services / Security Stack     | Stateful Service / Restricted Push     |
| 40          | IoT        | 10.0.40.0/24 | 10.0.40.1   | .100 - .250    | Cameras / Smart Home                 | No LAN / Restricted WAN                |
| 50          | Sandbox    | 10.0.50.0/24 | 10.0.50.1   | .100 - .150    | Malware Lab / Research               | No LAN / Proxied Egress                |
| 60          | Management | 10.0.60.0/24 | 10.0.60.1   | Static Only    | Local Out-of-Band Administration     | Inbound Only / No WAN                  |
