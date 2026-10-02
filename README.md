# Enterprise Multi-Protocol Routing Lab

🔬 **Project 4 — CCNP Level | Advanced Routing**

Implementation of advanced routing in an Enterprise topology including OSPF Multi-Area, EIGRP, Route Redistribution, and Route Filtering.

---

## 📋 Overview

This project is a comprehensive CCNP-level routing lab where the following concepts are implemented hands-on:

- **OSPF Multi-Area** with Route Summarization
- **EIGRP** with DUAL and Variance (Load Balancing)
- **Route Redistribution** between OSPF and EIGRP
- **Route Filtering** with Prefix-list and Route-map

---

## 🏗️ Topology

![Topology](topology/topology.png)

### Topology Components:
- **6 Cisco 7200 Routers:** R1-Core, R2, R3, R-ASBR, E1, E2
- **2 vios_l2 Switches:** SW-1, SW-2
- **2 VPCS Clients:** PC1, PC2

---

## 📊 IP Addressing

| Device | Interface | IP Address | Description |
|--------|-----------|------------|-------------|
| R1-Core | Gi0/0 | 10.0.0.1/30 | Link to R2 |
| R1-Core | Gi1/0 | 10.0.0.5/30 | Link to R3 |
| R1-Core | Gi2/0 | 10.0.0.9/30 | Link to R-ASBR |
| R1-Core | Lo0 | 1.1.1.1/32 | Router-ID |
| R2 | Gi0/0 | 10.0.0.2/30 | Link to R1-Core |
| R2 | Gi1/0 | 192.168.100.1/24 | LAN (PC1 Gateway) |
| R2 | Lo0 | 2.2.2.2/32 | Router-ID |
| R2 | Lo10 | 172.16.1.1/24 | Test Summarization |
| R2 | Lo11 | 172.16.2.1/24 | Test Summarization |
| R3 | Gi0/0 | 10.0.0.6/30 | Link to R1-Core |
| R3 | Lo0 | 3.3.3.3/32 | Router-ID |
| R3 | Lo10 | 172.17.1.1/24 | Test Summarization |
| R3 | Lo11 | 172.17.2.1/24 | Test Summarization |
| R-ASBR | Gi0/0 | 10.0.0.10/30 | Link to R1-Core |
| R-ASBR | Gi1/0 | 10.0.0.13/30 | Link to E1 |
| R-ASBR | Gi2/0 | 10.0.0.17/30 | Link to E2 |
| R-ASBR | Lo0 | 4.4.4.4/32 | Router-ID |
| E1 | Gi0/0 | 10.0.0.14/30 | Link to R-ASBR |
| E1 | Gi1/0 | 192.168.200.1/24 | LAN (PC2 Gateway) |
| E1 | Lo0 | 5.5.5.5/32 | Router-ID |
| E2 | Gi0/0 | 10.0.0.18/30 | Link to R-ASBR |
| E2 | Lo0 | 6.6.6.6/32 | Router-ID |
| PC1 | e0 | 192.168.100.10/24 | Gateway: 192.168.100.1 |
| PC2 | e0 | 192.168.200.10/24 | Gateway: 192.168.200.1 |

---

## 🎯 Design Overview

### OSPF Areas
| Router | Role | Areas |
|--------|------|-------|
| **R1-Core** | **ABR** | 0, 1, 2 |
| **R2** | Internal Router | 1 |
| **R3** | Internal Router | 2 |
| **R-ASBR** | Internal Router | 0 |

### EIGRP Domain
- **E1, E2** — EIGRP AS 10
- **R-ASBR** — Border point between OSPF and EIGRP (ASBR)

---

## ✅ Progress

- [x] **Phase 1: OSPF Multi-Area + Summarization**
- [x] Phase 2: EIGRP + DUAL + Variance
- [ ] Phase 3: Route Redistribution (OSPF ↔ EIGRP)
- [ ] Phase 4: Route Filtering (Prefix-list + Route-map)

