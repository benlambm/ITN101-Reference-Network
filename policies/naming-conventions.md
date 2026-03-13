# Network Naming Conventions & Design Standards

**ITN 101 Reference Network — Brightpoint Community College**
**Effective:** Spring 2026

> This document codifies the design decisions embedded in the ITN 101 network.
> Any future changes should follow these conventions for consistency.

---

## IP Addressing

### Core Rule: Third Octet = VLAN ID

```
192.168.10.x  →  VLAN 10 (Students)
192.168.20.x  →  VLAN 20 (Faculty)
192.168.30.x  →  VLAN 30 (Servers)
192.168.40.x  →  VLAN 40 (Wireless)
```

This mnemonic makes the entire addressing scheme self-documenting. When adding new VLANs, always assign the subnet to match: VLAN 50 = 192.168.50.0/24.

### HSRP Address Convention

| Role | Address | Example (VLAN 10) |
|------|---------|--------------------|
| Virtual IP (gateway) | .1 | 192.168.10.1 |
| Active router | .2 | 192.168.10.2 (R1) |
| Standby router | .3 | 192.168.10.3 (R2) |

Clients always point to `.1`. The physical router addresses (`.2` and `.3`) are only used for management and OSPF.

### Reserved Ranges

| Range | Purpose |
|-------|---------|
| .1 – .10 | Gateway, HSRP, and infrastructure (excluded from DHCP) |
| .11 – .20 | Static servers (VLAN 30 reserves through .20) |
| .21 – .254 | DHCP dynamic pool |

### Special Subnets

| Subnet | Purpose |
|--------|---------|
| 172.16.0.0/24 | Management (VLAN 99) — out-of-band |
| 10.0.1.0/30 | WAN point-to-point (R2 to R3) |

---

## VLAN Numbering

| VLAN ID | Name | Purpose |
|---------|------|---------|
| 10 | STUDENTS | Student workstation access |
| 20 | FACULTY | Faculty/instructor workstation access |
| 30 | SERVERS | Branch server LAN |
| 40 | WIRELESS | Wireless client access (WAP-1) |
| 99 | MANAGEMENT | Device management (SVIs, console hosts) |
| 999 | BLACKHOLE | Unused ports — no IP, shutdown |

**Rules:**

- VLAN 1 is never used (default VLAN security best practice)
- VLAN 999 is always the blackhole — all unused ports go here
- VLAN 99 is always management — native VLAN on all trunks
- User VLANs start at 10 and increment by 10

---

## Hostnames

| Convention | Example |
|------------|---------|
| Routers | R1, R2, R3 (role + number) |
| Switches | SW-DIST, SW-ACC (SW + layer abbreviation) |
| Access points | WAP-1 (WAP + number) |
| Servers | SRV-1 (SRV + number) |
| PCs | PC-A, PC-B, PC-C (PC + letter) |

Hostnames are kept short for readability in `show` command output. In a larger deployment, a more descriptive convention would be used (e.g., `RTR-CORE-01`, `SW-ACC-BLD2-FL1`).

---

## Interface Descriptions

All configured interfaces must have a description. The format is:

```
ROLE_DESTINATION
```

Examples from the golden configs:

| Interface | Description |
|-----------|-------------|
| R1 G0/0/0 | `ISP_UPLINK` |
| R1 G0/0/1 | `TRUNK_TO_SW-DIST` |
| R1 G0/0/1.10 | `VLAN10_STUDENTS` |
| R2 G0/0/1 | `WAN_TO_R3_BRANCH` |
| SW-ACC Fa0/1-4 | `STUDENT_PC` |
| SW-ACC Fa0/13-23 | `UNUSED_BLACKHOLED` |

Unused ports always get `UNUSED_BLACKHOLED`.

---

## Domain Name

```
itn101.lab
```

Used for SSH key generation and DHCP domain-name option. The `.lab` TLD signals that this is a non-production environment.

---

## OSPF

| Parameter | Convention |
|-----------|-----------|
| Process ID | 1 (single process) |
| Area | 0 (single area, backbone) |
| Router ID | `X.X.X.X` matching hostname number (R1=1.1.1.1, R2=2.2.2.2, R3=3.3.3.3) |
| Passive interfaces | All non-OSPF-facing interfaces (ISP uplink, branch LAN) |

---

## HSRP

| Parameter | Convention |
|-----------|-----------|
| Group number | = VLAN ID (Group 10 for VLAN 10, etc.) |
| Priority (Active) | 110 |
| Priority (Standby) | 100 |
| Preempt | Enabled on all groups |

---

## NTP

| Role | Device |
|------|--------|
| Master (stratum 4) | R1 (172.16.0.1) |
| Client | All other devices sync to R1 |

---

## Credentials (Lab Environment)

| Credential | Convention |
|-----------|-----------|
| Enable secret | `cisco` |
| Console password | `cisco` |
| SSH (R1, R2 only) | `admin` / `cisco` |
| Telnet (switches, R3) | `cisco` |
| Wireless PSK | `cisco12345` |

These credentials are intentionally simple and documented. In production, credentials would be unique per device, stored in a vault, and rotated regularly.

---

## Cable Labeling

Cables should be labeled at both ends using the format:

```
C-NN
```

Where NN is a sequential number matching the Cabling Matrix in the inventory spreadsheet. Example: `C-02` connects R1 G0/0/1 to SW-DIST Gi1/0/1.
