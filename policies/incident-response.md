# Incident Response Playbook

**ITN 101 Lab Network — Brightpoint Community College**
**Effective:** Spring 2026

> When the network is down and a class is waiting, this is the document you open.

---

## Step 1: Assess the Scope

Before touching anything, determine what is actually broken.

**Quick triage from any student PC:**

```
ping 127.0.0.1           → Is my NIC working?
ping [my IP]              → Is TCP/IP bound?
ping [gateway .1]         → Is my VLAN + HSRP working?
ping 192.168.30.10        → Is inter-VLAN routing working?
ping 8.8.8.8              → Is NAT / internet working?
nslookup google.com       → Is DNS working?
```

**Scope matrix:**

| Symptom | Likely Cause | Start Here |
|---------|-------------|------------|
| One PC cannot connect | PC NIC, cable, or port VLAN | Check cable, check `show interfaces status` on SW-ACC |
| One VLAN is down | HSRP issue or trunk misconfiguration | `show standby brief` on R1/R2 |
| All VLANs down | SW-DIST failure or both core routers down | Check power, check `show vlan brief` on SW-DIST |
| No internet, VLANs fine | R1 NAT or ISP uplink issue | `show ip nat translations` on R1, check G0/0/0 |
| Inter-VLAN works, branch down | WAN link R2-R3 or OSPF issue | `show ip ospf neighbor` on R2, check R3 power |
| Everything down | Power issue (PDU) or mass config loss | Check PDU, check if devices are booted |

---

## Step 2: Identify and Isolate

Once you know the scope:

- **Single device issue:** Focus on that device. Console in and check status.
- **Multi-device issue:** Check physical layer first (power, cables), then work up.
- **Do not make unrelated changes** while troubleshooting. Fix one thing at a time.

### Key Diagnostic Commands

**Routers (R1, R2, R3):**

```
show ip interface brief          → Interface status
show standby brief               → HSRP state (R1, R2)
show ip ospf neighbor            → OSPF adjacencies
show ip route                    → Routing table
show ip dhcp binding             → DHCP leases (R1, R3)
show ip nat translations         → NAT table (R1)
show ntp status                  → Clock sync
show logging                     → Recent errors
```

**Switches (SW-DIST, SW-ACC):**

```
show vlan brief                  → VLAN assignments
show interfaces trunk            → Trunk status and allowed VLANs
show interfaces status           → Port status overview
show spanning-tree summary       → STP state
show mac address-table           → MAC learning
```

---

## Step 3: Fix

### Common Fixes

**Port down on SW-ACC:**
```
SW-ACC# configure terminal
SW-ACC(config)# interface FastEthernet0/X
SW-ACC(config-if)# no shutdown
SW-ACC(config-if)# end
```

**HSRP not forming (R1/R2):**
Check that the trunk to SW-DIST is up and the correct VLANs are allowed:
```
SW-DIST# show interfaces trunk
R1# show standby brief
```

**OSPF adjacency lost (R2-R3):**
Check the WAN link:
```
R2# show ip interface brief | include 0/0/1
R3# show ip interface brief | include 0/1
R2# ping 10.0.1.2
```

**DHCP not handing out addresses:**
```
R1# show ip dhcp pool
R1# show ip dhcp binding
R1# show ip dhcp conflict
```
If the pool is exhausted, clear bindings: `clear ip dhcp binding *`

**Full config corruption — restore from golden configs:**
See `docs/runbooks/backup-restore.md`. Restore order: R1 → R2 → SW-DIST → SW-ACC → R3.

---

## Step 4: Verify

After applying a fix, verify using the same triage sequence from Step 1. Also:

- Have a student on a different VLAN confirm their connectivity
- Check that HSRP states are correct: `show standby brief`
- Check that OSPF adjacencies are all FULL: `show ip ospf neighbor`

---

## Step 5: Document

After the incident is resolved:

- Log the incident in `docs/CHANGELOG.md`
- If a permanent config change was made, update the golden configs
- If it was a novel issue, add it to `docs/KNOWN_ISSUES.md`
- Submit a retroactive RFC if the fix involved a non-trivial configuration change

---

## Contact / Escalation

| Role | Contact | When |
|------|---------|------|
| Lab Instructor | Present in classroom | First responder for all lab issues |
| BET Division Office | Building front desk / email | Equipment failure, hardware replacement |
| Brightpoint IT | IT help desk | Campus uplink issues, infrastructure problems |
| Cisco TAC | 1-800-553-2447 (if under SmartNet) | Hardware failure under warranty (unlikely for lab gear) |

---

## Incident Log Template

When documenting an incident, include:

```
Date:
Time noticed:
Time resolved:
Reported by:
Affected devices/VLANs:
Symptom:
Root cause:
Fix applied:
Config changes made (Y/N):
Docs updated:
Lessons learned:
```
