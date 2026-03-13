# Changelog

All notable changes to the ITN 101 Reference Network are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/). Dates are ISO 8601.

---

## [1.3.0] — 2026-03-13

### Added
- `docs/runbooks/factory-reset-*.md` — Device-specific factory reset runbooks:
  - `factory-reset-isr4221.md` — ISR 4221 (R1/R2): CLI, ROMMON, no-service-password-recovery
  - `factory-reset-isr1941.md` — ISR 1941 (R3): CLI, ROMMON, no-service-password-recovery
  - `factory-reset-c1000.md` — Catalyst 1000 (SW-DIST): CLI, Mode button, no-service-password-recovery
  - `factory-reset-c2960.md` — Catalyst 2960 (SW-ACC): CLI, Mode button + flash_init, no-service-password-recovery

### Changed
- Updated README.md repo structure tree to include factory reset runbooks

---

## [1.2.0] — 2026-03-13

### Added
- `change-management/` — Change management framework:
  - `RFC_Template.docx` — Blank Request for Change form (Word document)
  - `RFC-001_IOS_Upgrade.docx` — Pre-filled example RFC tied to actual IOS upgrade history
  - `change-management-policy.md` — Change categories, RFC process, and maintenance windows
- `policies/` — Governance and standards documentation:
  - `acceptable-use-policy.md` — Lab network AUP for students and staff
  - `sla-template.md` — Simplified Service Level Agreement with availability targets
  - `naming-conventions.md` — Codified IP, VLAN, hostname, OSPF, and cabling standards
  - `incident-response.md` — Triage playbook with diagnostic commands and escalation path
  - `lab-safety.md` — Physical equipment handling and emergency procedures

### Changed
- Repository structure now includes `change-management/` and `policies/` directories
- Updated README.md repo structure tree

---

## [1.1.0] — 2026-03-13

### Added
- `inventory/ITN101_Network_Inventory.xlsx` — Hardware asset inventory, IPAM plan, switch port maps, cabling matrix, and OSPF summary (6 tabs)
- `docs/CHANGELOG.md` — This file
- `docs/KNOWN_ISSUES.md` — Known issues, hardware quirks, and Packet Tracer limitations
- `docs/runbooks/` — Operational runbooks:
  - `ios-upgrade.md` — IOS upgrade procedure for ISR 4221 via USB
  - `backup-restore.md` — Config backup and restore procedures
  - `hsrp-failover.md` — HSRP failover demo and recovery
  - `add-vlan.md` — Adding a new VLAN end-to-end
- `diagrams/rack-layout.md` — Physical rack elevation diagram (ASCII)

### Changed
- Repository structure now includes `inventory/` and `docs/runbooks/` directories

---

## [1.0.0] — 2026-03-12

### Added
- Initial repository release
- Golden configs for all 5 network devices (R1, R2, R3, SW-DIST, SW-ACC)
- Packet Tracer-compatible configs with device substitutions
- `README.md` with full architecture overview, IP addressing, credentials, protocols, and teaching notes
- `diagrams/topology.txt` — ASCII topology with addressing tables and cabling matrix
- `labs/PACKET_TRACER_BUILD_GUIDE.md` — Step-by-step Packet Tracer assembly instructions
- `docs/ITN101_Reference_Network.docx` — Full reference documentation (accessible format)
- `docs/ITN101_Golden_Configs.docx` — Consolidated config guide with verification commands
- `slides/ITN101_Reference_Network.pptx` — 6-slide reference deck

### Design Decisions
- Domain: `itn101.lab` (academic lab context, not fictional company)
- IP scheme: 3rd octet = VLAN ID (192.168.**10**.x = VLAN **10**)
- HSRP convention: VIP = `.1`, Active = `.2`, Standby = `.3`
- SSH on ISR 4221s, Telnet on 2960/1941 (deliberate security contrast)
- VLAN 999 blackhole for all unused switch ports
- CC BY-NC-SA 4.0 license for educational sharing
