# Acceptable Use Policy (AUP)

**ITN 101 Lab Network — Brightpoint Community College**
**Effective:** Spring 2026 | **Owner:** BET Division

---

## Purpose

This policy defines acceptable and prohibited uses of the ITN 101 lab network and equipment. All students, instructors, and staff who access the lab network agree to these terms.

---

## Scope

This policy covers:

- All lab network equipment (routers, switches, access point, PDU, cabling)
- All VLANs and subnets within the ITN 101 network (192.168.x.0/24, 172.16.0.0/24, 10.0.1.0/30)
- Wireless access via SSID `ITN101-WIFI` (VLAN 40)
- Console and SSH/Telnet access to network devices
- Any device connected to lab switch ports

---

## Acceptable Uses

- Completing assigned lab exercises and coursework
- Practicing networking concepts (subnetting, VLAN configuration, routing, etc.)
- Testing configurations in preparation for certifications (CCNA, Network+)
- Instructor-led demonstrations and classroom activities
- Authorized research and experimentation with instructor approval

---

## Prohibited Uses

- Connecting the lab network to the campus production network without IT approval (R1 G0/0/0 is the only authorized uplink)
- Attempting to access systems, networks, or data outside the lab scope
- Running network attacks, vulnerability scanners, or penetration testing tools without explicit instructor authorization
- Installing unauthorized software or firmware on lab devices
- Modifying configurations outside of assigned lab exercises without submitting an RFC
- Removing, disconnecting, or relocating physical equipment without authorization
- Using the lab network for personal internet browsing during class
- Sharing lab credentials (even though they are intentionally simple, the habit matters)
- Connecting personal network equipment (routers, switches, hubs) to lab ports without approval

---

## Credentials and Access

Lab credentials are documented in the project README and are intentionally simple for educational purposes. This does **not** mean security is unimportant — it means students should understand that these credentials exist in a controlled environment and would never be acceptable in production.

| Access Method | Credential | Devices |
|---------------|-----------|---------|
| Enable secret | `cisco` | All |
| Console | `cisco` | All |
| SSH | `admin` / `cisco` | R1, R2 |
| Telnet | `cisco` | SW-DIST, SW-ACC, R3 |
| Wireless (WPA2-PSK) | `cisco12345` | WAP-1 |

---

## Accountability

- Students are responsible for the state of any device they configure
- If you break something, report it — do not leave it for the next class to discover
- All configuration changes should be documented per the Change Management Policy
- The golden configs in `configs/golden/` are the authoritative baseline; any device can be restored to this state

---

## Consequences

Violations of this policy may result in:

1. Loss of lab access for the remainder of the class session
2. Required meeting with the instructor to review the incident
3. Referral to the BET Division Chair for repeated or serious violations
4. Academic integrity proceedings if violations involve unauthorized access to systems

---

## Agreement

By accessing the ITN 101 lab network, users acknowledge that they have read and agree to this policy. The instructor will review this policy during the first lab session of each semester.
