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

## Scenario 2 — Shut Down WAN Interface on Core-R

### Symptom

Ping from PC0 to PC5 (172.16.0.19) failed with the exact same message and source as Scenario 1:

```
Reply from 10.0.0.2: Destination host unreachable.
(×4, 100% loss)
```

### Investigation

Because the symptom looked identical to Scenario 1, the fix from Scenario 1 was verified first — `show ip route` on Core-R confirmed the static route to `172.16.0.16/29` was still present, pointing to `10.0.0.6`. This meant the cause was different this time, despite the identical-looking error.

`show ip interface brief` on Core-R showed `Serial0/0/1` — the interface toward Branch01-R (Link2) — in an `administratively down / down` state.

### Root Cause

`Serial0/0/1` had been deliberately shut down. Since the route to `172.16.0.16/29` depends on reaching next-hop `10.0.0.6` (part of the `10.0.0.4/30` network on that same interface), the route could not be used once the interface was down.

### Fix

```
interface Serial0/0/1
no shutdown
```

### Verification

Ping succeeded immediately after re-enabling the interface, 0% loss.

### Note — Simulator Behavior vs. Real IOS

On real Cisco hardware, a static route depending on a next-hop reached through a now-down interface would normally disappear from `show ip route` entirely (a recursive-lookup failure), since the next-hop becomes unresolvable. In this Packet Tracer lab, the static route remained visible in the routing table throughout the outage, even though it was non-functional. This is a known simplification in Packet Tracer's routing simulation, not real IOS behavior — worth keeping in mind when comparing lab results to production Cisco devices. See [`documentation/lessons-learned.md`](lessons-learned.md) for further discussion.

### Key Distinction from Scenario 1

Both scenarios produced the identical error message from the identical source IP (`10.0.0.2`), despite having completely different root causes (a missing route vs. an unreachable next-hop). This is an important troubleshooting lesson: an identical symptom does not guarantee an identical cause, and each incident needs to be independently verified rather than assumed to match a previous one.

---

## Scenario 3 — Wrong IP Address on End Device (PC2)

### Symptom

Ping from PC0 to PC2 (172.16.0.10, Branch01-LAN) failed differently from Scenarios 1 and 2 — no rejection message at all, just silence:

```
Request timed out.
(×4, 100% loss)
```

This immediately distinguishes it from a routing-table problem: "Destination host unreachable" means a router explicitly knows it has no path and says so; "Request timed out" means nothing replied at all, positive or negative.

### Investigation

`tracert 172.16.0.10` from PC0 showed the path succeeding through all three routers on the way (HQ-R, Core-R, Branch01-R at `10.0.0.6`), then failing to get any response beyond that point:

```
1  172.16.0.1   (HQ-R)
2  10.0.0.2     (Core-R)
3  10.0.0.6     (Branch01-R)
4  *  *  *  Request timed out
```

Since the path reached Branch01-R successfully, the problem had to be at or beyond that last hop — either the router's own LAN interface, or the end device itself. `show ip interface brief` on Branch01-R confirmed `FastEthernet0/1` still held the correct address (`172.16.0.9`), ruling out the router. That left PC2 itself.

### Root Cause

PC2's IP address had been misconfigured to `172.16.1.10` — a valid-looking address, but from an entirely different subnet (`172.16.1.0/30`, the HQ-Server-Farm range) rather than its own `172.16.0.8/29` (Branch01-LAN).

### Fix

Corrected on PC2:
```
IP Address:      172.16.0.10
Subnet Mask:     255.255.255.248
Default Gateway: 172.16.0.9
```

### Verification

```
Reply from 172.16.0.10: bytes=32 time=51ms TTL=125
(×4, 0% loss)
```

`ipconfig /all` confirmed the corrected IP, mask, and gateway.

### Key Distinction from Scenarios 1 & 2

A missing/broken route (Scenarios 1–2) produces an explicit rejection from the router that detects it. A misconfigured end-device IP (Scenario 3) produces silent timeouts instead, because the packet is routed correctly all the way to the destination's segment — the device simply isn't reachable at the address being pinged. `tracert` was the key diagnostic here: it isolates exactly which hop is the last to respond, immediately narrowing the search to "at or past this point" rather than requiring a router-by-router routing-table comparison.

