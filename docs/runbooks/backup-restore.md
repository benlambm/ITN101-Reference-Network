# Runbook: Configuration Backup & Restore

**Applies to:** All devices (R1, R2, R3, SW-DIST, SW-ACC)
**Estimated time:** 5 minutes per device

---

## Method 1: Copy-Paste via Console (Simplest)

### Backup

1. Console into the device (9600/8N1)
2. Enter privileged exec mode: `enable`
3. Set terminal length to show full output:
   ```
   terminal length 0
   ```
4. Display the running config:
   ```
   show running-config
   ```
5. Select all output in your terminal emulator and copy to a text file
6. Save as `<hostname>_backup_<date>.txt`
7. Reset terminal length:
   ```
   terminal length 24
   ```

### Restore

1. Console into the device
2. Erase existing configuration:
   ```
   erase startup-config
   ```
   For switches, also delete the VLAN database:
   ```
   delete flash:vlan.dat
   ```
3. Reload the device:
   ```
   reload
   ```
4. After reboot, enter config mode and paste the saved config
5. Save:
   ```
   copy running-config startup-config
   ```

---

## Method 2: TFTP Backup (If TFTP Server Available)

### Prerequisites
- TFTP server running on a reachable host (e.g., SRV-1 at 192.168.30.10)
- IP connectivity between the device and the TFTP server

### Backup

```
R1# copy running-config tftp:
Address or name of remote host? 192.168.30.10
Destination filename [r1-confg]? R1_backup_2026-03-13.txt
```

### Restore

```
R1# copy tftp: running-config
Address or name of remote host? 192.168.30.10
Source filename? R1_backup_2026-03-13.txt
```
Then save:
```
R1# copy running-config startup-config
```

---

## Method 3: USB Backup (Routers Only)

### Backup

```
R1# copy running-config usb0:R1_backup.txt
```

### Restore

```
R1# copy usb0:R1_backup.txt running-config
R1# copy running-config startup-config
```

---

## Restore from Golden Configs

The golden configs in this repository (`configs/golden/`) represent the baseline known-good state.

**Restore order matters.** Follow this sequence:

1. **R1** → `configs/golden/R1.txt`
2. **R2** → `configs/golden/R2.txt`
3. **SW-DIST** → `configs/golden/SW-DIST.txt`
4. **SW-ACC** → `configs/golden/SW-ACC.txt`
5. **R3** → `configs/golden/R3.txt`

### Why This Order?

- R1 establishes DHCP, NTP master, NAT, and the default OSPF route
- R2 needs R1 up to form HSRP pairs and OSPF adjacency
- Switches need VLANs created before endpoints can communicate
- R3 depends on OSPF adjacency with R2 to learn routes

---

## Post-Restore Verification

Run on each device after restoring:

```
show running-config | include hostname
show ip interface brief
show standby brief          (R1, R2 only)
show ip ospf neighbor       (R1, R2, R3)
show vlan brief             (SW-DIST, SW-ACC)
show interfaces trunk       (SW-DIST, SW-ACC)
show ntp associations
```

---

## Backup Schedule Recommendation

For a classroom lab, back up configs:

- After initial deployment (the golden configs serve this purpose)
- Before any lab exercise that modifies configurations
- After any IOS upgrade
- At the end of each semester before equipment is powered down
