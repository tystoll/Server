# Homelab Rack Architecture – v3 (32U Design)

## Overview

This document defines the architecture for a scalable homelab system designed to support:

* ML / Quant research pipelines
* Distributed systems (Docker / Kubernetes)
* Media and service hosting (Plex, cameras, etc.)
* Future GPU compute expansion

The system is structured into **layered infrastructure**, following real-world datacenter principles:

* Network Layer
* Control Plane
* Interface / Workstation
* Storage Layer
* Compute Layer
* Power Layer

---

## Rack Specification

* **Rack:** 32U 4-Post Open Frame Server Rack
* **Depth:** Adjustable 24"–37"
* **Type:** Open frame, 4-post, floor standing
* **Mounting:** 19" standard square hole mounting
* **Material:** Steel
* **Use:** IT, AV & Telecom compatible

---

## Rack Layout (Top → Bottom)

```
[ 1U ] Rapink 24-Port Cat6 Patch Panel
[ 1U ] Network Switch (TBD)
[ 1U ] Firewall / Gateway (TBD – WireGuard / OPNsense)

[ 2U ] Mini PC Cluster (control plane – MS-01 Node #1 + future nodes)

[ 1U ] NavePoint Rail Kit – Workstation
[ 4U ] Rosewill RSV-L4000U – Rackmount Workstation (primary interface node)

[ 1U ] NavePoint Rail Kit – NAS
[ 4U ] Rosewill RSV-H424 – NAS Server (storage layer)

[ 4U ] GPU Server (future compute layer – empty for now)

[ 1U ] Pyle 19-Outlet PDU (power distribution)
[ 2U ] Tripp Lite SMART1500LCD – UPS (power layer)
```

**Total Used: 22U**
**Remaining Headroom: ~10U (for expansion, cable management, cooling)**

---

## Layer Breakdown

### 1. Network Layer (Top of Rack)

**Hardware**

| Component | Detail |
|---|---|
| Patch Panel | Rapink 24-Port Cat6 Inline Keystone – 1U Rackmount |
| Switch | TBD (Managed, 10G recommended) |
| Firewall | TBD (OPNsense / WireGuard appliance) |
| PDU | Pyle 19-Outlet 1U 19" Rackmount PDU |
| General Cabling | Cat 8 Ethernet 75ft – 40Gbps 2000MHz Shielded Direct Burial RJ45 |

**10G Direct Link (Workstation ↔ NAS)**

| Component | Detail |
|---|---|
| Workstation NIC | LinksTeK X520-DA2 – 2× 10GbE SFP+ PCIe |
| NAS NIC | LinksTeK X520-DA2 – 2× 10GbE SFP+ PCIe |
| Cable | 10Gtek SFP+ DAC Twinax 1m (direct connect) |

**Purpose:**

* Central network routing and switching
* VPN access (WireGuard / Tailscale)
* Secure ingress/egress point
* Clean cable distribution via patch panel
* Direct 10G link between workstation and NAS

**Notes:**

* X520-DA2 has 2× SFP+ ports each — second port on each card available for uplink to future switch
* DAC cable is a direct passive copper link — no transceiver needed, lowest latency possible
* MS-01 has native dual 10Gb SFP+ + dual 2.5Gb Ethernet — no additional NIC required

---

### 2. Control Plane

#### Primary Node – Minisforum MS-01 (Node #1)

The MS-01 serves as the **dedicated primary control node** and backbone of the system. It replaces the abstract "mini cluster" concept with a concrete, production-grade starting point.

**Hardware**

| Component | Detail |
|---|---|
| Unit | Minisforum MS-01 Mini Workstation |
| CPU | Intel Core i9-13900H (vPro Enterprise Support) |
| RAM | 32GB DDR5 (expandable) |
| Storage | 1TB NVMe SSD (expandable to 3× NVMe + U.2) |
| Networking | 2× 10Gbps SFP+ / 2× 2.5Gb RJ45 |
| Connectivity | 2× USB4 / HDMI |
| Expansion | 1× PCIe 4.0 x16 slot |
| Form Factor | Mini PC (desk / shelf mountable) |
| OS | Ubuntu Server |

**Responsibilities:**

* MT5 terminal (via Wine) – Forex.com account
* Expert Advisors (data output)
* Python socket server (data ingestion)
* Initial database host (Postgres / TimescaleDB – Phase 2)
* Automation services (future: Home Assistant, MQTT)
* API / reverse proxy layer (future)
* Docker host (Phase 2)

