# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [Unreleased] — targeting v1.0.0 (First Stable Release)

### Planned
- Troubleshooting Scenario 2: interface shutdown
- Troubleshooting Scenario 3: wrong IP address
- Troubleshooting Scenario 4: wrong subnet mask
- Troubleshooting Scenario 5: wrong next-hop / broken WAN link
- Final engineering self-review (scored per category)

## [0.9.0] - 2026-09-20 (Pre-release — one step from v1.0.0)

### Added
- Troubleshooting Scenario 1: missing static route on Core-R (symptom, investigation, root cause, fix, verification)
- Full documentation set: `ip-addressing.md`, `subnetting.md`, `wan-design.md`, `routing.md`, `testing.md`, `troubleshooting.md`, `security.md`, `lessons-learned.md`
- Final router configs published under `configs/`
- Network topology diagram (`diagrams/network-topology.svg`)
- Implementation screenshots (topology, per-router routing tables, end-to-end ping/tracert)
- `LICENSE` (MIT) and `SECURITY.md`

### Fixed
- Corrected Core-R's interface count in `architecture.md` (2 serial interfaces, not 3 — Core-R does not connect directly to Branch02-R)

## [0.2.0] - Static Routing & Validation

### Added
- Static routes configured on all 4 routers for full mesh reachability
- Full test plan executed: local, same-site, WAN, remote segment, end-to-end (4-hop), reverse path, and path analysis (`tracert`) — all passing with 0% packet loss

## [0.1.0] - Initial Design & Configuration

### Added
- Business scenario, requirements, and constraints defined (NovaTech, 3-site chained WAN topology)
- Architecture designed: 4 routers (HQ-R, Core-R, Branch01-R, Branch02-R), asymmetric chain topology, hardware selection (Cisco 2811 for Core-R, 2621XM for edge routers)
- VLSM addressing plan finalized across LAN (`172.16.0.0/16`) and WAN (`10.0.0.0/8`) address spaces
- Initial router configuration: hostnames, enable secret, console/VTY authentication, interface IP addressing, DCE/DTE clock rate assignment
- `README.md` and `documentation/architecture.md` drafted
