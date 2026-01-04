# oism
Arista OISM (Optimized Inter-Subnet Multicast) is an advanced technique within Arista's EVPN ecosystem that efficiently handles multicast traffic across different subnets in a data center, using standard EVPN routes (Type-6) to create a distributed multicast forwarding plane, avoiding bottlenecks by making every switch a potential first-hop router for local receivers, thus improving scalability and performance for streaming/broadcast applications.  
Key Concepts of OISM:  

-  Distributed Designated Router (DR): Instead of a single central router, OISM makes every switch (VTEP) in the EVPN fabric a multicast DR for its attached subnets, streamlining traffic.  
- Supplementary Bridge Domain (SBD): A shared bridge domain connects all leaf switches (VTEPs) in a VRF, allowing multicast traffic to be efficiently bridged (VXLAN bridged) across the fabric.  
- "Type-6" Signaling: VTEPs signal interest in multicast flows using Type-6 advertisements (SMET routes) to coordinate this distributed forwarding.  
- Optimized Delivery: Traffic is routed at the first point of interest, reducing latency and eliminating the need to send traffic to a central point for inter-subnet forwarding.   

Why it's Important:  
- Scalability: Handles large-scale multicast (like video, financial data) in modern, distributed data centers.
- Standardization: Leverages existing EVPN capabilities for a vendor-neutral, standards-based approach.
- Integration: Bridges data center fabrics with traditional IP multicast networks.  

In essence, OISM brings optimized, scalable, and standards-compliant multicast routing to EVPN networks, making Arista a leader in data center multicast solutions. 
# oism-demo-v1
check the README under the folder.

# oism-demo-v2
check the README under the folder.