**Notes:**

* Always-on operation
* Headless (SSH / remote management)
* Native 10G SFP+ connects directly to switch or workstation via DAC
* PCIe 4.0 x16 slot reserved for future expansion
* Low power consumption relative to CPU performance

#### Future Nodes (Phase 3+)

* Add 2–3 additional Mini PCs (HP EliteDesk / Dell Micro / Lenovo Tiny / additional MS-01)
* Introduce Proxmox or Kubernetes for orchestration
* Scale horizontally as workloads grow

---

### 3. Interface / Workstation Layer

**Case: Rosewill RSV-L4000U**

| Spec | Detail |
|---|---|
| Form Factor | 4U Rackmount |
| Drive Bays | 8 × 3.5" HDD, 3 × 5.25" |
| Motherboard Support | E-ATX compatible |
| Front Fans | 5 × 120mm |
| Rear Fans | 2 × 80mm |
| Front I/O | 2 × USB 3.0 |
| Security | Front panel lock |

**Internal Hardware**

| Component | Detail |
|---|---|
| CPU | Intel Core i7-6700K @ 4.00GHz (4.01 GHz boost) |
| Motherboard | MSI Z170A PC MATE (ATX) |
| RAM | 32.0 GB – 2 × G.Skill Ripjaws DDR4 |
| GPU | Gigabyte NVIDIA GeForce GTX 1080 8GB – full-length, dual-slot |
| Storage (HDD) | 3.64 TB Seagate Barracuda (ST4000DM000) |
| Storage (NVMe) | WD Black SN7100 1TB – Gen4 PCIe, M.2 2280, TLC 3D NAND (OS/boot + Steam library) |
| Storage (SSD) | 224 GB OCZ Agility 3 (legacy – retiring on NVMe arrival) |
| PSU | Corsair RM850e 850W (Standard ATX) |
| CPU Cooling | Tower air cooler with 120mm fan |
| NIC (10G) | LinksTeK X520-DA2 – 2× 10GbE SFP+ PCIe |
| OS | Windows 11 (64-bit, x64) |

**Purpose:**

* Primary user interaction node
* Gaming workstation
* Development environment
* System control interface

**Notes:**

* Connected to monitors/keyboard
* Not part of control plane
* GTX 1080 is a placeholder until GPU server (Phase 4) is built
* Can optionally run VMs, but not required

---

### 4. Storage Layer (NAS)

**Case: Rosewill RSV-H424**

| Spec | Detail |
|---|---|
| Form Factor | 4U Rackmount |
| Drive Bays | 24 × 3.5" Hot Swap |
| Interface | 12Gbps SATA/SAS |
| Motherboard Support | E-ATX & SSI-EEB |
| Cooling | 3 × 120×38mm PWM fans |

**Internal Hardware**

| Component | Detail |
|---|---|
| Motherboard | Asus Z170-A (ATX, DDR4) |
| RAM | 32GB Corsair Vengeance LPX DDR4 3200MHz CL16 (2× 16GB) |
| HBA Card | LSI 9300-16i IT Mode – 16-Port 12Gbps PCIe 3.0 x8, 4× SFF-8643 |
| Storage (Boot) | WD Black SN7100 500GB – Gen4 PCIe, M.2 2280, TLC 3D NAND (OS/boot drive) |
| NIC (10G) | LinksTeK X520-DA2 – 2× 10GbE SFP+ PCIe |
| Data Drives | TBD (HDD array via HBA) |
| PSU | Corsair RM650e 650W ATX 3.1 PCIe 5.1 Fully Modular |
| CPU | TBD |

**Software Options:**

* TrueNAS *(recommended – best ZFS + HBA support)*
* Unraid
* Linux (ZFS)

**Purpose:**

* Centralized long-term data storage
* Parquet datasets (market data, tick/OHLC)
* Model outputs and logs
* Backups
* Media storage (Plex)
* Cold-tier data offload from MS-01 local NVMe

**Storage Tiering Strategy:**

| Tier | Location | Data Type |
|---|---|---|
| Hot | MS-01 Local NVMe | OS, active services, DB, staging |
| Cold | NAS HDD Array | Long-term storage, parquet, backups, media |

**Key Principle:** Hot data stays on local NVMe for lowest latency. Cold data moves to NAS.

**Notes:**

* LSI HBA in IT Mode passes drives directly to OS — no RAID card overhead
* Asus Z170-A has 1× M.2 PCIe 3.0 x4 slot — NVMe boot drive recommended
* 16-port HBA + 24-bay chassis gives full drive population headroom

