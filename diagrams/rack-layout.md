# Physical Rack Layout

**Rack:** StarTech.com open-frame (model TBD)
**Location:** Brightpoint Community College, BET Division, Networking Lab
**Orientation:** Front-facing, top-to-bottom

```
┌─────────────────────────────────────────────────────────┐
│  U1   CyberPower CPS-1215RM (PDU)                      │
│       15A / 120V · 10 outlets · No network mgmt         │
├─────────────────────────────────────────────────────────┤
│  U2   Cisco ISR 4221 — R1 (Core)                        │
│       HSRP Active VL10/40 · NAT · DHCP · NTP Master     │
│       [G0/0/0 ISP] [G0/0/1 trunk] [Console] [NIM][NIM] │
│       Mgmt: 172.16.0.1 · Asset Tag: TBD                 │
├─────────────────────────────────────────────────────────┤
│  U3   Cisco ISR 4221 — R2 (Core)                        │
│       HSRP Active VL20 · WAN transit to R3               │
│       [G0/0/0 trunk] [G0/0/1 WAN] [Console] [NIM][NIM] │
│       Mgmt: 172.16.0.2 · Asset Tag: 42231 (JTCC)        │
├─────────────────────────────────────────────────────────┤
│  U4   Cisco ISR 1941 — R3 (Branch)                      │
│       Branch LAN VLAN 30 · EHWIC-2T serial module        │
│       [G0/0 LAN] [G0/1 WAN] [S0/0/0] [S0/0/1] [Cons]  │
│       Mgmt: 172.16.0.3 · Asset Tag: 41728 (JTCC)        │
├─────────────────────────────────────────────────────────┤
│  U5   Cisco Catalyst 1000 — SW-DIST (Distribution)      │
│       C1000-24T-4G-L · IOS 15.0(2k) · STP Root          │
│       [Gi1/0/1 R1][Gi1/0/2 R2][Gi1/0/3 ACC][Gi1/0/4 WAP]│
│       Mgmt: 172.16.0.11 · Asset Tag: TBD                │
├─────────────────────────────────────────────────────────┤
│  U6   Cisco Catalyst 2960 — SW-ACC (Access)             │
│       24x FastEthernet · PortFast all access ports       │
│       [Fa0/1-4 VL10][Fa0/5-8 VL20][Fa0/9-12 VL30]     │
│       [Fa0/24 trunk to SW-DIST]                          │
│       Mgmt: 172.16.0.12 · Asset Tag: TBD                │
└─────────────────────────────────────────────────────────┘

        WAP-1 (TBD model) — Wall/shelf mounted
        SSID: ITN101-WIFI · VLAN 40 · Mgmt: 172.16.0.20


   ┌────────────────── Power Cabling ──────────────────┐
   │  PDU Outlet → Device                               │
   │  ─────────────────────────────────                  │
   │  Outlet 1  → R1 (ISR 4221) — 12VDC 7.5A adapter   │
   │  Outlet 2  → R2 (ISR 4221) — 12VDC 7.5A adapter   │
   │  Outlet 3  → R3 (ISR 1941) — internal PSU          │
   │  Outlet 4  → SW-DIST (C1000) — internal PSU        │
   │  Outlet 5  → SW-ACC (C2960) — internal PSU         │
   │  Outlet 6  → WAP-1 (TBD)                           │
   │  Outlets 7-10 → Available                           │
   └────────────────────────────────────────────────────┘
```

## Notes

- The ISR 4221 uses an external 12VDC / 7.5A power brick (not a standard IEC C13 cord). Ensure the PDU outlets can accommodate the adapter plugs.
- The ISR 1941, C1000, and C2960 all use standard IEC C13 power cords.
- Asset tags labeled "John Tyler Community College" reflect the pre-2022 name (now Brightpoint Community College, VCCS).
- Rack unit assignments are approximate — verify against actual mounting positions during next physical audit.
- Console cables (USB-serial rollover) are run from a shared laptop to R1, R2, and R3. Switches are managed via Telnet from a VLAN 99 host.