---

## 📊 Phase 1: OSPF Multi-Area

### 🔧 Configuration Highlights

- Enabled OSPF Process 1 on all routers with manually configured Router-IDs
- **R1-Core** acts as ABR between three areas (0, 1, 2)
- **Route Summarization** with /22 on R1-Core:
  ```
  area 1 range 172.16.0.0 255.255.252.0
  area 2 range 172.17.0.0 255.255.252.0
  ```
- Added Loopback10 and Loopback11 on R2 and R3 for Summarization testing

### ✅ Verification Results

- ✅ **3 OSPF neighbors FULL** on R1-Core (R2, R3, R-ASBR)
- ✅ **1 OSPF neighbor FULL** on every other router
- ✅ Intra-area and Inter-area route lists working correctly
- ✅ Summary routes (`172.16.0.0/22`, `172.17.0.0/22`) visible in LSDB
- ✅ End-to-end ping tests — **100% success rate**

### 🐛 Issues Encountered & Solutions

**Issue #1: Loopback0 not advertised in OSPF**

- **Problem:** Pings with `source loopback 0` were failing between routers
- **Root Cause:** Loopback0 was not advertised via the `network` command
- **Solution:** Added `network X.X.X.X 0.0.0.0 area Y` on R2, R3, and R-ASBR
- **Lesson Learned:** In OSPF, every interface must be **explicitly** advertised using the `network` command — unlike EIGRP which uses class-based advertisement.

---


## 📊 Phase 2: EIGRP + DUAL

### 🔧 Configuration Highlights

- Enabled EIGRP AS 10 on R-ASBR, E1, E2
- Added secondary link between E1 and E2 (`10.0.1.0/30`)
- Disabled `auto-summary` on all EIGRP routers
- Advertised Loopbacks and LAN subnets (`192.168.200.0/24`)
- Configured `passive-interface` on Loopback and LAN-facing interfaces

### ✅ Verification Results

- ✅ 2 EIGRP neighbors on each router (R-ASBR, E1, E2)
- ✅ DUAL Topology Table analyzed with `all-links`
- ✅ ECMP active on `10.0.0.16/30` and `10.0.0.12/30`
- ✅ `192.168.200.0/24` propagated from E1 to E2

### 🐛 Issues Encountered & Solutions

**Issue #1: Wrong IP on E2's Gi1/0**
- **Problem:** Pings between E1 and E2 took the indirect path via R-ASBR
- **Root Cause:** IP `10.0.0.2/30` mistakenly configured on E2's Gi1/0 (conflict with R2)
- **Solution:** Corrected to `10.0.1.2/30`
- **Lesson Learned:** Always verify subnet consistency between directly-connected interfaces.

**Issue #2: `192.168.200.0/24` not advertised**
- **Problem:** E2 couldn't learn E1's LAN subnet
- **Root Cause:** Gi1/0 on E1 had no IP configured
- **Solution:** Assigned `192.168.200.1/24` to Gi1/0
- **Lesson Learned:** EIGRP `network` only advertises interfaces that are up with an assigned IP.


## 📂 Repository Structure

```
advanced-routing-lab/
├── README.md
├── topology/
│   └── topology.png                    # GNS3 topology screenshot
├── verification/                        # Command output verification
│   ├── phase1-ospf-neighbors.txt
│   ├── phase1-ospf-interface.txt
│   ├── phase1-ospf-database.txt
│   ├── phase1-ospf-summarization.txt
│   └── phase1-ospf-ping-tests.txt
└── configs/                             # (Coming soon — final router configs)
```

---

## 🛠️ Environment

| Component | Detail |
|-----------|--------|
| **Hypervisor** | GNS3 on Kali Linux |
| **Router Image** | c7200-adventerprisek9 |
| **Switch Image** | vios_l2-adventerprisek9 |
| **Editor** | VS Code Web (vscode.dev) |
| **Version Control** | GitHub |
