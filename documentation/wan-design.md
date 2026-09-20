# WAN Design

## Topology

```
[HQ-Staff LAN] ── HQ-R ──(Link1)── Core-R ──(Link2)── Branch01-R ──(Link3)── Branch02-R ── [Branch02-LAN]
                    │                                      │
              [HQ-Server-Farm]                      [Branch01-LAN]
```

Core-R is a dedicated transit router (no local LAN). Branch01-R plays a dual role: it serves its own LAN and also transits traffic for Branch02-R, which has no direct link to Core-R.

## Links, Subnets, and DCE/DTE

| Link | Between | Subnet | DCE (clock rate) | DTE |
|---|---|---|---|---|
| Link1 | HQ-R ↔ Core-R | 10.0.0.0/30 | Core-R | HQ-R |
| Link2 | Core-R ↔ Branch01-R | 10.0.0.4/30 | Core-R | Branch01-R |
| Link3 | Branch01-R ↔ Branch02-R | 10.0.0.8/30 | Branch01-R | Branch02-R |

## DCE/DTE Reasoning

There is no external ISP in this scenario — all four routers belong to the same company (NovaTech), so no third party dictates clocking. Instead, the DCE role was assigned to whichever router sits closer to the network's backbone (Core-R) on each link:

- **Link1 and Link2** touch Core-R directly, so Core-R holds the DCE role on both.
- **Link3** does not touch Core-R at all (it connects Branch01-R and Branch02-R). Applying the same "closer to core" principle one level down the chain, **Branch01-R** — being closer to Core-R in the topology — holds the DCE role there instead.

All DCE interfaces are configured with `clock rate 64000`.

## Implementation Note

Cisco Packet Tracer determines the physical DCE/DTE role based on the direction a serial cable is drawn (the end you start dragging from typically becomes DCE), not by IOS command alone. During implementation, an initially-drawn cable between HQ-R and Core-R resulted in the reverse of the intended DCE assignment; it was corrected by deleting and redrawing the cable starting from Core-R. `show controllers <interface>` was used to verify each link's actual DCE/DTE role before proceeding.
