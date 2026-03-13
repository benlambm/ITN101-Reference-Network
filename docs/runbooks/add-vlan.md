# Runbook: Adding a New VLAN (End-to-End)

**Applies to:** SW-DIST, SW-ACC, R1, R2 (if HSRP required)
**Estimated time:** 10–15 minutes

> This procedure walks through every step required to add a new VLAN to the
> ITN 101 network, from VLAN creation through DHCP, routing, and verification.

---

## Prerequisites

- Decide on: VLAN ID, name, subnet, gateway, DHCP range
- Follow the IP convention: 3rd octet = VLAN ID (e.g., VLAN 50 → 192.168.50.0/24)
- Follow the HSRP convention: VIP = `.1`, Active = `.2`, Standby = `.3`

## Example: Adding VLAN 50 — "GUESTS"

| Parameter | Value |
|-----------|-------|
| VLAN ID | 50 |
| Name | GUESTS |
| Subnet | 192.168.50.0/24 |
| HSRP VIP | 192.168.50.1 |
| R1 IP | 192.168.50.2 (Active, pri 110) |
| R2 IP | 192.168.50.3 (Standby, pri 100) |
| DHCP range | .11 – .254 |
| Access ports | SW-ACC Fa0/13–Fa0/16 |

---

## Step 1: Create VLAN on Both Switches

### SW-DIST
```
SW-DIST# configure terminal
SW-DIST(config)# vlan 50
SW-DIST(config-vlan)# name GUESTS
SW-DIST(config-vlan)# exit
```

### SW-ACC
```
SW-ACC# configure terminal
SW-ACC(config)# vlan 50
SW-ACC(config-vlan)# name GUESTS
SW-ACC(config-vlan)# exit
```

## Step 2: Allow VLAN on Trunk Links

### SW-DIST — Trunk to R1 (if R1 will route this VLAN)
```
SW-DIST(config)# interface GigabitEthernet1/0/1
SW-DIST(config-if)# switchport trunk allowed vlan add 50
SW-DIST(config-if)# exit
```

### SW-DIST — Trunk to R2 (if R2 will route this VLAN)
```
SW-DIST(config)# interface GigabitEthernet1/0/2
SW-DIST(config-if)# switchport trunk allowed vlan add 50
SW-DIST(config-if)# exit
```

### SW-DIST — Trunk to SW-ACC
```
SW-DIST(config)# interface GigabitEthernet1/0/3
SW-DIST(config-if)# switchport trunk allowed vlan add 50
SW-DIST(config-if)# exit
```

## Step 3: Assign Access Ports on SW-ACC

Reclaim ports from the blackhole VLAN:
```
SW-ACC(config)# interface range FastEthernet0/13 - 16
SW-ACC(config-if-range)# description GUEST_PC
SW-ACC(config-if-range)# switchport mode access
SW-ACC(config-if-range)# switchport access vlan 50
SW-ACC(config-if-range)# spanning-tree portfast
SW-ACC(config-if-range)# no shutdown
SW-ACC(config-if-range)# exit
```

## Step 4: Create Router Subinterfaces (HSRP)

### R1 (Active for VLAN 50)
```
R1(config)# interface GigabitEthernet0/0/1.50
R1(config-subif)# description VLAN50_GUESTS
R1(config-subif)# encapsulation dot1Q 50
R1(config-subif)# ip address 192.168.50.2 255.255.255.0
R1(config-subif)# ip nat inside
R1(config-subif)# standby 50 ip 192.168.50.1
R1(config-subif)# standby 50 priority 110
R1(config-subif)# standby 50 preempt
R1(config-subif)# exit
```

### R2 (Standby for VLAN 50)
```
R2(config)# interface GigabitEthernet0/0/0.50
R2(config-subif)# description VLAN50_GUESTS
R2(config-subif)# encapsulation dot1Q 50
R2(config-subif)# ip address 192.168.50.3 255.255.255.0
R2(config-subif)# standby 50 ip 192.168.50.1
R2(config-subif)# standby 50 priority 100
R2(config-subif)# standby 50 preempt
R2(config-subif)# exit
```

## Step 5: Add to OSPF (R1)

```
R1(config)# router ospf 1
R1(config-router)# network 192.168.50.0 0.0.0.255 area 0
R1(config-router)# exit
```

Also on R2 if both routers advertise the subnet:
```
R2(config)# router ospf 1
R2(config-router)# network 192.168.50.0 0.0.0.255 area 0
R2(config-router)# exit
```

## Step 6: Add NAT ACL Entry (R1)

```
R1(config)# access-list 1 permit 192.168.50.0 0.0.0.255
```

## Step 7: Create DHCP Pool (R1)

```
R1(config)# ip dhcp excluded-address 192.168.50.1 192.168.50.10
R1(config)# ip dhcp pool VLAN50-GUESTS
R1(dhcp-config)# network 192.168.50.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.50.1
R1(dhcp-config)# dns-server 192.168.30.10
R1(dhcp-config)# domain-name itn101.lab
R1(dhcp-config)# exit
```

## Step 8: Save All Devices

```
R1# copy running-config startup-config
R2# copy running-config startup-config
SW-DIST# copy running-config startup-config
SW-ACC# copy running-config startup-config
```

---

## Verification

```
SW-DIST# show vlan brief
```
→ VLAN 50 "GUESTS" should appear.

```
SW-ACC# show interfaces status
```
→ Fa0/13–16 should show VLAN 50.

```
SW-DIST# show interfaces trunk
```
→ VLAN 50 should appear on trunks to R1, R2, and SW-ACC.

```
R1# show standby brief
```
→ Group 50 should show R1 Active, R2 Standby.

Connect a PC to Fa0/13, set to DHCP:
```
PC> ipconfig /renew
PC> ping 192.168.50.1      (gateway)
PC> ping 192.168.10.1      (inter-VLAN)
PC> ping 192.168.30.10     (SRV-1)
```

---

## Documentation Updates

After adding a new VLAN, update:

- [ ] `inventory/ITN101_Network_Inventory.xlsx` — IPAM tab (new subnet row) and port map tabs
- [ ] `diagrams/topology.txt` — Add VLAN to addressing summary
- [ ] `README.md` — IP Addressing table
- [ ] `docs/CHANGELOG.md` — Log the change
- [ ] Golden configs in `configs/golden/` if this becomes part of the baseline
