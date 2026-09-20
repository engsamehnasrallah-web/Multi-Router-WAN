# Network Architecture — Multi-Router WAN

## 1. Business Scenario

**NovaTech** is a company with three sites:

- **Headquarters (HQ)** — main office, hosts staff workstations and a dedicated server (file/resource server).
- **Branch 01** — a secondary office that needs continuous access to HQ resources.
- **Branch 02** — a third, more remote site.

The sites are **not** all directly connected to HQ. Instead, the network uses a **chain-style, asymmetric topology** to force genuine multi-hop routing rather than a simple hub-and-spoke design, so that some traffic must traverse an intermediate router to reach its destination.

## 2. Topology Overview

```
[HQ-Staff LAN] ── HQ-R ──(Link1)── Core-R ──(Link2)── Branch01-R ──(Link3)── Branch02-R ── [Branch02-LAN]
                    │                                      │
              [HQ-Server-Farm]                      [Branch01-LAN]
```

- **Core-R** is a dedicated **transit router**: it has no directly attached LAN and exists purely to interconnect the three sites.
- **Branch01-R** plays a **dual role**: it serves its own local LAN *and* acts as a transit router for Branch02-R's traffic toward the rest of the network.
- Any traffic from **Branch02** to **HQ** or **Core** must pass through **Branch01-R** — this is the deliberate multi-hop learning point of the project.

## 3. Routers and Roles

| Router | Role | Hardware Model | Reasoning |
|---|---|---|---|
| **HQ-R** | Edge router for HQ site | Cisco 2621XM | Built-in LAN interfaces; only needs 1 WAN interface |
| **Core-R** | Backbone / transit-only router | Cisco 2811 | Needs 3 serial interfaces; modular chassis supports this without excess |
| **Branch01-R** | Edge router for Branch01 + transit for Branch02 | Cisco 2621XM | Built-in LAN; needs 2 WAN interfaces (dual role) |
| **Branch02-R** | Edge router for Branch02 (end of chain) | Cisco 2621XM | Built-in LAN; only needs 1 WAN interface |

## 4. Interfaces per Router

| Router | LAN Interfaces | Serial (WAN) Interfaces | Total |
|---|---|---|---|
| HQ-R | 2 (Staff LAN, Server Farm) | 1 (↔ Core-R) | 3 |
| Core-R | 0 | 2 (↔ HQ-R, ↔ Branch01-R) | 2 |
| Branch01-R | 1 (Branch01 LAN) | 2 (↔ Core-R, ↔ Branch02-R) | 3 |
| Branch02-R | 1 (Branch02 LAN) | 1 (↔ Branch01-R) | 2 |

> Note: the HQ Server Farm was deliberately placed on its **own physical interface and subnet**, separate from the HQ staff LAN, rather than using sub-interfaces (Router-on-a-Stick) — kept simple and explicit at this project's stage.

## 5. WAN Links

| Link | Between | Subnet | DCE (clock rate) | DTE |
|---|---|---|---|---|
| Link1 | HQ-R ↔ Core-R | 10.0.0.0/30 | Core-R | HQ-R |
| Link2 | Core-R ↔ Branch01-R | 10.0.0.4/30 | Core-R | Branch01-R |
| Link3 | Branch01-R ↔ Branch02-R | 10.0.0.8/30 | Branch01-R | Branch02-R |

> **DCE/DTE decision:** since there is no external ISP in this scenario (all routers belong to the same company), the DCE role is assigned to whichever router is **closer to the network's backbone (Core-R)** on each link. On Link1 and Link2, Core-R itself is one of the two endpoints, so it holds the DCE role directly. Link3 does not touch Core-R at all (it connects Branch01-R and Branch02-R), so the same principle is applied one level down the chain: **Branch01-R**, being the router closer to Core, holds the DCE role there instead.

## 6. LANs and Server Farm

| Site | Subnet | Hosts |
|---|---|---|
| HQ-Staff LAN | 172.16.0.0/29 | 2 PCs + gateway |
| HQ-Server-Farm | 172.16.1.0/30 | 1 server + gateway |
| Branch01-LAN | 172.16.0.8/29 | 2 PCs + gateway |
| Branch02-LAN | 172.16.0.16/29 | 2 PCs + gateway |

The Server Farm was deliberately placed in a **separate octet** (`172.16.1.x` instead of continuing in `172.16.0.x`) to keep room for future growth — if more branches or servers are added later, the addressing scheme stays organized without needing to renumber existing subnets.

## 7. Routing Design

- **Static routing only** — no dynamic routing protocols (OSPF, EIGRP, RIP, BGP) are used in this project; they are reserved for future projects in the portfolio series.
- Each router requires static routes for every remote subnet it does not directly connect to, pointing to the correct next-hop.
- Branch02-R's routes to HQ/Core, and HQ-R/Core-R's routes to Branch02, must transit through Branch01-R — this is the core learning objective of the project.

## 8. Security Baseline

A basic Cisco IOS security baseline is applied consistently across all four routers (scope limited — this is not a dedicated security project):

- Hostnames per router
- `enable secret` for privileged EXEC access
- Console line authentication
- VTY (remote) line authentication
- `service password-encryption`

## 9. Design Principles Followed

- **VLSM over flat subnetting** — every subnet is sized to its actual need rather than uniformly using /24 (as in Project 01).
- **Separate address space per traffic type** — LANs use `172.16.0.0/16`, WAN links use `10.0.0.0/8`, making it easy to distinguish user/server traffic from inter-site links at a glance.
- **Scalability-aware addressing** — the Server Farm's placement in its own octet anticipates future expansion without disrupting existing subnets.
- **No dynamic routing** — kept intentionally static to reinforce manual understanding of routing tables, next-hops, and multi-hop path resolution before introducing dynamic protocols in later projects.
