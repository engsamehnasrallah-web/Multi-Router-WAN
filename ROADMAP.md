# Roadmap

## Phase 0 — Requirements & Scenario
- [x] Business scenario defined (NovaTech, 3 sites)
- [x] Requirements and constraints agreed

## Phase 1 — Architecture
- [x] Topology designed (asymmetric chain via Core-R)
- [x] Router roles and interface counts defined
- [x] Hardware selected (Cisco 2811 for Core-R, 2621XM for edge routers)
- [x] DCE/DTE assignment decided

## Phase 2 — WAN Topology Design
- [x] Router and link naming finalized

## Phase 3 — IP Addressing & Subnetting
- [x] VLSM plan for LANs (172.16.0.0/16)
- [x] VLSM plan for WAN links (10.0.0.0/8)
- [x] Full addressing table finalized

## Phase 4 — Initial Router Configuration
- [x] Hostnames, enable secret, console/VTY auth on all 4 routers
- [x] Interface IP addressing on all 4 routers
- [x] DCE clock rate configuration verified

## Phase 5 — Static Routing & Validation
- [x] Static routes configured on all 4 routers
- [x] Full test plan executed (local, same-site, WAN, remote segment, end-to-end, reverse path, tracert)

## Phase 6 — Troubleshooting Lab
- [x] Scenario 1: missing static route
- [x] Scenario 2: interface shutdown
- [x] Scenario 3: wrong IP address
- [x] Scenario 4: mismatched subnet mask
- [x] Scenario 5: wrong next-hop

## Phase 7 — Documentation
- [x] `README.md`
- [x] `architecture.md`
- [x] `ip-addressing.md`
- [x] `subnetting.md`
- [x] `wan-design.md`
- [x] `routing.md`
- [x] `testing.md`
- [x] `troubleshooting.md` (all 5 scenarios)
- [x] `security.md`
- [x] `lessons-learned.md`
- [x] Network topology diagram
- [x] Screenshots

## Phase 8 — GitHub Publication
- [x] Repository created
- [x] Final cleanup and full commit history
- [x] Tagged release (v1.0.0)

## Phase 9 — Final Engineering Review
- [x] Self-assessment across all technical dimensions (/10 per category)
