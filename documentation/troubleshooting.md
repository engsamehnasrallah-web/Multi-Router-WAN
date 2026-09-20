# Troubleshooting Lab

Deliberate failure-injection scenarios were used to validate the network's resilience and to practice systematic fault isolation, rather than relying only on "it works" as proof of a correct design.

## Scenario 1 — Missing Static Route on Core-R

### Symptom

Ping from PC0 (HQ-Staff LAN) to PC5 (Branch02-LAN, 172.16.0.19) failed completely:

```
Reply from 10.0.0.2: Destination host unreachable.
(×4, 100% loss)
```

### Investigation

1. The unreachable reply originated from `10.0.0.2` (Core-R's interface toward HQ-R) — meaning the path from PC0 to Core-R was fully functional; Core-R itself was rejecting the request.
2. `show ip route` on Core-R was compared against the known-good routing table from Phase 5. The comparison showed:
   ```
   172.16.0.0/16 is variably subnetted, 3 subnets, 2 masks   ← should be 4 subnets
   S 172.16.0.0/29 [1/0] via 10.0.0.1
   S 172.16.0.8/29 [1/0] via 10.0.0.6
   S 172.16.1.0/30 [1/0] via 10.0.0.1
   ```
   The route to `172.16.0.16/29` (Branch02-LAN) was missing entirely.

### Root Cause

The static route on Core-R pointing to Branch02-LAN via Branch01-R (`10.0.0.6`) had been removed.

### Why Core-R Specifically Reported the Failure

Core-R is the router responsible for forwarding traffic toward the `172.16.0.16/29` range. Since it had no route (static or connected) for that destination, it could not forward the packet any further and generated the "Destination host unreachable" response itself — even though the network segment further down the chain (Branch01-R, Branch02-R) was completely healthy.

Had the same route been missing on Branch01-R instead, the symptom would have looked similar but originated from a different router (`10.0.0.10`, Branch01-R's interface toward Branch02-R) — since in that case Core-R would have forwarded the packet correctly, but Branch01-R would be the one unable to continue routing it.

### Fix

```
ip route 172.16.0.16 255.255.255.248 10.0.0.6
```

### Verification

```
Reply from 172.16.0.19: bytes=32 time=71ms TTL=124
(×4, 0% loss)
```

Full connectivity restored, matching the original Phase 5 baseline.

---

*Additional failure scenarios (interface shutdown, wrong IP, wrong subnet mask, wrong next-hop, broken WAN link) are planned for a future documentation update.*