---

### 5. Compute Layer (Future GPU Server)

**Components:**

* 4U GPU Server Chassis
* Multi-GPU support (2–8 GPUs)

**Purpose:**

* ML training
* Backtesting
* High-performance compute workloads

**Status:**

* Reserved space (not built in Phase 1)

---

### 6. Power Layer (Bottom of Rack)

**UPS: Tripp Lite SMART1500LCD**

| Spec | Detail |
|---|---|
| Form Factor | 2U Rackmount (short depth) |
| Capacity | 1500VA / 900W |
| Outlets | 8 |
| Waveform | PWM Sine Wave |
| Features | AVR, LCD Screen |

**PDU: Pyle 19-Outlet 1U Rackmount**

| Spec | Detail |
|---|---|
| Form Factor | 1U Rackmount |
| Outlets | 19 |
| Mount | 19" standard rack |

**Purpose:**

* UPS provides battery backup, surge protection, graceful shutdown, AVR voltage regulation
* PDU distributes power cleanly to all rack components
* Combined outlet count handles full rack population

**Notes:**

* UPS placed at bottom due to weight
* PDU fed from UPS for protected distribution

---

## Build Phases

### Phase 1 – Control Node Deployment *(Current)*

* ✅ MS-01 purchased (i9-13900H, 32GB DDR5, 1TB NVMe)
* Install Ubuntu Server
* Deploy MT5 via Wine (Forex.com)
* Restore Python socket server
* Validate live data ingestion
* File-based logging (Excel / CSV – temporary)

**Goal:** Stable, always-on control node with working MT5 + ingestion pipeline

---

### Phase 2 – Service Hardening

* Introduce Postgres / TimescaleDB
* Replace file-based logging with database storage
* Add Docker for service management
* Introduce Home Assistant + MQTT
* Begin API / reverse proxy layer
* ✅ Rack selected – 32U 4-Post Open Frame
* Network Layer (Switch + Firewall)
* Workstation (Rosewill RSV-L4000U) migrated to rack

**Goal:** Production-grade service layer with structured data and containerized workloads

---

### Phase 3 – Storage Expansion

* Add NAS – Rosewill RSV-H424 (24-bay hot swap)
* Configure ZFS / TrueNAS
* Integrate cold storage with pipelines
* Begin offloading parquet / archive data from MS-01
* Add additional Mini PC nodes to control plane (optional)
* Introduce Proxmox or Kubernetes (if warranted)

**Goal:** Centralized data layer with hot/cold tiering

---

### Phase 4 – Compute Expansion

* Add GPU server
* Connect to control plane
* Enable distributed ML workloads
* Backtest acceleration

**Goal:** Full ML compute capability

---

## Design Principles

* **Separation of Concerns**

  * Control ≠ Compute ≠ Storage ≠ Interface

* **Incremental Build**

  * Avoid premature clustering
  * Avoid early virtualization overhead
  * Prioritize stability of MT5 + ingestion pipeline
  * Build from a working system, layer by layer

* **Scalability**

  * Add nodes, not rebuild systems
  * 10U headroom available for future expansion

* **Modularity**

  * Each layer can be upgraded independently

* **Always-On Core**

  * MS-01 control plane runs continuously
  * Workstation is user-driven

* **Hot / Cold Data Strategy**

  * Active data lives on local NVMe (low latency)
  * Cold data offloads to NAS (cost-effective scale)

* **Future-Proofing**

  * Reserved rack space for expansion
  * 10G networking ready when scaling

---

## Final Architecture Model

```
User (Monitor / Keyboard)
        ↓
Rack Workstation – Rosewill RSV-L4000U (Interface Node)
        ↓
MS-01 (Control Plane – Node #1)
  ├── MT5 / Expert Advisors
  ├── Python Socket Server
  ├── Postgres / TimescaleDB (Phase 2)
  ├── Docker Services (Phase 2)
  └── Home Assistant / MQTT (Phase 2)
        ↓
NAS – Rosewill RSV-H424 (Cold Storage Layer – Phase 3)
        ↓
GPU Server (Compute Layer – Phase 4)
```

---

## Hardware Summary

