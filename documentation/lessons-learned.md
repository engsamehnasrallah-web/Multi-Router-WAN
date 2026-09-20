# Lessons Learned

Reflections from building this project, focused on concepts that only became clear through implementation and troubleshooting — not just design on paper.

## VLSM vs Flat Subnetting

Designing every subnet at its actual required size (instead of a uniform `/24` like Project 01) forced a much closer engagement with binary block sizes and address-space efficiency. It also surfaced a subtle mistake early on: initially sizing the Server Farm subnet for "1 host" without accounting for the router's own gateway interface — a reminder that every subnet with a router attached needs at least one extra usable address for the gateway itself.

## DCE/DTE Is a Cabling Decision, Not Just a Command

The most instructive mistake in this project was discovering that Cisco Packet Tracer assigns the DCE/DTE role based on the direction a serial cable is drawn — not purely by which side the `clock rate` command is issued on. A cable drawn "the wrong way" silently reversed the intended DCE role on Link1, and the mismatch only became visible through `show controllers`, not `show ip route` or `show ip interface brief`. This reinforced that Layer 1 assumptions should always be verified explicitly, not inferred from a device seeming to work.

## A Router Only Knows What's in Its Own Routing Table

Early static-routing design included a flawed assumption that Core-R wouldn't need its own routes for far-side networks (like Branch02-LAN) since "it's just in the middle and traffic will pass through it anyway." This is incorrect: every router along a path — including transit-only routers — needs an explicit route (or a directly connected interface) for every destination it's expected to forward toward. This same misunderstanding was deliberately reintroduced and diagnosed in the Troubleshooting Lab (Scenario 1), which made the concept concrete rather than abstract.

## First-Packet Timeouts Are Usually ARP, Not a Design Flaw

Seeing a single dropped packet on the very first ping to a new destination — followed by 100% success afterward — initially looked like a connectivity problem. It's actually the expected cost of ARP resolution (learning a destination or gateway's MAC address) before the first packet can be transmitted, and it's unrelated to routing correctness. Confirming this using TTL values (which independently validated hop count) was a useful way to separate "transient Layer 2 behavior" from "a real Layer 3 problem."

## Root-Cause Reasoning Beats Symptom-Chasing

When Scenario 1's ping failed with "Destination host unreachable" from a specific router's IP, the instinct might be to check the destination end of the network first. Instead, tracing the message back to *which device generated the rejection* (and reasoning about why that specific device — not any other — would be the one to reject it) pointed directly at the correct router to investigate, without needing to check every device in the topology.
