# ITN 101 Reference Network

**A complete, documented campus network built on real Cisco hardware — and your reference for how production networks are designed, deployed, and maintained.**

This repository is the knowledge base for the ITN 101 (Introduction to Network Concepts) lab network at Brightpoint Community College, VCCS. It contains the device configurations, IP addressing plan, operational procedures, governance policies, and change-management records for a 3-tier enterprise network with HSRP redundancy. Everything here mirrors the documentation practices used in real-world IT organizations — just scaled to fit our five-device lab rack. Packet Tracer-compatible configs are included for virtual lab work at home.

---

## Architecture

```
              [Internet]
                  |
          +-------+-------+     CORE LAYER
          |  R1 (4221)    |     HSRP Active VL10/40
          |  R2 (4221)    |     HSRP Active VL20
          +-------+-------+     NAT, DHCP, NTP, OSPF
                  |
          +-------+-------+     DISTRIBUTION LAYER
          | SW-DIST (C1000)|     L2 backbone, STP root
          +--+--+--+--+---+     802.1Q trunks
             |  |  |  |
            /   |  |   \__[WAP-1]  Wireless (VLAN 40)
           /    |  |
     [Branch]   |  |               ACCESS LAYER
     R3 (1941)  |  |
       |    +---+--+---+
       |    | SW-ACC    |          VLAN port assignments
       |    | (C2960)   |          PortFast on all access
       |    +--+--+--+--+
       |       |  |  |
     [VL30] [VL10][VL20]
```

### Design Highlights

- **3-Tier Hierarchy** — Core (routing + redundancy), Distribution (L2 aggregation), Access (endpoint connectivity)
- **HSRP Load Balancing** — R1 active for VLANs 10/40, R2 active for VLAN 20; no idle standby
- **OSPF Area 0** — Dynamic routing across all routers; R1 originates default route
- **Router-on-a-Stick** — Inter-VLAN routing via 802.1Q subinterfaces on both core routers
- **Simulated T1 WAN** — GigE link between R2 and R3 with `bandwidth 1544`
- **Wireless VLAN** — Dedicated VLAN 40 for AP clients, HSRP-protected
- **Security Best Practices** — SSH on core routers, VLAN 999 blackhole for unused ports, separate management VLAN

---

## IP Addressing

| Network | Subnet | Gateway | Notes |
|---------|--------|---------|-------|
| VLAN 10 — Students | `192.168.10.0/24` | `.1` (HSRP) | R1 active |
| VLAN 20 — Faculty | `192.168.20.0/24` | `.1` (HSRP) | R2 active |
| VLAN 30 — Servers | `192.168.30.0/24` | `.1` (R3) | Branch LAN |
| VLAN 40 — Wireless | `192.168.40.0/24` | `.1` (HSRP) | R1 active |
| WAN (R2 ↔ R3) | `10.0.1.0/30` | — | Simulated T1 |
| Management | `172.16.0.0/24` | `.1` (R1) | Out-of-band |

**HSRP Convention:** Virtual IP = `.1`, Active router = `.2`, Standby = `.3`

---

## Physical Lab Hardware

| Label | Model | Role | Mgmt IP |
|-------|-------|------|---------|
| R1 | Cisco ISR 4221 | Core (HSRP Active VL10/40) | 172.16.0.1 |
| R2 | Cisco ISR 4221 | Core (HSRP Active VL20) | 172.16.0.2 |
| R3 | Cisco ISR 1941 | Branch Router | 172.16.0.3 |
| SW-DIST | Cisco Catalyst 1000 | Distribution Switch | 172.16.0.11 |
| SW-ACC | Cisco Catalyst 2960 | Access Switch | 172.16.0.12 |
| WAP-1 | TBD | Wireless Access Point | 172.16.0.20 |
| PDU | CyberPower CPS-1215RM | Power Distribution | — |

---

## What's in This Repo

This repository is organized the way a real IT team would maintain documentation for a production network. Each folder serves a specific purpose, and understanding this structure is itself a learning objective — you will encounter these same categories of documentation in any enterprise environment.

### `configs/` — Golden Configurations

