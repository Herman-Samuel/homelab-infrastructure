# Home Lab: Network Segmentation & Hardening (OPNsense)
Note: A [non-technical analogy](#analogy-for-dummies-non-technical) is at the bottom, for readers who want a plain-language explanation.
## Project overview

I replaced my flat home network, where every device could reach every other device, with a segmented, enterprise-style design built around a custom OPNsense router, a managed switch, and a UniFi access point. The router is a repurposed HP t620 Plus thin client. Its processor supports AES-NI hardware encryption, and an Intel i350 dual-port gigabit NIC provides stable routing.

The network is divided into six VLANs using 802.1Q tagging: Main (10), Guest (20), Server (30), IoT (40), Sandbox (50), and Management (60). A default-deny firewall policy with explicit allow rules limits lateral movement between segments, and the OPNsense web interface listens only on the Main VLAN. A dedicated Management VLAN holds the administrative interfaces of the switch and access point, and a separate physical admin port on the router provides emergency access if a misconfiguration locks me out.

## Status

| Component | Status |
| --- | --- |
| OPNsense router, VLANs, DHCP, and DNS | Complete |
| Managed switch (802.1Q trunk and access ports) and UniFi access point with SSID-to-VLAN mapping | Complete |
| Default-deny firewall rules and ping-audit validation | Complete |
| Change-management documentation | Complete, updated as changes are made |
| Log monitoring (Grafana, Loki, Promtail) | In progress |
| Sandbox web proxy (Squid) | In progress |

## Governance and validation

The migration followed a structured change-management process: a baseline gap analysis of the original network, an implementation plan with a rollback procedure, a formal Request for Change (presented as a [video briefing](https://youtu.be/W24WhC4zqCA)), a change log, and before/during/after snapshots of device configurations. I also wrote a plain-language outage manual so the rest of my household can restore connectivity without me.

I validated the firewall policy with [recorded ping audits](https://youtu.be/UIDVXIaYgEY) between every VLAN, shown alongside the firewall's live logs.

## Monitoring (in progress)

A Raspberry Pi on the Server VLAN hosts the self-hosted UniFi controller and is being built into a log-monitoring stack using Grafana, Loki, and Promtail in Docker. I originally planned to use Splunk, but Splunk Enterprise does not run on the Pi's ARM processor, so I switched to a lighter stack.

## Purpose

The lab is the backbone of my home network and a safe, isolated place to learn IT and cybersecurity by building, breaking, and fixing things. The Sandbox VLAN is set aside for controlled experiments, and its internet access is limited to a Squid web proxy (in progress).

## Design decisions

- **Log analysis instead of a permanent mirror port.** I considered mirroring all traffic to the Raspberry Pi, but a full-time mirror port risks packet drops, CPU throttling, and storage problems on a Pi. I chose a log-based pipeline instead, with temporary port mirroring available on demand for deep packet inspection.
- **OPNsense as the only firewall.** The ISP gateway runs in IP passthrough with its Wi-Fi and packet filtering turned off, which avoids double NAT and makes OPNsense the sole authority for routing and filtering.
- **Scoped rules for noisy devices.** The Ring cameras and a game console each need a wide range of ports. I gave them static DHCP reservations so those permissive rules apply to those devices only, not the whole IoT VLAN.
- **Aliases for maintainability.** Firewall rules reference named aliases for hosts, subnets, and port groups, so a change is made in one place.
- **Temporary access, then removed.** When the Pi needed internet access to install Docker and software, I created a narrow temporary rule and disabled it afterward.
- **Wireless security by segment.** Main uses WPA3 with protected management frames required, Guest has client isolation, and IoT uses WPA2 for device compatibility.
- **Verified downloads.** I checked file hashes and signatures for the OPNsense image and Linux ISOs before flashing them.
- **Self-hosted controller.** The UniFi controller runs in Docker on the Pi (Server VLAN), so the access point is managed locally, with no cloud account.

## Troubleshooting highlights

- **Router boot loop.** After I added two flow-control tunables to chase inconsistent Wi-Fi speeds, the router stopped starting. First it gave a hardware error beep code, which a CMOS reset cleared. Then OPNsense crashed during boot. I tried unsetting the tunables at the loader prompt, then booted the installer USB, imported the ZFS pool, and removed them from the configuration file. Neither fixed it. Verbose boot output showed the crash right after the EFI framebuffer initialized, and forcing a basic text console in the loader configuration resolved it.
- **Access point traffic blocked by my own rules.** Firewall live view showed the access point's DNS, NTP, and logging traffic hitting the Management VLAN's deny rule. I added narrowly scoped allow rules for that traffic. Later speed tests suggested the slow results came from running speed tests back to back, so I stopped tuning.
- **UniFi controller authentication loop.** The controller container failed to start because its database user was never created. I rebuilt the Docker Compose configuration to define that user and relaunched the stack.
- **Sandbox laptop install.** An encrypted-root install of Lubuntu kept failing at the GRUB stage. After checking partitions, crypttab, initramfs, and the bootloader from a live USB, I changed the design: the root partition is unencrypted, swap is encrypted, and the splash screen is disabled so the passphrase prompt appears reliably.

## Hardware and cost

The Request for Change secured a **$290** budget.

| Item | Role | Cost |
| --- | --- | --- |
| HP t620 Plus thin client, 128 GB SanDisk SSD, 65 W adapter, Intel NIC (bundle) | OPNsense router and firewall | $116.43 |
| Netgear 8-port managed gigabit switch | 802.1Q trunk and access ports | $49.99 |
| Ubiquiti UniFi U7 Lite | Wi-Fi with SSID-to-VLAN mapping | $99.99 |
| TP-Link USB 3.0 to gigabit adapter | Wired connection for the Sandbox laptop | $13.99 |
| Inland CAT6 patch cables (3-pack) | Cabling | $8.99 |
| **Planned total** | | **$289.39** |

Unplanned purchases:

| Item | Reason | Cost |
| --- | --- | --- |
| Replacement Intel i350 NIC | The first NIC arrived cracked and was returned | $40.29 |
| HP M.2 standoff screw | Needed to mount the SSD | $16.11 |
| Ubiquiti PoE+ injector | The access point needs PoE+, which the switch does not provide | $21.89 |

Actual spending was $367.68 (roughly $78 over budget) before the refund for the returned NIC.

Reused equipment: a Raspberry Pi, an old Dell laptop for the Sandbox, and the ISP gateway.

**Lesson learned:** I did not check how the access point would be powered before buying it, which caused a delay and an extra purchase. I now check power and compatibility requirements before ordering hardware.

## Repository guide

| Folder / File | Contents |
| --- | --- |
| `01-Legacy-Flat-Network` | Original network: gap analysis, diagram, and topology review |
| `02-Hardened-Segmented-Design` | Network plan, firewall rules, architecture diagram, and firewall testing video |
| `03-Change-Management` | Implementation plan, RFC, change log, snapshots, and work logs |
| `04-System-Operations` | Outage manual and operational documentation |
| `project-objectives.md` | The project's objectives, including revisions to the original plan |

## Analogy "For Dummies" (Non-technical)
Before the change, my home internet was like a giant studio apartment with no internal walls. The front door is locked, but once someone is inside, they have a key to every drawer, the safe, and the jewelry box. If a 'smart' lightbulb or a cheap security camera gets hacked, the hacker is standing in the middle of the room with our sensitive information.

My project is like remodeling that studio into a secure mansion.
- VLANs (The Walls): Building solid walls and locking the doors between the 'public' areas (like the Guest Wi-Fi or the TV) and our 'private' areas (like my computer and mobile phone).
- OPNsense (The Security Guard): Installing a smart security guard at the front door who checks everyone's ID. He also watches the hallways to make sure the 'Guest' in the living room isn't trying to sneak into the 'Private' bedroom.
- The Result: Even if a hacker 'breaks in' through a cheap smart device, they are trapped in one room and can’t see or touch anything else in the rest of the house. It makes our digital life much safer and more organized.
