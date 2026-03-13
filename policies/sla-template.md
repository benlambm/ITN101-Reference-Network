# Service Level Agreement (SLA)

**ITN 101 Lab Network — Brightpoint Community College**
**Effective:** Spring 2026 | **Owner:** BET Division

> **Teaching Note:** This is a simplified SLA that models what a real IT department
> provides to its users. In production, SLAs are contractual documents with financial
> penalties. Here, it serves as a reference for students to understand what SLAs
> contain and why they matter.

---

## 1. Service Description

The ITN 101 Lab Network provides a dedicated, isolated network environment for teaching networking fundamentals. The service includes:

- Layer 3 routing across all VLANs (inter-VLAN communication)
- DHCP address assignment for student and faculty VLANs
- DNS resolution via SRV-1 (192.168.30.10)
- Internet access via NAT on R1 (when campus uplink is connected)
- Wireless access via ITN101-WIFI (VLAN 40)
- Console and remote management access to all network devices

---

## 2. Service Hours

| Period | Hours | Notes |
|--------|-------|-------|
| Scheduled class sessions | Per semester schedule | Full service expected |
| Open lab hours | As posted by BET Division | Best-effort support |
| Evenings / weekends | No guaranteed availability | Equipment may be powered down |
| Semester breaks | Offline | Equipment powered down for maintenance |

---

## 3. Availability Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Network availability during class | 99% | Uptime from bell to bell |
| Inter-VLAN routing | Functional within 5 min of power-on | Verified by ping to HSRP VIPs |
| DHCP lease time | < 30 seconds | Time from plug-in to IP assignment |
| HSRP failover recovery | < 10 seconds | Measured by continuous ping during demo |
| Full lab restore from golden configs | < 30 minutes | All 5 devices wiped and reconfigured |

---

## 4. Support and Escalation

| Level | Responder | Response Time | Scope |
|-------|-----------|--------------|-------|
| **L1: Student self-service** | Student | Immediate | Follow troubleshooting sequence in README |
| **L2: Instructor** | Lab instructor | Within class session | Config issues, device reboots, golden config restore |
| **L3: IT Department** | Brightpoint IT | Next business day | Hardware failure, campus uplink issues, equipment replacement |

### Troubleshooting Sequence (L1)

Before escalating, students should attempt:

```
1. Ping own IP               → NIC / TCP-IP stack
2. Ping HSRP virtual GW (.1) → VLAN / L2 / HSRP
3. Ping DNS (192.168.30.10)  → Inter-VLAN routing / WAN
4. Ping 8.8.8.8              → NAT / Internet
5. nslookup google.com       → DNS resolution
```

---

## 5. Exclusions

This SLA does **not** cover:

- Internet speed or bandwidth (campus uplink is shared and not under lab control)
- Packet Tracer virtual labs (software issues are outside network SLA scope)
- Student-caused outages during lab exercises (expected and educational)
- Force majeure (power outages, campus closures, natural disasters)

---

## 6. Maintenance Notifications

Planned maintenance (IOS upgrades, hardware changes) will be:

- Announced at least one class session in advance
- Scheduled outside class hours when possible (Saturday preferred)
- Documented via RFC and logged in CHANGELOG.md

---

## 7. Reporting

Lab network status can be assessed at any time using:

- `show standby brief` on R1 or R2 (HSRP health)
- `show ip ospf neighbor` on any router (routing health)
- `show interfaces status` on SW-ACC (port connectivity)
- Continuous ping from a student PC to 192.168.10.1 (basic availability)

---

## 8. SLA Review

This SLA will be reviewed and updated:

- At the start of each semester
- After any major network redesign
- After any incident that caused a full lab outage