---

## Scenario 4 — Mismatched Subnet Mask on a WAN Link

### Symptom

`Serial0/0` on Branch02-R was changed from `/30` (255.255.255.252) to `/29` (255.255.255.248) — widening only one side of Link3 (Branch01-R ↔ Branch02-R). The first ping packet timed out (expected ARP-resolution delay), and the remaining three succeeded.

### Why "It Still Worked" Is Misleading

A wider mask on only one end of a point-to-point link is a genuine misconfiguration, not a valid optimization — it does not achieve route summarization (which is a Layer 3 routing-table technique applied elsewhere, not a mismatched interface mask). With no other device occupying the extra address range the wider mask now includes, the mismatch produced no visible symptom in this small lab — but it leaves the two routers disagreeing about what "the same network" even means on that link. A third device landing in the extra range would be treated as a local neighbor by Branch02-R but as an external, unreachable address by Branch01-R, on the same physical link — a subtle and hard-to-diagnose failure mode in a larger network.

### Fix

```
interface Serial0/0
ip address 10.0.0.9 255.255.255.252
```

Mask restored to `/30` to match Branch01-R's side of the same link.

### Verification

`show ip interface brief` on Branch02-R confirmed `Serial0/0` back at `255.255.255.252`, and end-to-end ping succeeded with 0% loss.

### Key Lesson

Both endpoints of a point-to-point WAN link must agree on the subnet mask. A mismatch may not produce any visible symptom in a small or lightly-populated network — it becomes a real problem only once a device happens to fall in the range the two sides disagree about — which makes it a genuinely dangerous class of misconfiguration to leave unverified.

---

## Scenario 5 — Wrong Next-Hop on a Static Route

### Symptom

An incorrect static route was added on Branch02-R for HQ-Staff LAN, pointing to an unreachable next-hop:

```
ip route 172.16.0.0 255.255.255.248 10.0.0.5
```

`10.0.0.5` is Core-R's interface address, not reachable from Branch02-R (which only connects directly to Branch01-R). The correct route (`via 10.0.0.9`) was also removed at this point, leaving only the broken one.

Ping from PC0 (HQ) to PC5 (Branch02-LAN) failed completely:
```
Request timed out.
(×4, 100% loss)
```

### Investigation

This scenario is a **one-way failure**, not a full connectivity loss — an important distinction from the others:

- The **forward** path (PC0 → PC5) was entirely unaffected. No router along the way (HQ-R, Core-R, Branch01-R, Branch02-R) needs a route to `172.16.0.0/29` to deliver a packet *to* Branch02-LAN — only the reverse trip does.
- The **return** path (PC5's replies → PC0) failed: once Branch02-R received the ICMP echo request and needed to send the reply back toward `172.16.0.0/29`, its only route pointed to an unreachable next-hop (`10.0.0.5`, not directly connected to Branch02-R).

Because `ping` reports failure whenever no reply is received — regardless of whether the *request* successfully arrived — this one-way break still produced a 100% loss result identical in appearance to a full outage.

`show ip route 172.16.0.0` on Branch02-R confirmed only the broken route was present, pointing to the unreachable `10.0.0.5`.

### Fix

```
no ip route 172.16.0.0 255.255.255.248 10.0.0.5
ip route 172.16.0.0 255.255.255.248 10.0.0.9
```

The old route was explicitly removed before adding the corrected one (rather than assuming the new command would overwrite it), since Cisco IOS allows multiple static routes to the same destination network to coexist.

### Verification

```
Reply from 172.16.0.19: bytes=32 time=67ms TTL=124
(×4, 0% loss)
```

### Key Lesson

A next-hop pointing to a real, existing IP address — but one this router cannot actually reach — is a more subtle failure than a missing route entirely (Scenario 1). It highlights that static routing failures aren't always symmetric: traffic can flow correctly in one direction while failing in the other, and a single failed ping doesn't by itself reveal *which* direction is actually broken. Reasoning about forward vs. return path separately was the key step in isolating the cause here.

---

All five planned failure-injection scenarios are now complete, covering: a missing route, a shut-down interface, a misconfigured end-device IP, a mismatched WAN subnet mask, and an unreachable next-hop.
