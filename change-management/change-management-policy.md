# Change Management Policy

**ITN 101 Reference Network — Brightpoint Community College**
**Effective:** Spring 2026 | **Owner:** Lab Instructor / Network Administrator

---

## Purpose

This policy defines how changes to the ITN 101 lab network are proposed, approved, implemented, and documented. Even in a classroom environment, practicing change management builds habits that prevent outages and configuration drift in production networks.

---

## Scope

This policy applies to any modification to:

- Router or switch configurations (running-config or startup-config)
- IOS/firmware upgrades or downgrades
- Physical cabling or rack layout changes
- VLAN additions, modifications, or removals
- IP addressing changes
- DHCP, DNS, NAT, or OSPF configuration changes
- Physical hardware additions or replacements

It does **not** apply to:

- Packet Tracer lab exercises (virtual environments are disposable)
- Read-only `show` commands on lab equipment
- Endpoint configuration (student PCs connecting to access ports)

---

## Change Categories

| Category | Description | Approval Required | Example |
|----------|-------------|-------------------|---------|
| **Standard** | Pre-approved, low-risk, routine | No (pre-approved) | Restoring golden configs, restarting a device |
| **Normal** | Planned change with known scope | Yes (RFC required) | Adding a VLAN, upgrading IOS, changing OSPF config |
| **Emergency** | Unplanned fix for a live outage | Post-hoc approval OK | Restoring connectivity after accidental misconfiguration |

---

## Request for Change (RFC) Process

### 1. Submit

Fill out the RFC template (`change-management/RFC_Template.docx`) with:

- What is changing and why
- Which devices are affected
- Step-by-step implementation plan (reference runbooks where applicable)
- Rollback plan
- Verification steps

### 2. Review & Approve

For the ITN 101 lab, the approval chain is:

- **Submitter** (student or instructor) fills out the RFC
- **Lab Instructor** reviews and approves (or returns for revision)
- **IT Department** approval only if the change affects campus infrastructure (e.g., R1 G0/0/0 ISP uplink)

### 3. Implement

- Execute during the agreed maintenance window
- Follow the implementation plan step by step
- If unexpected issues arise, execute the rollback plan
- Do not improvise changes outside the RFC scope

### 4. Verify

- Run all verification commands listed in the RFC
- Confirm end-to-end connectivity
- Check that no unintended side effects occurred

### 5. Document

After every change, update:

- `docs/CHANGELOG.md` — date, who, what, why
- `inventory/ITN101_Network_Inventory.xlsx` — if IPs, ports, or hardware changed
- `configs/golden/` — if the change becomes part of the new baseline
- `docs/KNOWN_ISSUES.md` — if the change introduced or resolved a known issue
- The RFC itself — fill in Section 8 (Post-Implementation Review)

### 6. File

Save completed RFCs in the `change-management/` directory with the naming convention:

```
RFC-NNN_Short_Description.docx
```

Example: `RFC-001_IOS_Upgrade.docx`

---

## Maintenance Windows

For the ITN 101 lab network, the preferred maintenance windows are:

- **Saturday mornings (0800–1200)** — lab not in use
- **Before/after scheduled class sessions** — coordinate with instructor
- **Never during a live class** unless it is a planned classroom demo

---

## Emergency Changes

If the network is down and a class is in session:

1. Fix the immediate issue (restore connectivity)
2. Document what happened and what was changed
3. Submit a retroactive RFC within 24 hours
4. Update CHANGELOG.md

---

## Why This Matters (Teaching Context)

In production networks, unauthorized or undocumented changes are the leading cause of outages. This policy is intentionally lightweight for a classroom setting, but it models the discipline that organizations like hospitals, banks, and ISPs rely on every day. Students who internalize "document before you change" will stand out in any IT role.
