# 🌐 Multi-Router WAN

[![Status](https://img.shields.io/badge/status-in--progress-yellow?style=for-the-badge)]()
[![Cisco Packet Tracer](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)]()
[![Routing](https://img.shields.io/badge/Routing-Static-blue?style=for-the-badge)]()
[![Routers](https://img.shields.io/badge/Routers-4-informational?style=for-the-badge)]()
[![License](https://img.shields.io/badge/License-MIT-lightgrey?style=for-the-badge)]()

> Project 02 in the Cisco Networking Portfolio series. A simulated multi-site company WAN built in Cisco Packet Tracer, focused on multi-router static routing across an asymmetric, chained topology.

---

## 📋 01. Project Overview

**Multi-Router WAN** simulates a fictional company, **NovaTech**, with three sites (Headquarters, Branch01, Branch02) connected through a dedicated backbone router. Unlike a simple point-to-point WAN link (Project 01), this project introduces **multiple routers and multi-hop static routing**, where some traffic must transit through an intermediate router to reach its destination.

This is Project 02 of a planned 10-project Cisco Networking Portfolio, building progressively from single-link routing toward more advanced enterprise networking topics (VLANs, OSPF, ACLs, security) in later projects.

## 🎯 02. Objectives

- Design and implement a realistic multi-router WAN topology
- Practice VLSM-based IP addressing across LAN and WAN address spaces
- Understand DCE/DTE roles and clock rate configuration on serial links
- Configure and troubleshoot multi-hop static routing
- Apply a Cisco IOS security baseline consistently across multiple devices
- Produce professional, portfolio-quality documentation

## ✅ 03. Requirements

- Four Cisco routers, three WAN links, four LAN segments (including a dedicated server subnet)
- Static routing only — no dynamic routing protocols
- Full end-to-end connectivity between all sites
- Deliberate failure-injection troubleshooting scenarios
- Basic Cisco IOS security baseline

## 🏗️ 04. Network Architecture

See [`documentation/architecture.md`](documentation/architecture.md) for the full breakdown of roles, interfaces, and design reasoning.

```
[HQ-Staff LAN] ── HQ-R ──(Link1)── Core-R ──(Link2)── Branch01-R ──(Link3)── Branch02-R ── [Branch02-LAN]
                    │                                      │
              [HQ-Server-Farm]                      [Branch01-LAN]
```

- **Core-R** is a dedicated transit router (no local LAN).
- **Branch01-R** has a dual role: local LAN + transit for Branch02.

## 🗺️ 05. Topology

*(Diagram to be added — see [`diagrams/`](diagrams/))*

## 🔗 06. WAN Design

| Link | Between | Subnet | DCE | DTE |
|---|---|---|---|---|
| Link1 | HQ-R ↔ Core-R | 10.0.0.0/30 | Core-R | HQ-R |
| Link2 | Core-R ↔ Branch01-R | 10.0.0.4/30 | Core-R | Branch01-R |
| Link3 | Branch01-R ↔ Branch02-R | 10.0.0.8/30 | Branch01-R | Branch02-R |

## 🧮 07. IP Addressing

| Segment | Subnet |
|---|---|
| HQ-Staff LAN | 172.16.0.0/29 |
| Branch01-LAN | 172.16.0.8/29 |
| Branch02-LAN | 172.16.0.16/29 |
| HQ-Server-Farm | 172.16.1.0/30 |
| Link1 | 10.0.0.0/30 |
| Link2 | 10.0.0.4/30 |
| Link3 | 10.0.0.8/30 |

Full addressing plan with usable ranges and broadcast addresses: [`documentation/ip-addressing.md`](documentation/ip-addressing.md)

## 🧩 08. Subnetting

VLSM breakdown and design reasoning: [`documentation/subnetting.md`](documentation/subnetting.md)

## 🖥️ 09. Device Inventory

| Device | Model | Role |
|---|---|---|
| HQ-R | Cisco 2621XM | HQ edge router |
| Core-R | Cisco 2811 | Backbone / transit router |
| Branch01-R | Cisco 2621XM | Branch01 edge + transit for Branch02 |
| Branch02-R | Cisco 2621XM | Branch02 edge router |
| 6× PC | Generic PC | End devices (2 per site) |
| 1× Server | Generic Server | HQ Server Farm |

## 🛠️ 10. Technologies

- Cisco IOS
- Cisco Packet Tracer
- Static routing
- VLSM (Variable Length Subnet Masking)
- Serial WAN links (DCE/DTE, clock rate)

## ⚙️ 11. Router Configuration

Full running-configs for all 4 routers are published under [`configs/`](configs/): [`HQ-R`](configs/HQ-R-config.txt), [`Core-R`](configs/Core-R-config.txt), [`Branch01-R`](configs/Branch01-R-config.txt), [`Branch02-R`](configs/Branch02-R-config.txt).

## 🧭 12. Routing Design

Static routing only. Details and per-router route tables: [`documentation/routing.md`](documentation/routing.md)

## 🧪 13. Testing & Validation

Full test plan and results (local, same-site, WAN, remote LAN, end-to-end, reverse path, path analysis) — all passing with 0% packet loss: [`documentation/testing.md`](documentation/testing.md)

## 🩺 14. Troubleshooting

One deliberate failure-injection scenario completed (missing static route) — full symptom/investigation/root-cause/fix writeup: [`documentation/troubleshooting.md`](documentation/troubleshooting.md). Additional scenarios planned.

## 🔒 15. Security Baseline

Basic Cisco IOS security baseline applied consistently across all routers (hostnames, `enable secret`, console/VTY authentication, `service password-encryption`). Details: [`documentation/security.md`](documentation/security.md)

## 💡 16. Lessons Learned

*(To be completed at the end of the project)*

## 🚀 17. Future Improvements

*(To be completed at the end of the project)*

## 📸 18. Screenshots

*(To be added during implementation and testing)*

## 📦 19. How to Open the Project

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer).
2. Clone this repository.
3. Open `PacketTracer/Multi-Router-WAN.pkt`.

## 👤 20. Author

**Sameh Nasrallah**
GitHub: [@engsamehnasrallah-web](https://github.com/engsamehnasrallah-web)

---

## 🔗 Related Projects

- [Project 01 — Multi-LAN Routed Network](#) *(link to be added)*

## 🗓️ Roadmap

See [`ROADMAP.md`](ROADMAP.md) for the full phased plan.

## 📜 Changelog

See [`CHANGELOG.md`](CHANGELOG.md).
