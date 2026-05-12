# 🏗️ High-Availability Enterprise Network: VRRP + MSTP Load Balancing
### *My Third Project as a Network Engineer*

> Redundancy isn't an accident—it's architecture.

---

## 🗺️ Topology

![Network Topology](https://YOUR-IMAGE-HOST.com/topology.png)

**Core Layer:** 2x MikroTik CCR2116 (L3)  
**Access Layer:** 3x MikroTik CRS Series (L2)  
**Redundancy:** Dual ISP Failover + VRRP + MSTP  
**Segmentation:** VLAN 10 (IT) & VLAN 20 (HR) via VLSM

**The Goal:** Build a network where no single cable or router failure can bring the office down, while ensuring both core routers share the traffic load.

---

## 🎭 The Lore

Okay so here's the scenario:

Most junior admins set up a "main" router and a "backup" that just sits there gathering dust. That's a waste of hardware.

For this project, I didn't just want a backup. I wanted **active-active efficiency**.

I needed IT department using Core 1 while HR used Core 2. But if either router died, the other would take over instantly without users even noticing.

So I combined:
- **VRRP** (Layer 3) for gateway redundancy
- **MSTP** (Layer 2) for loop prevention and path steering

Two protocols. One goal. No idle links.

---

## 🛠️ The Implementation: Layer 3 (VRRP)

I used a split-scope DHCP and VRRP strategy. Each VLAN has a virtual gateway, but the "Master" status is split between the two cores.

**The Logic:**

| VLAN | Core 1 Role | Core 1 IP | Core 2 Role | Core 2 IP |
|------|-------------|-----------|-------------|-----------|
| VLAN 10 (IT) | Master | .61 | Backup | .62 |
| VLAN 20 (HR) | Backup | .29 | Master | .30 |

**DHCP Split:** Each router handles half the IP pool. If one fails, the other still has enough addresses to keep things running.

---

## ⚡ The Implementation: Layer 2 (MSTP)

Standard STP is boring—it just shuts down links. MSTP is where the real engineering happens.

I created two MSTP instances (MSTI):

| Instance | VLAN | Root Bridge |
|----------|------|-------------|
| MSTI 1 | VLAN 10 (IT) | Core 1 |
| MSTI 2 | VLAN 20 (HR) | Core 2 |

**The Result:** IT traffic travels up the left wire. HR traffic travels up the right wire. We're using **100% of the copper we paid for**.

---

## 🧪 The Proof: Validation Tests

I ran four critical tests to prove this network is both resilient and optimized.

---

### Test 1: VRRP Redundancy (The "Failover" Test)

**What I did:** Manually disabled the SFP+ uplink on Core 1 while running a continuous ping from an IT workstation.

**What happened:** Core 2 transitioned from Backup to Master in under 1 second. Only one packet was lost.

![VRRP Failover Test](https://YOUR-IMAGE-HOST.com/vrrp-failover.gif)

---

### Test 2: DHCP Split-Scope Verification

**What I did:** Connected a new client to VLAN 10 and checked the assigned IP. Then disabled Core 1's DHCP server and renewed the lease.

**What happened:** Client first received IP from Core 1's pool (.2-.31), then successfully failed over to Core 2's pool (.32-.60).

![DHCP Split Test](https://YOUR-IMAGE-HOST.com/dhcp-split.png)

---

### Test 3: MSTP Root Bridge & Port Steering

**What I did:** Monitored MSTIs on Access-SW-1 to verify which ports were blocking.

**What happened:**
- **MSTI 1 (VLAN 10):** Port 9 (to Core 1) = Forwarding. Port 8 (to Core 2) = Discarding.
- **MSTI 2 (VLAN 20):** Port 9 = Discarding. Port 8 = Forwarding.

Traffic is perfectly split. Both links active.

![MSTP Port Steering](https://YOUR-IMAGE-HOST.com/mstp-ports.gif)

---

### Test 4: Inter-VLAN Routing & Firewall Isolation

**What I did:** Attempted a ping from a VLAN 10 host to a VLAN 20 host.

**What happened:** Traffic was dropped by the Firewall Filter Rule. IT and HR stay separate.

![Firewall Isolation](https://YOUR-IMAGE-HOST.com/firewall-drop.png)

---

## 📊 Summary Table

| Feature | Protocol | Benefit |
|---------|----------|---------|
| Gateway Redundancy | VRRP | No manual IP changes if a router dies |
| Loop Prevention | MSTP | Sub-second convergence, both links utilized |
| IP Efficiency | VLSM | Maximized address space for 2,800+ hosts |
| Security | Firewall Filters | Strict inter-department isolation at the Core |

---

## 🚧 Challenges I Faced

**1. Syncing VRRP and MSTP**

If Core 1 is the VRRP Master but MSTP makes the path to Core 2 the only open link, your traffic "hairpins" across the core-to-core link.

**Solution:** I carefully aligned the Bridge Priorities with the VRRP Priorities to keep paths optimal.

**2. Bridge VLAN Filtering on MikroTik**

I learned that enabling `vlan-filtering=yes` is the "point of no return." If your tagged/untagged assignments aren't perfect before you toggle that switch, you'll lose management access immediately.

**Solution:** Double-checked every port assignment. Still stressful.

---

## 💡 Final Thoughts

Building a network that "just works" is easy. Building a network that is **redundant, optimized, and secure** requires a deep dive into the protocols.

By implementing MSTP alongside VRRP, I created a system that isn't just "High Availability"—it's **high performance**.

**The Metric That Matters: Link Utilization**

I measured which ports were forwarding vs discarding on each MSTI. Standard STP would have left one link completely idle. MSTP gives me 100% utilization.

**Why This Matters to an Employer:**

Most engineers set up VRRP for redundancy. That's table stakes.

I went further. I aligned Layer 2 path selection with Layer 3 active gateways so both cores are always doing work. No idle hardware. No wasted bandwidth.

If I'm managing your infrastructure, your "redundant" links won't be sitting there gathering dust. They'll be working for you.

---

*Third project down. More to come.* 🔥
