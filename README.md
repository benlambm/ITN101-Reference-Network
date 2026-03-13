# ITN 101 Reference Network

**A complete, documented campus network topology for teaching networking fundamentals.**

Built for ITN 101 (Introduction to Network Concepts) at Brightpoint Community College, VCCS. This repo contains everything needed to deploy, document, and teach a 3-tier enterprise network with HSRP redundancy on real Cisco hardware — plus Packet Tracer-compatible configs for virtual lab work.

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

## Repository Structure

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
│   │
│   └── packet-tracer/                 ← PT-compatible configs (adjusted syntax)
│       ├── R1-PT.txt
│       ├── R2-PT.txt
│       ├── R3-PT.txt
│       ├── SW-DIST-PT.txt
│       └── SW-ACC-PT.txt
│
├── docs/
│   ├── ITN101_Reference_Network.docx  ← Full reference documentation (accessible)
│   └── ITN101_Golden_Configs.docx     ← Consolidated config guide with verification
│
├── slides/
│   └── ITN101_Reference_Network.pptx  ← 6-slide reference deck
│
├── diagrams/
│   └── topology.txt                   ← ASCII topology + addressing tables
│
└── labs/
    └── PACKET_TRACER_BUILD_GUIDE.md   ← Step-by-step PT assembly instructions
```

---

## Quick Start

### Physical Lab Deployment

1. Cable the rack per the [cabling table](diagrams/topology.txt)
2. Console into each device (9600/8N1)
3. Erase any existing configs: `erase startup-config` → `reload`
4. For switches also: `delete flash:vlan.dat`
5. Paste configs **in order**: R1 → R2 → SW-DIST → SW-ACC → R3
6. Verify with `show standby brief`, `show ip ospf neighbor`, `show ip dhcp binding`

### Packet Tracer Virtual Lab

See [labs/PACKET_TRACER_BUILD_GUIDE.md](labs/PACKET_TRACER_BUILD_GUIDE.md) for detailed instructions including:
- Device substitutions (PT uses 4321 instead of 4221, 3560 instead of C1000)
- Module installation (HWIC-2T for R3 serial ports)
- Cabling diagram
- Config paste tips
- Endpoint configuration
- Verification commands
- HSRP failover demo procedure

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

## Teaching Notes

### Deliberate Pedagogical Choices

- **SSH on 4221s, Telnet on 2960/1941**: Creates a natural "why does this matter?" conversation about plaintext credentials
- **OSPF over static routes**: Only 3-4 lines per router, but students see dynamic route learning in real-time
- **HSRP load balancing**: Both routers carry traffic — demonstrates that redundancy doesn't mean idle hardware
- **`default-information originate`**: R3 learns the default route via OSPF, showing how dynamic routing distributes reachability
- **VLAN 999 blackhole**: Security best practice baked into the baseline — unused ports are never in VLAN 1
- **Third octet = VLAN ID**: 192.168.**10**.x = VLAN **10** — the mnemonic that makes the whole scheme memorable

### HSRP Failover Demo (Live in Class)

1. Student starts continuous ping from VLAN 10 PC: `ping -t 192.168.10.1`
2. Instructor shuts down R1 subinterface: `interface G0/0/1.10` → `shutdown`
3. 1-3 pings drop, traffic resumes through R2
4. Re-enable: `no shutdown` — preemption restores R1 as Active
5. This takes ~60 seconds and is one of the most memorable demos in the course

### Troubleshooting Sequence (Teach This)

```
1. Ping own IP                → NIC / TCP-IP stack
2. Ping HSRP virtual GW (.1) → VLAN / L2 / HSRP
3. Ping DNS (192.168.30.10)  → Inter-VLAN routing / WAN
4. Ping 8.8.8.8              → NAT / Internet
5. nslookup google.com       → DNS resolution
```

---

## Course Alignment

- **CompTIA Network+ (N10-009)**: VLANs, subnetting, DHCP/DNS, NAT, routing, redundancy, wireless
- **Cisco CCNA (200-301)**: OSPF, HSRP, SSH, STP, trunk/access ports, router-on-a-stick
- **VCCS ITE/ITN Curriculum**: ITN 101 Introduction to Network Concepts

---

## Contributing

This is a teaching resource. If you're a VCCS instructor or networking educator and want to adapt it:

1. Fork the repo
2. Adjust IP schemes, VLANs, or device models to match your lab
3. Submit a PR if you build something others could use (lab worksheets, Packet Tracer activities, etc.)

---

## License

[CC BY-NC-SA 4.0](LICENSE) — Free to share and adapt for non-commercial educational use with attribution.

**Benjamin Lamb** · Brightpoint Community College · BET Division · Spring 2026
