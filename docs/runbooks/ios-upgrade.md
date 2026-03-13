# Runbook: IOS Upgrade via USB (Cisco ISR 4221)

**Applies to:** R1, R2 (ISR 4221)
**Last tested:** Spring 2026
**Estimated time:** 15–20 minutes per router

> This procedure was used to upgrade R1 and R2 from IOS XE 16.10.3a to 16.12.x.
> Adapted from the laminated lab procedure posted in the Brightpoint networking lab.

---

## Prerequisites

- USB flash drive with the new IOS image (e.g., `isr4200-universalk9_ias.16.12.05.SPA.bin`)
- Console cable (USB-serial or RJ45 rollover) connected to the router
- Terminal emulator (PuTTY, SecureCRT, etc.) set to **Serial**, correct COM port, **9600/8N1**
- Physical access to the router's USB port (rear panel)

## Finding Your COM Port (Windows)

1. Right-click the Windows icon → **Device Manager**
2. Expand **Ports (COM & LPT)**
3. Note the COM port number for your USB-serial adapter

---

## Procedure

### Step 1: Enter ROMMON

1. Connect USB drive to the rear USB port on the router
2. Open terminal session (Serial, correct COM port)
3. Power on (or reload) the router
4. Interrupt boot with **Ctrl+C** or right-click terminal header → **Special Command** → **Break**
5. You should see the `rommon 1 >` prompt

### Step 2: Boot from USB

```
rommon 1 > dir usb0:
```
Verify the new IOS file is listed.

```
rommon 2 > boot usb0:isr4200-universalk9_ias.16.12.05.SPA.bin
```
Wait for the router to boot fully into configuration mode.

### Step 3: Remove Old IOS from Bootflash

```
Router# dir bootflash:
```
Identify the OLD image file (e.g., `ISR4200-ucmk9.16.10.3a.SPA.bin`).

```
Router# delete bootflash:ISR4200-ucmk9.16.10.3a.SPA.bin
```
Confirm deletion when prompted.

### Step 4: Copy New IOS to Bootflash

```
Router# dir usb0:
```
Note the exact filename of the new image.

```
Router# copy usb0: bootflash:
```
When prompted for the source filename, enter:
```
isr4200-universalk9_ias.16.12.05.SPA.bin
```

> **Note:** If the copy fails or hangs, you may need to power off, enter ROMMON, and boot from USB again.

### Step 5: Set Boot Variable

```
Router# configure terminal
Router(config)# no boot system
Router(config)# boot system flash bootflash:isr4200-universalk9_ias.16.12.05.SPA.bin
Router(config)# config-register 0x2102
Router(config)# end
Router# write memory
```

### Step 6: Verify and Reload

```
Router# show version
```
Confirm the new IOS version is listed as the system image.

```
Router# show boot
```
Confirm the boot variable points to the new image.

1. Remove the USB drive
2. Reload the router: `reload`
3. After reboot, verify again with `show version`

---

## Rollback

If the new IOS fails to boot:
1. Enter ROMMON (interrupt boot sequence)
2. If old image is still on bootflash: `boot bootflash:<old_image>.bin`
3. If old image was deleted: boot from USB and copy a known-good image to bootflash

---

## Post-Upgrade Checklist

- [ ] `show version` confirms new IOS version
- [ ] `show ip interface brief` — all interfaces up as expected
- [ ] `show standby brief` — HSRP states correct (if applicable)
- [ ] `show ip ospf neighbor` — neighbor adjacencies reformed
- [ ] `show ip ssh` — SSH version 2 operational
- [ ] `show ntp status` — NTP synced (master or client)
- [ ] Re-apply golden config if startup-config was lost during ROMMON operations
- [ ] `copy running-config startup-config`
