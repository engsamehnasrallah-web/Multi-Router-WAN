# Routing Design

Static routing only — no dynamic routing protocols (OSPF, EIGRP, RIP, BGP) are used in this project. They are reserved for later projects in the portfolio series, so the focus here stays on manually understanding routing tables, next-hops, and multi-hop path resolution.

## Principle

Each router only needs a route for the *next hop* toward a remote network — not the full end-to-end path. Every router's static routes reflect only what it can't reach via a directly connected interface.

## Static Routes — Final Configuration

### HQ-R
```
ip route 172.16.0.8  255.255.255.248 10.0.0.2
ip route 172.16.0.16 255.255.255.248 10.0.0.2
```
Both remote LANs are reached via Core-R (`10.0.0.2`) — HQ-R has only one exit point (Serial0/0 toward Core-R), so every remote network uses the same next-hop.

### Core-R
```
ip route 172.16.0.0  255.255.255.248 10.0.0.1
ip route 172.16.1.0  255.255.255.252 10.0.0.1
ip route 172.16.0.8  255.255.255.248 10.0.0.6
ip route 172.16.0.16 255.255.255.248 10.0.0.6
```
HQ-Staff and HQ-Server-Farm are reached via HQ-R (`10.0.0.1`). Branch01-LAN and Branch02-LAN are both reached via Branch01-R (`10.0.0.6`) — Branch02-LAN is not directly connected to Core-R, so Core-R must forward it one hop further, to Branch01-R, which knows how to reach it.

### Branch01-R
```
ip route 172.16.0.0  255.255.255.248 10.0.0.5
ip route 172.16.1.0  255.255.255.252 10.0.0.5
ip route 172.16.0.16 255.255.255.248 10.0.0.10
```
HQ-Staff and HQ-Server-Farm are reached via Core-R (`10.0.0.5`). Branch02-LAN is directly reachable one hop further via Branch02-R (`10.0.0.10`), since Branch01-R sits directly next to it.

### Branch02-R
```
ip route 172.16.0.0  255.255.255.248 10.0.0.9
ip route 172.16.0.8  255.255.255.248 10.0.0.9
ip route 172.16.1.0  255.255.255.252 10.0.0.9
```
Branch02-R has only one exit point (toward Branch01-R, `10.0.0.9`), so all three remote networks (HQ-Staff, Branch01-LAN, HQ-Server-Farm) use the same next-hop.

## On Route Summarization

`172.16.0.8/29` and `172.16.0.16/29` (both reached via the same next-hop from Core-R and HQ-R) could theoretically be summarized into a single, wider supernet route. This wasn't applied here — with only 4 subnets, the benefit is minimal, and explicit per-network routes keep the routing table easier to read and reason about for learning purposes. Summarization becomes genuinely valuable at larger scale (hundreds of routes) and will be explored properly with dynamic routing protocols (OSPF/BGP) in later portfolio projects.
