# Factory Reset: Cisco ISR 4221 (R1, R2)

**Applies to:** R1 (172.16.0.1), R2 (172.16.0.2)
**IOS XE:** 16.12.x (universalk9_ias)
**Console:** 9600/8N1, USB-serial or RJ45 rollover to Console port

> **Sources:** [Cisco 4000 Series Password Recovery](https://www.cisco.com/c/en/us/td/docs/routers/access/4400/troubleshooting/guide/isr4400trbl/isr4400trbl02.html) · [Cisco 4000 Series Factory Reset (IOS XE 16.12)](https://www.cisco.com/c/en/us/td/docs/routers/access/4400/software/configuration/xe-16-12/isr4400swcfg-xe-16-12-book/m_perform_factory_reset_isr.html) · [Troubleshoot Password Recovery in IOS XE](https://www.cisco.com/c/en/us/support/docs/ios-nx-os-software/ios-xe-16/217045-troubleshoot-password-recovery-in-cisco.html)

---

## Method A: CLI Reset (If You Have Admin Access) — EASY

| Command | Description |
|---------|-------------|
| `Router> enable` | Enter privileged EXEC mode. |
| `Router# write erase` | Erases startup-config. Confirm when prompted. |
| `Router# reload` | Type **no** if asked to save config, then confirm reload. |

The router reboots with no configuration and enters the Initial Configuration Dialog. Type **no** to skip setup.

> **Note:** IOS XE 16.12+ also supports the `factory-reset all` command, which erases startup-config, running-config, and cleans ROMMON variables. Use this for a more thorough wipe. It runs from privileged EXEC and can take several minutes. **Do not power cycle while it runs.**

---

## Method B: ROMMON Password Recovery + Wipe — MODERATE

Use this when you cannot enter enable mode (password unknown).

**Prerequisites:** Console cable connected, terminal emulator open (9600/8N1).

### Step 1: Enter ROMMON

1. Power cycle the router (or type `reload` if you have any CLI access).
2. During boot, send a **Break** signal:
   - **PuTTY:** Right-click title bar → Special Command → Break
   - **SecureCRT:** Ctrl+Break
   - **macOS Terminal (screen):** Ctrl+A then Ctrl+B
3. You should see the `rommon 1 >` prompt.

### Step 2: Bypass Startup Config

```
rommon 1 > confreg 0x2142
rommon 2 > reset
```

The router reboots but **ignores startup-config** (the config register `0x2142` tells it to skip NVRAM).

### Step 3: Enter Enable Mode

When prompted with the setup dialog, type **no** or press Ctrl+C to skip.

```
Router> enable
```

You now have privilege 15 access with no password required.

### Step 4: Choose — Recover or Wipe

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

**To factory-wipe (erase everything):**

```
Router# write erase
Router# configure terminal
Router(config)# config-register 0x2102
Router(config)# end
Router# reload
```

Type **no** when asked to save. Router reboots clean with normal boot behavior restored.

---

## Method C: If "No Service Password-Recovery" Is Enabled — HARDEST

If the router was configured with `no service password-recovery`, the standard ROMMON break procedure is blocked.

1. During boot, when you send the Break signal, the router displays:

   ```
   PASSWORD RECOVERY IS DISABLED.
   Do you want to reset the router to the factory default configuration and proceed [y/n]?
   ```

2. **Type `y`** — this erases startup-config entirely and boots to factory defaults.
3. **Type `n`** — normal boot continues; you remain locked out.

> **Warning:** If you type `y`, there is no way to recover the old configuration. It is gone. This is by design — the whole point of `no service password-recovery` is to prevent unauthorized config access.

---

## Post-Reset Checklist

After a factory reset, the router has no configuration. To restore to the ITN 101 baseline:

- [ ] Console in and skip the Initial Configuration Dialog
- [ ] Paste the golden config: `configs/golden/R1.txt` or `configs/golden/R2.txt`
- [ ] Verify: `show version` (confirm IOS version and config-register 0x2102)
- [ ] Verify: `show ip interface brief` (all interfaces up)
- [ ] Verify: `show standby brief` (HSRP states correct)
- [ ] Verify: `show ip ospf neighbor` (adjacencies formed)
- [ ] Save: `copy running-config startup-config`
