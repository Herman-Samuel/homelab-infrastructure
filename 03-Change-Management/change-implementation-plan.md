# **Change Management Implementation Plan**
**Project:** Standard flat ISP migration to hardened OPNsense Gateway for VLAN segmentation, advanced firewall capabilities, and network monitoring
**Primary Hardware:** HP T620 (OPNsense router), AT&T BGW320-500 (Bridge), NETGEAR GS108E (managed switch), and Ubiquiti UniFi U7 Lite (access point)

#### **Step 1: Pre-Flight & Backup (The Safety Net)**
Before changing anything, I must ensure I can get back to how things were, if needed.
1. **Administrative Snapshot:** Log into 192.168.1.254 and take screenshots or record settings for:
	- **Broadband Status:** (Note Public IP, Default Gateway, and DNS).
	- **Home Network > Subnets & DHCP:** (Note the current range and any fixed allocations).
	- **Firewall > Advanced:** (Note which default toggles are ON).
2. **Hardware Prep:** Ensure the HP T620 has OPNsense pre-installed via USB.
3. **Local Admin PC:** Assign a Static IP to 'Sam PC' (192.168.1.10) temporarily. This prevents me from losing access if the DHCP server goes down during the swap.
---
#### **Step 2: Gateway Transition (IP Passthrough)**
This removes the ISP's control and hands the Public IP to the new router.
1. **Configure IP Passthrough:**
    - On the BGW320, go to Firewall > IP Passthrough.
    - Set Allocation Mode to Passthrough.
    - Set Passthrough Mode to DHCPS-Fixed.
    - Select the MAC Address of the HP T620's WAN port from the list.
2. **Disable ISP Wi-Fi:**
	- On the BGW320, go to Home Network > Wi-Fi
	- Click Advanced Options
	- Switch 2.4 GHz Wi-Fi Configuration and Home SSID to Off. The new access point will handle Wi-Fi
3. **Disable ISP Security:** 
	- On the BGW320, go to Firewall > Packet Filter
	- Turn off Packet Filter
	- Go to Firewall > Firewall Advanced
	- Turn everything to Off. The OPNsense will handle all security now
4. **Restart Cycle:** Power cycle the BGW320. Wait 2 minutes for it to sync with the ISP.
---
#### **Step 3: OPNsense Initial Handshake**
1. **Physical Link:** Connect the BGW320 (LAN Port 1) to the HP T620 (WAN Port).
2. **Interface Assignment:** Log into the OPNsense Console (via monitor/keyboard on the T620) and assign the physical ports to WAN and LAN.
3. **Verify WAN IP:** Ensure OPNsense pulls a Public IP from the BGW320. If it pulls a 192.168.1.x address (a private address), IP Passthrough isn't working yet.
---
#### **Step 4: NIC Assignment & "Backdoor" Configuration**
Map the software interfaces to the physical hardware ports in OPNsense.
1. **WAN Assignment (Intel NIC 1):** Assign to the port connected to the BGW320. Set to DHCP to pull the Public IP.
2. **LAN Trunk Assignment (Intel NIC 2):** Assign to the port connected to the managed switch. This port will not have an IP itself; it will house the VLAN Parent Interfaces.
3. **Emergency "Backdoor" (Built-in Realtek):** * Assign this port as a standalone interface (e.g. EMERGENCY).
    - **Static IP:** Give it a unique, non-VLAN IP like 10.99.99.1/24.
    - **DHCP:** Enable a small DHCP pool on this port.
    - **Rule:** Create a "Pass All" rule on this interface.
    - **Purpose:** If I accidentally misconfigure a VLAN tag or lock myself out of the switch, I can plug my laptop directly into the Realtek port to regain WebGUI access.
---
#### **Step 5: Configuring the Managed Switch (The "Trunk")**
The switch needs to know how to handle the "tagged" traffic coming from the HP T620.
1. **Trunk Ports (802.1Q Tagged):**
	- **Port 1:** Set the port connected to the HP T620 to Trunk Mode (tagged for VLANs 10, 20, 30, 40, 50, 60). Set PVID to 60, allowing the switch's own management IP to communicate with OPNsense.
	- **Port 8 (Access Point):** Configure as a Trunk Port with VLAN 60 as the Native VLAN (PVID) for AP management. Tagged for VLANs 10, 20, and 40 to support logically isolated SSIDs (Main, Guest, IoT).
