# Known Issues & Lessons Learned

Documented quirks, hardware-specific behaviors, and workarounds discovered during deployment of the ITN 101 Reference Network.

---

## Hardware Issues

### ISR 4221 — NIM Slots Empty
Both R1 and R2 have NIM-1 and NIM-2 slots unpopulated. If serial WAN labs are needed on core routers, a NIM-2T module would be required. Currently, serial labs are only available on R3 via the EHWIC-2T.

### ISR 1941 (R3) — EHWIC-2T Module
The HWIC-2T (actually EHWIC-2T) is installed in EHWIC slot 0, providing Serial0/0/0 and Serial0/0/1. These are left `shutdown` in the golden config. When used for serial labs, the DCE end requires `clock rate 64000`.

### ISR 4221 — IOS Upgrade History
The 4221s were originally running IOS XE 16.10.3a (`ISR4200-ucmk9.16.10.3a.SPA.bin`). They were upgraded via USB to the `universalk9_ias` image (16.12.x). The upgrade procedure is documented in `docs/runbooks/ios-upgrade.md`. The old image was deleted from bootflash after upgrade.

### Catalyst 1000 (SW-DIST) — IOS Version
Running IOS 15.0(2k) per rear chassis label. Model is C1000-24T-4G-L V01 (24 GigE copper + 4 SFP uplinks). The SFP uplinks are not used in this topology.

### Asset Tags — Legacy College Name
Physical asset labels read "John Tyler Community College Property" with IT ASSET barcodes. JTCC merged into Brightpoint Community College (VCCS realignment, 2022). Known asset tags: R3 = #41728, R2 = #42231. Remaining tags should be recorded during next physical audit.

### CyberPower PDU (CPS-1215RM)
15A / 120V, 10-outlet rackmount unit. No network management interface. Located at top of rack (U1). Verify that total draw of all devices stays well under 15A capacity.

### Rack — StarTech.com
Open-frame rack visible in lab photos. Exact model TBD. Wall-mounted or table-mounted depending on classroom layout.

---

## Packet Tracer Limitations

These are documented in `labs/PACKET_TRACER_BUILD_GUIDE.md` but repeated here for the centralized issue tracker.

### Device Substitutions Required
PT does not have exact models for ISR 4221 or Catalyst 1000. Substitutions: 4221 → 4321, C1000 → 3560-24PS. Interface naming differs (e.g., `Gi0/1` in PT vs `Gi1/0/1` on physical C1000).

### `bandwidth 1544` on WAN Link
PT accepts the command but it may not affect OSPF cost calculations in all PT versions. On real hardware, this correctly sets the OSPF interface cost.

### `ntp master` Behavior
Works in PT but `show ntp status` may show limited or inconsistent detail compared to real IOS output.

### HSRP Timer Resolution
HSRP failover in PT simulation mode is noticeably slower than in real-time mode. For classroom demos, use real-time mode for more realistic failover timing (1–3 seconds).

### Limited `show` Command Output
Several verification commands produce less detail in PT than on real hardware. Notable examples: `show standby brief` formatting, `show ip ospf interface` detail, `show controllers serial`.

---

## Configuration Notes

### DHCP Excluded Ranges
R1 excludes `.1` through `.10` for VLANs 10, 20, and 40. R3 excludes `.1` through `.20` for VLAN 30 (wider range to accommodate future static servers). DHCP clients receive addresses starting from `.11` or `.21` respectively.

### NAT ACL Scope
R1's NAT ACL (access-list 1) permits all five internal subnets including 172.16.0.0/24 (management). This is intentional for lab simplicity but in production, management traffic would typically not be NATted to the internet.

### STP Root Bridge
SW-DIST is configured as STP root primary for VLANs 10, 20, 30, 40, and 99. This is hardcoded rather than relying on default bridge priority, ensuring deterministic STP behavior.

### OSPF Passive Interfaces
R1: G0/0/0 (ISP-facing) is passive — prevents OSPF hello packets from leaking to the campus/ISP network. R3: G0/0 (Branch LAN) is passive — no need for OSPF on the server VLAN.

---

## Open Items / TBD

- [ ] Record serial numbers for all devices (run `show version` on each)
- [ ] Record MAC addresses for all interfaces (for IPAM completeness)
- [ ] Confirm exact IOS version on R3 and SW-ACC (run `show version`)
- [ ] Identify and document WAP-1 make/model
- [ ] Record remaining asset tags (R1, SW-DIST, SW-ACC)
- [ ] Verify CyberPower PDU outlet assignments
- [ ] Document StarTech rack model number
- [ ] Label all cables with cable IDs from cabling matrix
