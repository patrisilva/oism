# Arista EVPN Mulicast (OISM) Demo

## QuickStart

From the terminal:

- Start the lab: `make start`
- Stop the lab: `make stop`
- Inspect the lab: `make inspect`

> [!TIP]
> For the best experience, the [VS Code ContainerLab Extension](https://containerlab.dev/manual/vsc-extension/) is Highly Recommended

## Overview

- 21 EOS Nodes
- 15 Ubuntu Nodes
- Built and validated with cEOS-Lab 4.35.1F

## Requirements

- [ContainerLab](https://containerlab.dev/install/) Host
- [cEOS-Lab image](https://www.arista.com/en/support/software-download)
- cEOS-Lab image imported into Docker as `ceos:latest`
- 48G RAM

## Topology

![Topology](./assets/images/topo.png)

## Description
This topology is a multi-domain multicast design that combines EVPN, PIM, and external multicast domains. I’ll walk through it layer by layer and explain how multicast actually works end-to-end in this diagram.

⸻

1️⃣ Big picture (what this design is doing)

You have three multicast domains interconnected:
	1.	External PIM Domain (VRF PROD)
	•	Traditional PIM Sparse Mode
	•	Central RP(s)
	•	Multicast sources and receivers live here
	2.	EVPN Domain A (ASN 65001)
	•	VXLAN EVPN fabric
	•	Multicast is not flooded; it is control-plane driven
	•	Leaf switches act as multicast gateways (PEG / GW)
	3.	EVPN Domain B (ASN 65002)
	•	Another independent EVPN fabric
	•	Same multicast gateway concept
	•	Inter-domain communication via CORE (ASN 65000)

👉 Key idea:
Multicast never traverses EVPN as raw PIM.
It is translated at the fabric edges.

⸻

2️⃣ Multicast inside EVPN fabrics (Domains A & B)

🔹 Inside each EVPN domain:
	•	Hosts join multicast groups (e.g. 239.0.10.101, 239.0.40.101)
	•	IGMP snooping happens on access VLANs
	•	EVPN Type-6 routes advertise multicast membership
	•	Traffic is carried over VXLAN using ingress replication or P2MP

What EVPN does instead of PIM:
Traditional         EVPN
PIM Join            EVPN Type-6
(*,G) / (S,G)       MAC/IP + IMET
Flooding            Selective replication

🚫 No PIM runs inside the fabric

3️⃣ Multicast Gateways (PEG / GW nodes)

These are the most important devices in the diagram.

Examples:
	•	A-LEAF-7, A-LEAF-8
	•	B-LEAF-7, B-LEAF-8

They perform multicast domain translation:

Northbound (toward PIM):
	•	Act as PIM routers
	•	Send IGMP / PIM Joins upstream
	•	Participate in RP-based multicast trees

Southbound (toward EVPN):
	•	Act as EVPN multicast gateways
	•	Convert EVPN Type-6 ↔ PIM joins
	•	Replicate traffic into VXLAN

👉 These nodes are often called:
	•	PIM-EVPN Gateways
	•	PEGs (PIM Edge Gateways)

⸻

4️⃣ External PIM Domain (VRF PROD)

What we see:
	•	RP (PD1-R3)
	•	PIM routers (PD1-R1, PD1-R2)
	•	External hosts (PD1-H1, PD1-H2, PD1-H3)
	•	Classic Sparse Mode behavior

Multicast flow example:
	1.	PD1-H3 sources 239.201.30.101
	2.	PIM builds (*,G) and (S,G) trees
	3.	PEGs join the tree only if EVPN has interested receivers
	4.	Traffic is forwarded only where needed

⸻

5️⃣ Inter-domain multicast via CORE (ASN 65000)

The CORE connects:
	•	EVPN Domain A
	•	EVPN Domain B
	•	External PIM Domain

Key points:
	•	CORE is L3 only
	•	Multicast is routed using PIM or MVPN
	•	Each EVPN domain remains isolated (different ASNs)
	•	Multicast is policy-controlled at domain borders

This avoids:
	•	Multicast flooding between fabrics
	•	RP sprawl
	•	Cross-domain IGMP chaos

⸻

6️⃣ Multicast source / receiver patterns in the diagram

EVPN-only multicast

Example:
	•	HostA1 → HostA4
	•	Source and receivers inside Domain A
	•	Never leaves EVPN
	•	VXLAN replication only

External → EVPN multicast

Example:
	•	PD1-H3 → HostA5
	•	External PIM source
	•	PEG imports traffic
	•	EVPN delivers to correct VLAN/VNI

EVPN → External multicast

Example:
	•	HostB3 → PD1-H2
	•	EVPN host is the source
	•	PEG registers with RP
	•	PIM delivers to external receivers

⸻

7️⃣ Anycast Gateway & Multicast

You are using:
	•	Anycast GW IPs on leaves
	•	Distributed IGMP processing
	•	Consistent RP reachability

This ensures:
	•	IGMP joins hit the closest leaf
	•	No single multicast choke point
	•	Symmetric multicast paths

⸻

8️⃣ Why this design is “correct” (and scalable)

✅ No PIM inside the fabric
✅ No multicast flooding
✅ Clear domain boundaries
✅ RP stays centralized
✅ EVPN handles scale
✅ PIM handles inter-domain routing

This is exactly how modern EVPN multicast is designed in large networks.

⸻

9️⃣ In one sentence

Multicast in this topology is EVPN-native inside each fabric, PIM-native outside the fabric, and seamlessly translated at the fabric edges using multicast gateways (PEGs).