2. **Access Ports:** Assign physical ports on the switch to specific VLANs (untagged/PVID):
    - **Ports 2-4:** VLAN 10 (Main: Desktops).
    - **Port 5-6:** VLAN 30 (Server: Raspberry Pi & NAS). 
    - **Port 7:** VLAN 50 (Sandbox: Dell laptop).
3. **Management IP:** Give the switch a static IP on the Management VLAN (10.0.60.20).
---
#### **Step 6: Configuring the Access Point (The "Air-Waves")**
Since the BGW320 Wi-Fi is off, the dedicated AP must handle the wireless segmentation.
1. **Management:** Connect the AP to switch port 8, which has VLAN 60 (Management) as the PVID and is tagged for VLANs 10, 20, 40.    
2. **SSID Mapping:**
    - **SSID "Saki_Main":** Map to VLAN 10.
    - **SSID "Saki_Guest":** Map to VLAN 20.
    - **SSID "Saki_IoT":** Map to VLAN 40.
3. **Isolate SSIDs:** Ensure the AP’s "Guest Policy" is active for VLAN 20 to prevent wireless clients from seeing each other.
#### **Step 7: VLAN & Logical Architecture Deployment**
Now that the hardware knows where to send the tags, tell OPNsense what those tags mean.
1. **Define VLAN Parent:** Navigate to Interfaces > Other Types > VLAN.
    - **Crucial:** Ensure the Intel NIC (LAN) is selected as the Parent Interface.
    - **Add Tags:** 10 (Main), 20 (Guest), 30 (Server), 40 (IoT), 50 (Sandbox), 60 (Mgmt).
- **Assign & Enable:** Go to Interfaces > Assignments.
    - Assign each new VLAN tag to a logical interface (e.g., VLAN10 on eth1 → MAIN).
    - Enable each interface and set the Static IPv4 address according to the plan (e.g., 10.0.60.1/24 for Mgmt), this is the gateway address for that VLAN.
- **DHCP Scopes:** Navigate to Services > DHCPv4.
    - Enable the DHCP Server for **each** VLAN.
    - Define the range based on the Network Addressing Plan and set the DNS Server to the gateway IP of that specific VLAN (e.g., 10.0.10.1 for Main) to ensure traffic stays local.
- **DNS (Unbound) Configuration:** Navigate to Services > Unbound DNS > General.
    - **Network Interfaces:** Ensure all new VLAN interfaces are selected so Unbound "listens" for requests from all segments.
    - **DNSSEC:** Enable DNSSEC for hardware-level validation of DNS records.
---
#### **Step 8: Security Policy (Firewall Rules)**
1. **Apply Rules:** Apply the "Zero-Trust" logic developed in firewall-rules.md.
2. **Emergency Access:** Ensure the Main_Admin_Host → MGMT_Net (ALLOW) rule is active first to prevent lockout.
3. **Logging:** Ensure "Log packets handled by this rule" is checked for all BLOCK rules to feed Splunk later.
---
#### **Step 9: Testing & Validation (The "Proof")**
Prove the "Walls" are solid before moving devices over.
1. Plug Sam PC into a VLAN 10 switch port. Verify IP 10.0.10.X.
2. Plug into the built-in Realtek Port on the t620. Verify IP 10.99.99.X and WebGUI access.
3. **Isolation Test:** Plug a laptop into a port assigned to the IoT VLAN. Try to ping 10.0.10.10 (Sam PC). It must fail.
4. **Egress Test:** From the Sandbox VLAN, try to browse to a non-standard port. It must fail. Try to browse via the Proxy. It should pass.
5. **Management Test:** Try to access 10.0.60.1 from the Guest Wi-Fi. It must be rejected.
---
#### **Step 10: Rollback Plan (In Case of Failure)**
If internet connectivity is not restored or OPNsense is unstable within 60 minutes:
1. Disconnect the HP T620.
2. Factory Reset the BGW320 (hold Reset button for 15s).
3. Re-configure the BGW320 using the screenshotted settings.
4. **Re-enable Wi-Fi:** Re-activate the "Saki" SSID to bring the legacy network back online.
5. Reconnect Sam PC directly to the BGW320 to restore "Legacy" wired connectivity.