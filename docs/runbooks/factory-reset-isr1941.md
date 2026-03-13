# Factory Reset: Cisco ISR 1941 (R3)

**Applies to:** R3 (172.16.0.3)
**IOS:** 15.x (classic IOS, not IOS XE)
**Console:** 9600/8N1, USB-serial or RJ45 rollover to Console port
**Module:** EHWIC-2T in EHWIC slot 0 (Serial0/0/0, Serial0/0/1)

> **Sources:** [Cisco 1900 Series Password Recovery](https://www.cisco.com/c/en/us/support/docs/routers/3800-series-integrated-services-routers/112058-c1900-pwd-rec-00.html) · [Troubleshoot Password Recovery in IOS Routers](https://www.cisco.com/c/en/us/support/docs/ios-nx-os-software/ios-xe-16/217045-troubleshoot-password-recovery-in-cisco.html)

---

## Method A: CLI Reset (If You Have Admin Access) — EASY

| Command | Description |
|---------|-------------|
| `Router> enable` | Enter privileged EXEC mode. |
| `Router# erase startup-config` | Erases NVRAM. Press Enter to confirm. |
| `Router# reload` | Type **no** if asked to save config, then confirm reload. |

The router reboots with no configuration and enters the Initial Configuration Dialog. Type **no** to skip setup.

> **Note:** The ISR 1941 runs classic IOS, not IOS XE. There is no `factory-reset` command and no `write erase` alias. Use `erase startup-config`.

---

## Method B: ROMMON Password Recovery + Wipe — MODERATE

Use this when you cannot enter enable mode (password unknown).

**Prerequisites:** Console cable connected, terminal emulator open (9600/8N1).

### Step 1: Record the Config Register (If Accessible)

If you have any level of access:

```
Router> show version
```

Note the **Configuration register** value at the bottom of the output (typically `0x2102`).

### Step 2: Enter ROMMON

1. Power cycle the router.
2. During boot, after you see the "program load complete" message, send a **Break** signal:
   - **PuTTY:** Right-click title bar → Special Command → Break
   - **SecureCRT:** Ctrl+Break
   - **macOS Terminal (screen):** Ctrl+A then Ctrl+B
   - Send it **two or three times** rapidly if the first one does not catch.
3. You should see the `rommon 1 >` prompt.

> **Tip:** If Break does not work, try: power off the router, remove the compact flash card from CF 0, power on (router enters ROMMON because there is no bootable image), reinsert the CF card, then proceed with the commands below.

### Step 3: Bypass Startup Config

```
rommon 1 > confreg 0x2142
rommon 2 > reset
```

The router reboots but **ignores startup-config**.

### Step 4: Enter Enable Mode

When prompted with the setup dialog, type **no** or press Ctrl+C.

```
Router> enable
```

No password is required (startup-config was skipped).

### Step 5: Choose — Recover or Wipe

**To recover the existing config with a new password:**

```
Router# copy startup-config running-config
Router# configure terminal
Router(config)# enable secret <new-password>
Router(config)# config-register 0x2102
Router(config)# end
Router# copy running-config startup-config
Router# reload
```

> **Important:** The `enable secret` is a one-way hash — you cannot read the old one. You must replace it. The `enable password` (if used instead) may be visible in the config in cleartext.

**To factory-wipe (erase everything):**

```
Router# erase startup-config
Router# configure terminal
Router(config)# config-register 0x2102
Router(config)# end
Router# reload
```

Type **no** when asked to save. Router reboots clean.

---

## Method C: If "No Service Password-Recovery" Is Enabled — HARDEST

If the router was configured with `no service password-recovery`:

1. During boot, when you send the Break signal, the router displays:

   ```
   PASSWORD RECOVERY IS DISABLED.
   Do you want to reset the router to the factory default configuration and proceed [y/n]?
   ```

2. **Type `y`** — startup-config is erased, router boots to factory defaults.
3. **Type `n`** — normal boot continues, you remain locked out.

> **Warning:** Choosing `y` permanently destroys the existing configuration. There is no undo.

---

## ISR 1941 Hardware Notes

- The **EHWIC-2T module** (Serial ports) survives a factory reset — it is hardware, not config. The serial interfaces will appear as `Serial0/0/0` and `Serial0/0/1` after reset but will have no configuration.
- The **CF 0 slot** (compact flash) contains the IOS image. A config reset does not erase the IOS — only startup-config is wiped.
- The ISR 1941 has **no physical reset button**. All resets require console access or power cycling.

---

## Post-Reset Checklist

After a factory reset, restore to the ITN 101 baseline:

- [ ] Console in and skip the Initial Configuration Dialog
- [ ] Paste the golden config: `configs/golden/R3.txt`
- [ ] Verify: `show version` (confirm IOS version and config-register 0x2102)
- [ ] Verify: `show ip interface brief` (G0/0 and G0/1 up, serials shutdown)
- [ ] Verify: `show ip ospf neighbor` (R2 adjacency formed)
- [ ] Verify: `show ip dhcp binding` (VLAN 30 pool active)
- [ ] Save: `copy running-config startup-config`
