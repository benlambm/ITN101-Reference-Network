# Factory Reset: Cisco Catalyst 1000 C1000-24T-4G-L (SW-DIST)

**Applies to:** SW-DIST (172.16.0.11)
**IOS:** 15.0(2k) (Catalyst 1000 specific)
**Console:** 9600/8N1, USB-serial or RJ45 rollover to Console port

> **Sources:** [Catalyst 1000 Password Recovery](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst1000/software/releases/15-2_0_7_e/configuration/guide/b_1527e_consolidated_c1000_cg/m_troubleshooting.html) · [Catalyst 1000 Series Getting Started](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst1000/hardware/install/b_c1000_hig/m_troubleshooting.html)

---

## Method A: CLI Reset (If You Have Admin Access) — EASY

| Command | Description |
|---------|-------------|
| `Switch> enable` | Enter privileged EXEC mode. |
| `Switch# delete vlan.dat` | Deletes the VLAN database. Confirm when prompted. |
| `Switch# erase startup-config` | Erases NVRAM. Press Enter to confirm. |
| `Switch# reload` | Type **no** if asked to save config, then confirm reload. |

The switch reboots with no configuration and no VLANs (except default VLAN 1) and enters the Initial Configuration Dialog. Type **no** to skip setup.

> **Note:** You must delete `vlan.dat` separately — `erase startup-config` does **not** remove the VLAN database. If you skip this, the switch reboots with VLANs intact but no port assignments, which can cause confusion.

---

## Method B: Mode Button Recovery + Wipe — MODERATE

Use this when you cannot enter enable mode (password unknown).

**Prerequisites:** Console cable connected, terminal emulator open (9600/8N1). Physical access to the switch.

### Step 1: Power Off the Switch

Disconnect the power cable from the back of the switch.

### Step 2: Hold the Mode Button During Power-On

1. **Press and hold** the **Mode** button on the front panel (left side, below the LEDs).
2. While holding Mode, **reconnect the power cable**.
3. Continue holding for approximately **15 seconds** until:
   - The System LED goes **solid green** briefly, then turns off
   - The LEDs cycle through their startup pattern
4. Release the Mode button.

> **Tip:** The exact LED behavior varies slightly by firmware. The key indicator is that the switch interrupts normal boot and drops to the boot loader prompt. Watch your console — you should see `switch:` appear.

### Step 3: At the Boot Loader Prompt

Rename the existing config so the switch boots clean:

```
switch: rename flash:config.text flash:config.text.old
switch: boot
```

The switch boots with no configuration.

### Step 4: Choose — Recover or Wipe

**To recover the existing config with a new password:**

```
Switch> enable
Switch# rename flash:config.text.old flash:config.text
Switch# copy flash:config.text running-config
Switch# configure terminal
Switch(config)# enable secret <new-password>
Switch(config)# end
Switch# copy running-config startup-config
Switch# reload
```

**To factory-wipe (erase everything):**

```
Switch> enable
Switch# delete flash:config.text.old
Switch# delete vlan.dat
Switch# reload
```

The switch reboots clean with no configuration and no VLAN database.

---

## Method C: If "No Service Password-Recovery" Is Enabled — HARDEST

If the switch was configured with `no service password-recovery`, the Mode button behavior changes.

1. Follow the same Mode button procedure (hold during power-on).
2. Instead of dropping to the boot loader, the switch displays:

   ```
   PASSWORD RECOVERY IS DISABLED.
   Do you want to reset the router to the factory default configuration and proceed [y/n]?
   ```

3. **Type `y`** — startup-config **and** vlan.dat are erased. The switch boots to factory defaults.
4. **Type `n`** — normal boot continues, you remain locked out.

> **Warning:** Choosing `y` permanently destroys the existing configuration and VLAN database. There is no undo.

---

## Catalyst 1000 Hardware Notes

- The **Mode button** is on the front-left of the switch, near the LED mode indicators. It serves double duty: normally it cycles through LED display modes (STAT, SPD, etc.), but during power-on it triggers boot loader / recovery.
- The C1000-24T-4G-L has **24 Gigabit Ethernet ports** and **4 SFP uplink ports** (Gi1/0/25 through Gi1/0/28).
- There is **no USB flash slot** on the Catalyst 1000 — config backup/restore is via TFTP or console paste.
- A factory reset does not affect the IOS image in flash — only startup-config and vlan.dat are wiped.

---

## Post-Reset Checklist

After a factory reset, restore to the ITN 101 baseline:

- [ ] Console in and skip the Initial Configuration Dialog
- [ ] Paste the golden config: `configs/golden/SW-DIST.txt`
- [ ] Verify: `show version` (confirm IOS version)
- [ ] Verify: `show vlan brief` (VLANs 10, 20, 30, 40, 99, 999 present)
- [ ] Verify: `show interfaces trunk` (trunks to R1, R2, and SW-ACC with correct allowed VLANs)
- [ ] Verify: `show spanning-tree summary` (STP root status)
- [ ] Save: `copy running-config startup-config`
