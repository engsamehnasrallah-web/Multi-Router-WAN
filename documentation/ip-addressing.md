# IP Addressing Plan

## Address Space

- **LANs:** `172.16.0.0/16`
- **WAN Links:** `10.0.0.0/8`

Two separate private ranges were used deliberately so that user/server traffic (LAN) and inter-site traffic (WAN) can be distinguished at a glance — useful for future ACL/filtering work.

## Full Addressing Table

| # | Subnet | Prefix | Usable Range | Broadcast | Segment |
|---|---|---|---|---|---|
| 1 | 172.16.0.0 | /29 | .1 – .6 | 172.16.0.7 | HQ-Staff LAN |
| 2 | 172.16.0.8 | /29 | .9 – .14 | 172.16.0.15 | Branch01-LAN |
| 3 | 172.16.0.16 | /29 | .17 – .22 | 172.16.0.23 | Branch02-LAN |
| 4 | 172.16.1.0 | /30 | .1 – .2 | 172.16.1.3 | HQ-Server-Farm |
| 5 | 10.0.0.0 | /30 | .1 – .2 | 10.0.0.3 | Link1 (HQ-R ↔ Core-R) |
| 6 | 10.0.0.4 | /30 | .1 – .2 | 10.0.0.7 | Link2 (Core-R ↔ Branch01-R) |
| 7 | 10.0.0.8 | /30 | .1 – .2 | 10.0.0.11 | Link3 (Branch01-R ↔ Branch02-R) |

`172.16.0.24/29` was left unused/reserved.

## Per-Device Assignment

| Device | Interface | IP Address | Subnet Mask |
|---|---|---|---|
| HQ-R | FastEthernet0/1 | 172.16.0.1 | 255.255.255.248 |
| HQ-R | FastEthernet0/0 | 172.16.1.1 | 255.255.255.252 |
| HQ-R | Serial0/0 | 10.0.0.1 | 255.255.255.252 |
| PC0 | NIC | 172.16.0.2 | 255.255.255.248 |
| PC1 | NIC | 172.16.0.3 | 255.255.255.248 |
| Server0 | NIC | 172.16.1.2 | 255.255.255.252 |
| Core-R | Serial0/0/0 | 10.0.0.2 | 255.255.255.252 |
| Core-R | Serial0/0/1 | 10.0.0.5 | 255.255.255.252 |
| Branch01-R | FastEthernet0/1 | 172.16.0.9 | 255.255.255.248 |
| Branch01-R | Serial0/1 | 10.0.0.6 | 255.255.255.252 |
| Branch01-R | Serial0/0 | 10.0.0.9 | 255.255.255.252 |
| PC2 | NIC | 172.16.0.10 | 255.255.255.248 |
| PC3 | NIC | 172.16.0.11 | 255.255.255.248 |
| Branch02-R | FastEthernet0/1 | 172.16.0.17 | 255.255.255.248 |
| Branch02-R | Serial0/0 | 10.0.0.10 | 255.255.255.252 |
| PC4 | NIC | 172.16.0.18 | 255.255.255.248 |
| PC5 | NIC | 172.16.0.19 | 255.255.255.248 |

> **Note on physical port numbering:** the logical design originally listed HQ-R's LAN interfaces and Branch01-R's serial interfaces in a specific order. During implementation the physical ports ended up reversed (HQ-R: Fa0/0 = Server-Farm, Fa0/1 = Staff LAN; Branch01-R: S0/0 = Link3 toward Branch02-R, S0/1 = Link2 toward Core-R). The IP addressing itself is unaffected — only the port *labels* differ from the original plan. This table reflects the actual as-built configuration.

## Design Reasoning

- All LAN subnets sized to `/29` (6 usable) even though each only strictly needs 3 addresses (2 PCs + gateway) — small headroom kept for simplicity and future device additions without resubnetting.
- The Server Farm was intentionally placed in a separate octet (`172.16.1.x`) rather than continuing sequentially in `172.16.0.x`, to keep room for future site/server growth without disturbing existing LAN subnets.
- WAN links use `/30` (2 usable addresses) — the minimum needed for a point-to-point link, avoiding address waste.
