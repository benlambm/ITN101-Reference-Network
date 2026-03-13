# Packet Tracer Build Guide

## Overview

This guide walks you through building the ITN 101 Reference Network in
Cisco Packet Tracer. The topology mirrors the physical lab rack at Brightpoint,
with device substitutions where PT does not offer the exact model.

## Device Substitutions

| Physical Lab      | Packet Tracer         | Why                                    |
|-------------------|-----------------------|----------------------------------------|
| ISR 4221 (x2)    | 4321 (x2)             | PT has no 4221; 4321 is closest match  |
| ISR 1941          | 1941                  | Direct match                           |
| Catalyst 1000     | 3560-24PS             | PT has no C1000; 3560 supports L2/L3   |
| Catalyst 2960     | 2960-24TT             | Direct match                           |
| Wireless AP       | AccessPoint-PT        | Generic PT access point                |
| PCs/Servers       | PC-PT / Server-PT     | Standard PT endpoints                  |

## Step 1: Place Devices

Open Packet Tracer and place the following devices on the logical workspace.
Arrange them in a vertical hierarchy for clarity.

**Top (Core Layer):**
- 1x 4321 router → label `R1`
- 1x 4321 router → label `R2`

**Middle (Distribution Layer):**
- 1x 3560-24PS switch → label `SW-DIST`

**Lower-Middle (Access Layer):**
- 1x 2960-24TT switch → label `SW-ACC`
- 1x AccessPoint-PT → label `WAP-1`

**Right Side (Branch):**
- 1x 1941 router → label `R3`

**Bottom (Endpoints):**
- 3x PC-PT → label `PC-A`, `PC-B`, `PC-C`
- 1x Server-PT → label `SRV-1`

## Step 2: Add Modules to R3

The 1941 needs a serial module for future lab exercises:

1. Click R3
2. Go to the **Physical** tab
3. **Power off** the router (click the power switch)
4. From the module list, find **HWIC-2T**
5. Drag it into an open HWIC slot
6. **Power on** the router

This adds Serial0/0/0 and Serial0/0/1.

## Step 3: Cable Everything

Use **straight-through copper** cables unless noted otherwise.

| From               | Port          | To              | Port          | Cable Type      |
|--------------------|---------------|-----------------|---------------|-----------------|
| R1                 | G0/0/1        | SW-DIST         | G0/1          | Straight        |
| R2                 | G0/0/0        | SW-DIST         | G0/2          | Straight        |
| R2                 | G0/0/1        | R3              | G0/1          | Cross-over*     |
| SW-DIST            | Fa0/1         | SW-ACC          | Fa0/24        | Straight        |
| SW-DIST            | Fa0/2         | WAP-1           | Port 0        | Straight        |
| SW-ACC             | Fa0/1         | PC-A            | Fa0            | Straight        |
| SW-ACC             | Fa0/2         | PC-B            | Fa0            | Straight        |
| SW-ACC             | Fa0/5         | PC-C            | Fa0            | Straight        |
| SW-ACC             | Fa0/9         | SRV-1           | Fa0            | Straight        |

*PT may auto-negotiate with straight-through. Try cross-over first for router-to-router.

**Optional:** Connect R1 G0/0/0 to a Cloud-PT or leave disconnected.

## Step 4: Apply Configurations

For each device, open the CLI tab and paste the corresponding config file
from `configs/packet-tracer/`.

**Configuration order matters.** Follow this sequence:

1. **R1** → `R1-PT.txt` (establishes HSRP, DHCP, NTP, NAT, OSPF)
2. **R2** → `R2-PT.txt` (joins HSRP, connects WAN)
3. **SW-DIST** → `SW-DIST-PT.txt` (creates VLANs, trunks)
4. **SW-ACC** → `SW-ACC-PT.txt` (assigns port VLANs)
5. **R3** → `R3-PT.txt` (branch LAN, OSPF, DHCP)

### Tips for pasting configs in PT

- PT CLI can be slow with large pastes. Paste in sections (one `!` block at a time).
- After `crypto key generate rsa`, PT will prompt: `How many bits? [512]:` — type `2048` and press Enter.
- After `copy running-config startup-config`, PT will prompt for filename — press Enter to accept.

## Step 5: Configure Endpoints

### PC-A and PC-B (VLAN 10 - Students)
- IP Configuration: **DHCP**
- Should receive 192.168.10.x with gateway 192.168.10.1

### PC-C (VLAN 20 - Faculty)
- IP Configuration: **DHCP**
- Should receive 192.168.20.x with gateway 192.168.20.1

### SRV-1 (VLAN 30 - Branch Server)
- IP: `192.168.30.10`
- Subnet: `255.255.255.0`
- Gateway: `192.168.30.1`
- DNS: `192.168.30.10` (itself)
- Enable DNS service in the **Services** tab

### WAP-1 (Wireless AP)
- In the AP Config tab, set SSID to `ITN101-WIFI`
- Set to WPA2-PSK with passphrase `cisco12345`
- The AP bridges wireless clients into VLAN 40

## Step 6: Verify

Run these commands and confirm expected output:

```
R1# show standby brief
```
→ R1 Active for groups 10, 40. Standby for group 20.

```
R1# show ip ospf neighbor
```
→ R2 should appear as FULL neighbor.

```
R2# show ip ospf neighbor
```
→ R1 and R3 should both appear.

```
R3# show ip route ospf
```
→ Should see routes to 192.168.10.0, .20.0, .40.0, 172.16.0.0 and O*E2 default.

```
PC-A> ping 192.168.30.1
```
→ Confirms end-to-end: VLAN 10 → HSRP → OSPF → WAN → Branch LAN.

## HSRP Failover Demo

1. On PC-A, open Command Prompt: `ping -t 192.168.10.1`
2. On R1 CLI: `configure terminal` → `interface g0/0/1.10` → `shutdown`
3. Watch: 1-3 pings fail, then traffic resumes through R2
4. On R1: `no shutdown`
5. Preemption restores R1 as Active

## Known PT Limitations

- `bandwidth 1544` may be accepted but not affect routing metrics in PT OSPF
- `ntp master` works but `show ntp status` may show limited detail
- HSRP timers are slower in PT simulation mode than in real-time mode
- Some `show` commands produce less detail than real IOS
- PT 3560 interface naming (Gi0/1 vs Gi1/0/1) differs from physical C1000
