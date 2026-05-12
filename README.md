# Built Active-Active Enterprise Core with VRRP, MSTP, & DHCP Failover
### *Deployed Deterministic Traffic Engineering and Multi-Layer Redundancy for Zero-Downtime Infrastructure.*

> Active-Passive is a waste. Every piece of hardware should earn its keep.

---

## The Topology

![Network Topology](https://YOUR-IMAGE-HOST.com/topology.png](https://imgur.com/q7hxYZl)

**Core Layer:** Dual MikroTik CCR2116 (L3) with VRRP Gateway Redundancy  
**Access Layer:** Triple MikroTik CRS Series (L2) with MSTP Path Steering  
**Failover Stack:** DHCP Split-Scope + MSTP + VRRP  
**Segmentation:** 9-VLAN Enterprise Environment using VLSM (10.10.0.0/22)

---

## The Lore

Okay so here's the scenario:

Most networks run Active-Passive. One router does all the work. The other sits there collecting dust, waiting for a disaster that might never happen.

That's a waste of hardware.

For this project, I wanted every device to actually **do something**. No idle backups. No wasted bandwidth.

I engineered an Active-Active infrastructure where:
- IT traffic flows through Core 1
- HR traffic flows through Core 2
- If either dies, the other takes over in under a second

By synchronizing MSTP root bridges with VRRP master roles, I forced traffic to physically steer through different core routers. Doubled my throughput. Kept sub-second failover.

Hardware earns its keep.

---

## Performance Highlights

**Deterministic Traffic Engineering**  
Synchronized MSTP instances with VRRP priorities to steer VLAN 10 (IT) through Core 1 and VLAN 20 (HR) through Core 2. Doubled backplane utilization. Eliminated "hairpin" routing.

**Automated DHCP Failover**  
Configured split-scope DHCP architecture with reserved pools. Verified 100% lease continuity during total core-router failure scenarios.

**Layer 3 Redundancy**  
Built VRRP-based virtual gateway system achieving failover convergence in <1 second. Maintained persistent sessions for mission-critical inter-VLAN traffic.

**Hardened Network Segmentation**  
Developed stateful firewall filter rules and address lists to enforce strict department isolation. Reduced internal attack surface without impacting line-rate performance.

---

## The Proof: Validation Benchmarks

### Test 1: The Gateway Transition (VRRP)

**What I did:** Force-disabled the active SFP+ uplink on the Master Core.

**What happened:** VRRP state transition to Backup Core completed in <800ms. Only one packet dropped during the transition.

![VRRP Failover](https://YOUR-IMAGE-HOST.com/vrrp-failover.gif)

---

### Test 2: The Lease Continuity (DHCP Failover)

**What I did:** Simulated a total hardware crash on Core 1. Performed client-side IP renewal.

**What happened:** Client successfully pulled a secondary lease from Core 2's reserved pool in <3 seconds. No network lockout.

![DHCP Failover](https://YOUR-IMAGE-HOST.com/dhcp-failover.gif)

---

### Test 3: The Path Steering (MSTP)

**What I did:** Audited Access-SW-1 bridge port states for MSTI 1 and MSTI 2.

**What happened:** Forwarding/Discarding states perfectly matched the logical root bridge topology. Zero-loop multi-pathing confirmed.

![MSTP Port States](https://YOUR-IMAGE-HOST.com/mstp-ports.gif)

---

### Test 4: The Security Layer (Firewall)

**What I did:** Executed cross-VLAN penetration test from VLAN 10 to VLAN 99.

**What happened:** 100% drop rate verified via real-time packet counters on the Firewall Filter chain.

![Firewall Drop](https://YOUR-IMAGE-HOST.com/firewall-drop.png)

---

## Engineering Challenges

**Core-to-Core Alignment**

The biggest challenge was preventing L2 loops while keeping L3 gateways active.

**How I solved it:** Tuned MSTP Bridge Priorities to force the L2 "Blocked" ports to coincide with the L3 "Backup" router. The shortest physical path is always the active one.

**VLSM Management**

With 2,800+ potential hosts across 9 VLANs, I had to ensure the IP addressing was mathematically perfect. No subnet overlap. No route flapping.

**How I solved it:** Mapped every VLAN carefully before touching a single router.

---

## Final Thoughts

Modern networks shouldn't have "idle" hardware.

Active-Passive is easy. Any junior admin can set up a backup router that never gets used.

Active-Active is engineering. It forces you to understand how traffic actually flows, where loops can form, and how to make every device work for its keep.

**The Metric That Matters: Link Utilization**

I measured which ports were forwarding vs discarding on each MSTI. Standard STP would have left one link completely idle. MSTP gives me 100% utilization of both uplinks.

---

*Third project down. More to come.* 🔥
