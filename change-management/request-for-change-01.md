# Request For Change (RFC) 2026-01
**Subject:** Migration to Segmented Security Architecture
**Requested By:** Samuel Herman, Network Security Lead 
**Priority:** Medium (Proactive Security Hardening) 
**Change Window:** Saturday, 20:00 – 22:00 (Low-impact window)

---
### **1. Executive Summary**
The current network environment operates on a "Flat" architecture, where all devices, ranging from critical administrative computers to high-risk Internet of Things (IoT) cameras, reside in the same digital space. This lack of separation means a single compromise on a low-security device (like a smart doorbell) grants an attacker an unimpeded path to sensitive personal data and financial records.

This proposal requests approval to migrate to a Segmented Architecture using professional-grade gateway hardware. This will create "Digital Silos" that isolate high-risk devices from critical systems.

---
### **2. Business & Security Justification**
- **Containment of Threats:** By placing IoT devices in their own isolated "zone," we prevent "Lateral Movement." If a camera is hacked, the attacker is trapped in that zone and cannot see the rest of the network.
- **Enhanced Visibility:** The new system allows for "Real-Time Monitoring." We will finally have the ability to see exactly what data is entering and leaving the network, which is currently a "Black Box" under the ISP's equipment.
- **Privacy & Control:** We are moving away from ISP-managed DNS (where the provider can track browsing habits) to a private, encrypted resolution system that we control.

---
### **3. Proposed Changes**
- **Hardware Upgrade:** Replacing the consumer-grade ISP router's "brain" with a dedicated, high-performance security appliance.
- **Infrastructure Zoning:** Establishing six distinct security zones (Main, Guest, Server, IoT, Sandbox, and Management).
- **Access Control:** Implementing a "Zero-Trust" policy where devices are only allowed to talk to the specific services they need to function.

---
### **4. Project Budget & Resource Allocation**
To achieve this enterprise-grade security, the following hardware investment is required. This represents a one-time capital expenditure to replace "rented" ISP functionality with owned, high-performance equipment.

| **Component**               | **Description**                                   | **Cost** |
| --------------------------- | ------------------------------------------------- | -------- |
| **Security Appliance**      | HP t620 Plus, 128GB SSD, Intel NIC, Power Adapter | $116.43  |
| **Managed Distribution**    | Netgear Business 8-Port Gigabit Switch            | $49.99   |
| **Wireless Infrastructure** | Ubiquiti UniFi U7 Lite (Access Point)             | $99.99   |
| **Emergency Access**        | TP-Link Gigabit Ethernet Adapter                  | $13.99   |
| **Connectivity**            | Cat6 Ethernet Cabling (3-Pack)                    | $8.99    |
#### **TOTAL PROJECT COST: $289.39**

---
### **5. Risk Assessment & Impact**
- **Service Interruption:** There will be a projected 1 to 2 hours of internet downtime during the hardware swap.
- **Device Re-provisioning:** Wireless devices will need to be moved to new, dedicated Wi-Fi names (SSIDs) to ensure they land in the correct security zones. 
- **Complexity:** The environment will require more active management than a standard "Plug-and-Play" router.

---
### **6. Implementation Strategy (High Level)**
1. **Preparation:** Documenting all current device settings to ensure no hardware is "lost" during the move.
2. **Conversion:** Setting the ISP equipment to "Pass-Through" mode, essentially turning it into a simple bridge.
3. **Deployment:** Activating the new security appliance and managed switch to distribute the new isolated zones.
4. **Verification:** Testing each zone to ensure "The Walls" are working as intended.

---
### **7. Back-out (Rollback) Plan**
In the event of a critical failure (e.g., hardware defect in the new appliance or inability to restore internet sync):
- **Action:** Disconnect the new hardware and perform a "Factory Reset" on the ISP gateway.
- **Result:** The network returns to its original, "Flat" state using the original factory settings. Total recovery time is estimated at 15 minutes.

---
### **8. Success Criteria**
1. Internet connectivity is restored to all baseline devices.
2. An "IoT" device is unable to communicate with a "Main" administrative computer.
3. All network activity is successfully logged and visible to the administrator.