| Layer | Hardware | Status |
|---|---|---|
| Rack | 32U 4-Post Open Frame, Adj. 24"–37" | Ordered |
| Rail Kits | NavePoint Adjustable 1U Rack Mount Rails × 2 | Ordered |
| Patch Panel | Rapink 24-Port Cat6 Inline Keystone 1U | Cart |
| PDU | Pyle 19-Outlet 1U Rackmount | Cart |
| Switch | TBD (Managed, 10G recommended) | Pending |
| Firewall | TBD | Pending |
| Cabling | Cat 8 Ethernet 75ft – 40Gbps 2000MHz Shielded Direct Burial RJ45 | Ordered |
| Control Node | Minisforum MS-01 (i9-13900H, 32GB DDR5, 1TB NVMe) | **Purchased** |
| Workstation (Case) | Rosewill RSV-L4000U (4U, 8-bay, E-ATX) | Ordered |
| Workstation (CPU) | Intel Core i7-6700K @ 4.00GHz | Owned |
| Workstation (Mobo) | MSI Z170A PC MATE ATX | Owned |
| Workstation (RAM) | 32GB G.Skill Ripjaws DDR4 (2× stick) | Owned |
| Workstation (GPU) | Gigabyte GTX 1080 8GB (dual-slot) | Owned |
| Workstation (NIC) | LinksTeK X520-DA2 2× 10GbE SFP+ | Cart |
| Workstation (HDD) | 3.64TB Seagate Barracuda | Owned |
| Workstation (NVMe) | WD Black SN7100 1TB Gen4 M.2 2280 (boot + Steam) | Ordered |
| Workstation (SSD) | 224GB OCZ Agility 3 (legacy, retiring) | Owned |
| Workstation (PSU) | Corsair RM850e 850W ATX | Owned |
| Workstation (Cooling) | Tower air cooler, 120mm fan | Owned |
| 10G Link Cable | 10Gtek SFP+ DAC Twinax 1m | Cart |
| NAS (Case) | Rosewill RSV-H424 (4U, 24-bay Hot Swap) | Ordered |
| NAS (Mobo) | Asus Z170-A ATX DDR4 | Ordered |
| NAS (RAM) | 32GB Corsair Vengeance LPX DDR4 3200MHz (2× 16GB) | Ordered |
| NAS (HBA) | LSI 9300-16i IT Mode 16-Port 12Gbps PCIe 3.0 | Ordered |
| NAS (NIC) | LinksTeK X520-DA2 2× 10GbE SFP+ | Cart |
| NAS (NVMe) | WD Black SN7100 500GB Gen4 M.2 2280 (boot drive) | Ordered |
| NAS (PSU) | Corsair RM650e 650W ATX 3.1 Fully Modular | Ordered |
| NAS (CPU) | TBD – LGA1151 6th/7th Gen (in organizer, needs ID) | Owned |
| NAS (Data Drives) | TBD (HDD array) | Pending |
| UPS | Tripp Lite SMART1500LCD (2U, 1500VA/900W) | Ordered |
| Future Cluster Nodes | TBD (Phase 3) | Pending |
| GPU Server | TBD | Phase 4 |

---

## Notes / Considerations

* Use short, uniform Ethernet cables for clean wiring
* Route cables along rack rails (vertical management)
* Open frame rack requires careful cable management planning
* Avoid overloading a single node with mixed workloads
* MS-01 native 10G SFP+ eliminates need for separate NIC on control node
* Plan for future network upgrades (2.5G / 10G)
* 10U of headroom available – plan VLAN/network topology before filling

---

## Status

* [x] Rack acquired
* [x] Control node selected and purchased (MS-01 i9-13900H)
* [ ] Ubuntu Server installed on MS-01
* [ ] MT5 deployed via Wine
* [ ] Python socket server restored
* [ ] Live data ingestion validated
* [ ] Database transition planning begun
* [ ] Network layer installed
* [x] Workstation case acquired (RSV-L4000U)
* [ ] Workstation migrated to rack case
* [x] NAS case acquired (RSV-H424)
* [ ] NAS built
* [ ] GPU server added

---

## Immediate Next Steps

1. Install Ubuntu Server on MS-01
2. Install and configure MT5 via Wine
3. Restore Python socket server
4. Validate live data ingestion
5. Begin database transition planning (Postgres / TimescaleDB)

## Upcoming Next Steps

* Select network hardware (switch + firewall)
* Design cable routing plan for open frame rack
* Choose container / orchestration stack (Docker → Kubernetes / Proxmox)
* Configure networking (VLANs, VPN, remote access)
* Plan NAS drive array and ZFS topology

---

**End of Document**
