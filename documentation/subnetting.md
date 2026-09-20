# Subnetting (VLSM)

This project uses **Variable Length Subnet Masking (VLSM)** rather than a single flat mask applied to every segment (as in Project 01). Each subnet is sized to its actual host requirement.

## LAN Subnets — derived from 172.16.0.0/16, using /29 (block size 8)

Starting from `172.16.0.0/29`:

| Subnet | Network ID | Usable Range | Broadcast | Assigned To |
|---|---|---|---|---|
| 1 | 172.16.0.0 | .1 – .6 | 172.16.0.7 | HQ-Staff LAN |
| 2 | 172.16.0.8 | .9 – .14 | 172.16.0.15 | Branch01-LAN |
| 3 | 172.16.0.16 | .17 – .22 | 172.16.0.23 | Branch02-LAN |
| 4 | 172.16.0.24 | .25 – .30 | 172.16.0.31 | *(unused/reserved)* |

## Server Farm — /30, separate octet (172.16.1.0)

| Subnet | Network ID | Usable Range | Broadcast |
|---|---|---|---|
| 172.16.1.0/30 | 172.16.1.0 | .1 – .2 | 172.16.1.3 |

Deliberately placed in `172.16.1.x` instead of continuing `172.16.0.x` sequentially, to keep the core LAN octet clean for future branch expansion.

## WAN Subnets — derived from 10.0.0.0/8, using /30 (block size 4)

| Subnet | Network ID | Usable Range | Broadcast | Link |
|---|---|---|---|---|
| 1 | 10.0.0.0 | .1 – .2 | 10.0.0.3 | Link1 (HQ-R ↔ Core-R) |
| 2 | 10.0.0.4 | .1 – .2 | 10.0.0.7 | Link2 (Core-R ↔ Branch01-R) |
| 3 | 10.0.0.8 | .1 – .2 | 10.0.0.11 | Link3 (Branch01-R ↔ Branch02-R) |
| 4+ | 10.0.0.12+ | — | — | *(reserved for future links)* |

Sequential/contiguous subnets were used for WAN links (no octet-separation needed) since the WAN layer only ever contains router-to-router links — unlike LANs, it has no scalability concern requiring a dedicated address block.

## Why /30, Not /31, for Point-to-Point Segments

A `/31` (RFc 3021, no network/broadcast address) was considered for the HQ-Server-Farm subnet at one point, since it only needs 2 usable addresses. It was rejected because `/31` is conventionally reserved for router-to-router WAN links — a LAN segment with an end device (like a server) expects a standard network/broadcast structure. `/30` was used for both the Server Farm and all WAN links instead, keeping addressing conventional throughout the project.
