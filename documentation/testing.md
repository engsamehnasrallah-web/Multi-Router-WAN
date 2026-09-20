# Testing & Validation

All tests below were performed after full static routing configuration across all 4 routers.

## Test Plan and Results

| # | Test | From → To | Result |
|---|---|---|---|
| 1 | Local (PC → Gateway) | PC0 → 172.16.0.1 | ✅ 0% loss |
| 2 | Same-Site (PC → PC) | PC0 → PC1 (172.16.0.3) | ✅ 0% loss |
| 3 | WAN (Router → Router) | HQ-R → 10.0.0.2 (Core-R) | ✅ 0% loss |
| 4 | Remote Segment (PC → Server) | PC0 → 172.16.1.2 | ✅ 0% loss |
| 5 | End-to-End (furthest point, 4 hops) | PC0 → PC5 (172.16.0.19) | ✅ 0% loss (first packet timed out due to ARP resolution, then 0% loss) |
| 6 | Reverse Path | PC5 → PC0 (172.16.0.2) | ✅ 0% loss |
| 7 | Path Analysis | `tracert` from PC0 to 172.16.0.19 | ✅ 5-hop path matches design exactly |

## Path Analysis (tracert)

```
Tracing route to 172.16.0.19 over a maximum of 30 hops:

  1    172.16.0.1     (HQ-R)
  2    10.0.0.2        (Core-R)
  3    10.0.0.6        (Branch01-R)
  4    10.0.0.10       (Branch02-R)
  5    172.16.0.19     (PC5 - destination)
```

This confirms the deliberate multi-hop chain design (HQ-R → Core-R → Branch01-R → Branch02-R) is functioning exactly as architected.

## Notes on First-Packet Timeout

The first ICMP packet in a new conversation between two devices commonly times out due to ARP resolution — the sending device must first discover the destination's (or gateway's) MAC address via an ARP request/reply exchange before it can transmit. This is a Layer 2 mechanism unrelated to routing correctness, and subsequent packets succeed immediately once the ARP cache is populated.

## TTL Verification

Round-trip TTL values were used to independently confirm hop count:
- HQ-Staff LAN (same segment): TTL 128 (0 hops)
- Server Farm (1 hop): TTL 127
- Branch02-LAN (4 hops through HQ-R, Core-R, Branch01-R, Branch02-R): TTL 124 (128 − 4 = 124)

This matches the tracert output and confirms the topology's hop count independently.
