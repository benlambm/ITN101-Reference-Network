# Factory Reset: Cisco Catalyst 2960 (SW-ACC)

**Applies to:** SW-ACC (172.16.0.12)
**IOS:** 15.0(2)SE (LAN Base)
**Console:** 9600/8N1, USB-serial or RJ45 rollover to Console port

> **Sources:** [Catalyst 2960 Password Recovery](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst2960/software/release/12-2_55_se/configuration/guide/scg_2960/swtroub.html) · [Recovering from a Corrupted or Missing Image](https://www.cisco.com/c/en/us/support/docs/switches/catalyst-2960-series-switches/71581-2960-recovery.html)

---

## Method A: CLI Reset (If You Have Admin Access) — EASY

| Command | Description |
|---------|-------------|
| `Switch> enable` | Enter privileged EXEC mode. |
| `Switch# delete vlan.dat` | Deletes the VLAN database. Confirm when prompted (press Enter twice). |
| `Switch# erase startup-config` | Erases NVRAM. Press Enter to confirm. |
| `Switch# reload` | Type **no** if asked to save config, then confirm reload. |

The switch reboots with no configuration and no VLANs (except default VLAN 1) and enters the Initial Configuration Dialog. Type **no** to skip setup.

> **Note:** You must delete `vlan.dat` separately — `erase startup-config` does **not** remove the VLAN database. If you skip this, the switch reboots with old VLANs intact but no port assignments.

---

## Method B: Mode Button Recovery + Wipe — MODERATE

Use this when you cannot enter enable mode (password unknown).

**Prerequisites:** Console cable connected, terminal emulator open (9600/8N1). Physical access to the switch.

### Step 1: Power Off the Switch

Disconnect the power cable from the back of the switch.

### Step 2: Hold the Mode Button During Power-On

1. **Press and hold** the **Mode** button on the front panel (left side).
2. While holding Mode, **reconnect the power cable**.
3. Continue holding for approximately **15 seconds** until the System LED turns **amber** and then **solid green**.
4. Release the Mode button.
5. You should see the `switch:` boot loader prompt on the console.

### Step 3: Initialize Flash and Rename Config

```
switch: flash_init
switch: dir flash:
```

Verify you can see `config.text` (and optionally `vlan.dat`) in the directory listing.

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

If the switch was configured with `no service password-recovery`, the Mode button procedure changes.

1. Follow the same Mode button procedure (hold during power-on).
2. After approximately 1–2 seconds, the switch displays on the console:

   ```
   PASSWORD RECOVERY IS DISABLED.
   Do you want to reset the router to the factory default configuration and proceed [y/n]?
   ```

3. **Type `y`** — startup-config **and** vlan.dat are erased. The switch boots to factory defaults.
4. **Type `n`** — normal boot continues, you remain locked out.

> **Warning:** Choosing `y` permanently destroys the existing configuration and VLAN database. There is no undo. You must act quickly — the prompt times out after a few seconds and defaults to normal boot.

---

## Catalyst 2960 Hardware Notes

- The **Mode button** is on the front-left of the switch. Normally it cycles LED display modes (STAT, SPEED, DUPLX, PoE). During power-on it triggers boot loader / recovery.
- The 2960 boot loader requires `flash_init` before you can access the flash filesystem — unlike the Catalyst 1000 which initializes flash automatically.
- A factory reset does not affect the IOS image in flash — only startup-config and vlan.dat are wiped.
- The Catalyst 2960 has a **USB mini-B console port** in addition to the RJ45 console port. Either works for recovery.

---

## Post-Reset Checklist

After a factory reset, restore to the ITN 101 baseline:

- [ ] Console in and skip the Initial Configuration Dialog
- [ ] Paste the golden config: `configs/golden/SW-ACC.txt`
- [ ] Verify: `show version` (confirm IOS version)
- [ ] Verify: `show vlan brief` (VLANs 10, 20, 30, 40, 99, 999 present)
- [ ] Verify: `show interfaces trunk` (trunk to SW-DIST with correct allowed VLANs)
- [ ] Verify: `show interfaces status` (ports assigned to correct VLANs, unused ports in VLAN 999)
- [ ] Save: `copy running-config startup-config`
