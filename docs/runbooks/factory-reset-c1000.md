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

## Method B: Mode Button Recovery + Full Wipe

Use this when you cannot enter enable mode (password unknown) or want a complete factory reset.

**Prerequisites:** Console cable connected, terminal emulator open (9600/8N1). Physical access to the switch.

### The Factory Reset Protocol

1. **Establish a Console Session:** Connect to the switch via the console port using your preferred terminal emulator (9600 baud, 8 data bits, no parity, 1 stop bit).
2. **Interrupt the Power:** Unplug the power cable from the switch.
3. **Engage the Mode Button:** Press and hold the Mode button on the front panel of the switch.
4. **Restore Power:** While continuing to hold the Mode button, plug the power cable back in.
5. **Release and Wait:** Keep the button depressed until the SYST LED blinks amber and then turns solid green, or until the `switch:` prompt appears in your terminal window (typically about 10-15 seconds). Release the button.

You are now in the ROMMON/bootloader environment. Execute the following sequence to completely obliterate the existing configuration and VLAN database:

```
switch: flash_init
switch: delete flash:config.text
switch: delete flash:vlan.dat
switch: boot
```

> **Note:** Press `y` or confirm if the system prompts you to verify the deletions.

### Post-Boot Cleanup

Once the switch finishes its boot sequence, it will present the System Configuration Dialog.

1. Type `no` to bypass the setup wizard.
2. Enter privileged EXEC mode.
3. Save the currently empty running configuration to the startup configuration. This finalizes the reset.

```
Switch> enable
Switch# copy running-config startup-config
```

> **Recommendation:** Configure a new `enable secret` immediately after reset.

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