A **golden config** (sometimes called a "baseline config") is the known-good, approved configuration for a device. If something breaks, the golden config is what you restore to. The `golden/` subfolder contains the real IOS configs running on our physical lab devices. The `packet-tracer/` subfolder has adjusted versions that work in Cisco Packet Tracer (which uses different device models and doesn't support every command).

### `inventory/` — Asset Inventory & IPAM

In any organization, you need to know what you have and where it is. The inventory spreadsheet tracks our hardware assets (model numbers, serial numbers, rack locations), our **IP Address Management (IPAM)** plan (which IPs are assigned to what), switch port maps (what's plugged into which port), and the cabling matrix (every cable, labeled and documented). When someone asks "what's the IP of SW-DIST?" or "which port is the trunk on?", this is where you look.

### `change-management/` — Requests for Change (RFCs)

In a production environment, you don't just log into a router and start typing commands. **Change management** is the process of proposing, reviewing, approving, and documenting changes before they are made. This prevents outages caused by untested or poorly planned modifications. This folder contains a blank RFC (Request for Change) template in Word format, a filled-in example based on an actual IOS upgrade we performed, and the policy document that defines when an RFC is required and how the approval process works. Even in our small lab, practicing this discipline builds habits that matter in the real world.

### `policies/` — Governance & Standards

Policies are the rules of the road. Every network needs them, and in a professional setting they are often mandated by compliance frameworks (HIPAA, PCI-DSS, SOX, etc.). Our simplified versions cover the essentials:

- **Acceptable Use Policy (AUP)** — What you are and aren't allowed to do on this network
- **Service Level Agreement (SLA)** — Availability targets and support expectations
- **Naming Conventions** — Why R1 is called R1, why VLAN 10 uses 192.168.**10**.x, and every other standard that keeps the network consistent and readable
- **Incident Response** — The step-by-step playbook for "the network is down" — what to check first, what commands to run, who to escalate to
- **Lab Safety** — Physical equipment handling, power procedures, and emergency protocols

### `docs/runbooks/` — Operational Runbooks

A **runbook** (also called a Standard Operating Procedure, or SOP) is a step-by-step procedure for a specific operational task. The idea is that any qualified person should be able to pick up a runbook and execute the procedure without guessing. Good runbooks reduce human error and make knowledge transferable — you don't want critical procedures locked inside one person's head. Our runbooks cover:

- **[IOS Upgrade](docs/runbooks/ios-upgrade.md)** — Upgrading router firmware via USB
- **[Backup & Restore](docs/runbooks/backup-restore.md)** — Saving and restoring device configurations
- **[HSRP Failover Demo](docs/runbooks/hsrp-failover.md)** — Testing redundancy by failing over between routers
- **[Add a VLAN](docs/runbooks/add-vlan.md)** — End-to-end procedure for adding a new VLAN across the entire network
- **Factory Reset** — Returning each device to factory defaults, from easy (admin access) to hard (locked out, password unrecoverable):
  [ISR 4221 (R1/R2)](docs/runbooks/factory-reset-isr4221.md) ·
  [ISR 1941 (R3)](docs/runbooks/factory-reset-isr1941.md) ·
  [Catalyst 1000 (SW-DIST)](docs/runbooks/factory-reset-c1000.md) ·
  [Catalyst 2960 (SW-ACC)](docs/runbooks/factory-reset-c2960.md)

### `diagrams/` — Topology & Rack Layout

Visual references for the network: a [logical topology](diagrams/topology.txt) showing how devices interconnect with IP addresses and VLAN assignments, and a [physical rack elevation](diagrams/rack-layout.md) showing where each device sits in the rack and how power is cabled.

### `docs/` — Reference Documents & Change Log

The [CHANGELOG](docs/CHANGELOG.md) tracks every addition and modification to this repository (versioned using [Keep a Changelog](https://keepachangelog.com/) format). The [KNOWN_ISSUES](docs/KNOWN_ISSUES.md) file documents hardware quirks, Packet Tracer limitations, and open items — because in the real world, every network has a list of things that aren't quite right yet.

### Repository File Tree

```
ITN101-Reference-Network/
├── README.md                          ← You are here
├── LICENSE                            ← CC BY-NC-SA 4.0
├── .gitignore
│
├── configs/
│   ├── golden/                        ← Production IOS configs (real hardware)
│   │   ├── R1.txt
│   │   ├── R2.txt
│   │   ├── R3.txt
│   │   ├── SW-DIST.txt
│   │   └── SW-ACC.txt
│   └── packet-tracer/                 ← PT-compatible configs (adjusted syntax)
│       ├── R1-PT.txt ... SW-ACC-PT.txt
│
├── inventory/
│   └── ITN101_Network_Inventory.xlsx  ← Asset inventory, IPAM, port maps, cabling
│
├── change-management/
│   ├── RFC_Template.docx              ← Blank Request for Change form
│   ├── RFC-001_IOS_Upgrade.docx       ← Pre-filled example RFC
│   └── change-management-policy.md    ← Change management policy
│
├── policies/
│   ├── acceptable-use-policy.md       ← Lab network AUP
│   ├── sla-template.md                ← Service Level Agreement
│   ├── naming-conventions.md          ← IP, VLAN, hostname, and cabling standards
│   ├── incident-response.md           ← Triage and escalation playbook
│   └── lab-safety.md                  ← Physical access and equipment handling
│
├── docs/
│   ├── ITN101_Reference_Network.docx  ← Full reference documentation
│   ├── ITN101_Golden_Configs.docx     ← Consolidated config guide
│   ├── CHANGELOG.md                   ← Version history
│   ├── KNOWN_ISSUES.md                ← Hardware quirks, PT limitations, open items
│   └── runbooks/                      ← Operational procedures (SOPs)
│       ├── ios-upgrade.md
│       ├── backup-restore.md
│       ├── hsrp-failover.md
│       ├── add-vlan.md
│       ├── factory-reset-isr4221.md
│       ├── factory-reset-isr1941.md
│       ├── factory-reset-c1000.md
│       └── factory-reset-c2960.md
│
├── slides/
│   └── ITN101_Reference_Network.pptx  ← 6-slide reference deck
│
├── diagrams/
│   ├── topology.txt                   ← Logical topology + addressing tables
│   └── rack-layout.md                 ← Physical rack elevation diagram
│
└── labs/
    └── PACKET_TRACER_BUILD_GUIDE.md   ← Step-by-step PT assembly instructions
```

---

## Quick Start

### Physical Lab Deployment

1. Cable the rack per the [cabling table](diagrams/topology.txt)
2. Console into each device (9600/8N1)
3. Factory-reset each device to clear any previous configs — see the device-specific runbooks:
   [ISR 4221](docs/runbooks/factory-reset-isr4221.md) ·
   [ISR 1941](docs/runbooks/factory-reset-isr1941.md) ·
   [Catalyst 1000](docs/runbooks/factory-reset-c1000.md) ·
   [Catalyst 2960](docs/runbooks/factory-reset-c2960.md)
4. Paste golden configs **in order**: R1 → R2 → SW-DIST → SW-ACC → R3
   (This order matters — see [backup-restore.md](docs/runbooks/backup-restore.md) for why)
5. Verify with `show standby brief`, `show ip ospf neighbor`, `show ip dhcp binding`

### Packet Tracer Virtual Lab

See [labs/PACKET_TRACER_BUILD_GUIDE.md](labs/PACKET_TRACER_BUILD_GUIDE.md) for detailed instructions

---

## Credentials

This is a **classroom demo environment**. All credentials are intentionally simple.

| Credential | Value |
|-----------|-------|
| Enable secret | `cisco` |
| Console password | `cisco` |
| VTY password (switches/R3) | `cisco` |
| SSH username (R1/R2) | `admin` |
| SSH password (R1/R2) | `cisco` |
| Domain name | `itn101.lab` |

---

## Protocols & Services

| Protocol | Device | Details |
|----------|--------|---------|
| HSRP | R1 + R2 | Load-balanced: R1 active VL10/40, R2 active VL20 |
| OSPF | R1, R2, R3 | Area 0, R1 originates default route |
| DHCP | R1 | Pools for VLANs 10, 20, 40 (.100-.199) |
| DHCP | R3 | Pool for VLAN 30 (.100-.199) |
| DNS | SRV-1 | 192.168.30.10 (all pools reference this) |
| NAT/PAT | R1 | Overload on G0/0/0 |
| NTP | R1 (master) | Stratum 4; all devices sync to R1 |
| SSH | R1, R2 | RSA 2048, version 2, local auth |
| Telnet | SW-DIST, SW-ACC, R3 | Password auth (deliberate contrast with SSH) |
| STP | SW-DIST | Root bridge for all VLANs |

---

## Design Decisions Worth Understanding

This network was built with specific choices that mirror real-world best practices. Understanding *why* things are configured a certain way is just as important as knowing *how*.

**SSH vs. Telnet** — R1 and R2 (the ISR 4221s) use SSH for remote access. SW-DIST, SW-ACC, and R3 use Telnet. This is deliberate. If you run a packet capture on a Telnet session, you'll see the password in plaintext. SSH encrypts everything. Both are in this lab so you can see the difference for yourself.

**OSPF instead of static routes** — We could have hard-coded every route manually. Instead, all three routers run OSPF (Open Shortest Path First), a dynamic routing protocol. Add a new subnet to any router, and every other router learns about it automatically. R1 advertises the default route to the internet via `default-information originate`, so R3 doesn't need a static default — it learns it through OSPF.

**HSRP load balancing** — R1 and R2 both carry traffic simultaneously. R1 is the active gateway for VLANs 10 and 40, R2 is active for VLAN 20. If either router fails, the other takes over all VLANs. Redundancy doesn't mean one router sits idle waiting for a failure.

**VLAN 999 blackhole** — Every unused switch port is assigned to VLAN 999, which has no gateway and no routing. This means someone can't walk up to an empty port, plug in a laptop, and get on the network. It's a basic security measure, but it's one that many production networks skip (and shouldn't).

**Third octet = VLAN ID** — The IP scheme uses the third octet to match the VLAN number: 192.168.**10**.x = VLAN **10**, 192.168.**20**.x = VLAN **20**, and so on. This is a naming convention (documented in [policies/naming-conventions.md](policies/naming-conventions.md)) that makes the entire addressing scheme self-documenting.

### HSRP Failover Demo

This is one of the best demonstrations in the course. See the full procedure in [docs/runbooks/hsrp-failover.md](docs/runbooks/hsrp-failover.md), but the short version: start a continuous ping from a VLAN 10 PC to the gateway (`ping -t 192.168.10.1`), then shut down R1's subinterface. You'll see 1-3 pings drop, then traffic resumes through R2 — automatically, with no human intervention. Bring R1 back up and preemption restores the original state. The whole thing takes about 60 seconds.

### Troubleshooting Sequence

When something isn't working, don't start randomly. Work through this sequence — each step isolates a different layer of the network:

```
1. Ping your own IP             → Is your NIC / TCP-IP stack working?
2. Ping the gateway (.1)        → Is your VLAN / L2 path / HSRP working?
3. Ping the DNS server (30.10)  → Is inter-VLAN routing / the WAN working?
4. Ping 8.8.8.8                 → Is NAT / internet access working?
5. nslookup google.com          → Is DNS resolution working?
```

If step 2 fails but step 1 works, the problem is between your device and the switch (wrong VLAN, bad cable, STP issue). If step 4 fails but step 3 works, the problem is NAT. This layered approach saves time and teaches systematic thinking. See [policies/incident-response.md](policies/incident-response.md) for the full triage playbook.

---

## Course Alignment

- **CompTIA Network+ (N10-009)**: VLANs, subnetting, DHCP/DNS, NAT, routing, redundancy, wireless
- **Cisco CCNA (200-301)**: OSPF, HSRP, SSH, STP, trunk/access ports, router-on-a-stick
- **VCCS ITE/ITN Curriculum**: ITN 101 Introduction to Network Concepts

---

## Contributing

1. Fork the repo
2. Adjust IP schemes, VLANs, or device models to match your lab
3. Submit a PR if you build something others could use (lab worksheets, Packet Tracer activities, etc.)

---

## License

[CC BY-NC-SA 4.0](LICENSE) — Free to share and adapt for non-commercial educational use with attribution.

