# Lab Safety & Physical Access Policy

**ITN 101 Lab Network — Brightpoint Community College**
**Effective:** Spring 2026 | **Owner:** BET Division

---

## Equipment Handling

### Power

- Never unplug or power-cycle equipment without instructor authorization
- The CyberPower PDU (top of rack) controls power to all devices — do not toggle its master switch
- ISR 4221 routers (R1, R2) use external 12VDC power adapters — handle the barrel connector carefully; do not yank by the cord
- ISR 1941 (R3) and both switches use IEC C13 power cords — push in firmly, do not force
- If a device will not power on, check the PDU outlet before assuming hardware failure

### Cabling

- Do not remove or reroute cables without documenting the change
- When connecting a console cable, use the RJ45 console port (light blue) — not the Ethernet ports
- Console connections: 9600 baud, 8 data bits, no parity, 1 stop bit (9600/8N1)
- Label any new cables at both ends (see naming conventions in `policies/naming-conventions.md`)
- If you disconnect a cable for a lab exercise, reconnect it before leaving

### Modules

- **Do not hot-swap modules** — always power off the device first (especially R3's EHWIC-2T)
- The ISR 4221 NIM slots (NIM-1, NIM-2) are empty — do not insert modules without instructor approval
- R3's EHWIC-2T module provides Serial0/0/0 and Serial0/0/1 — these are fragile connectors; do not force smart-serial cables

### General

- No food or drinks on or near the rack
- Do not stack items on top of equipment (airflow matters, even for a small lab)
- If you smell burning or see smoke, power off the PDU immediately and notify the instructor
- Return all tools (console cables, USB drives, cable testers) to their designated location

---

## Console Access

- Shared laptops with USB-serial adapters are provided for console access
- Open PuTTY (or your preferred terminal emulator) and select **Serial** connection type
- Find your COM port: Device Manager → Ports (COM & LPT)
- Settings: 9600/8N1, no flow control
- Do not leave a console session logged in at privilege level 15 when you walk away — type `exit` or `logout`

---

## Configuration Safety

- **Always save before reloading:** `copy running-config startup-config`
- **Know your rollback:** Before making changes, know how to undo them
- **Never erase startup-config on a shared device** without instructor authorization
- If you accidentally break a device's configuration beyond repair, inform the instructor — the golden configs can restore it in minutes

---

## Rack Access

- The rack is an open-frame StarTech.com unit — there is no locking door
- Only access the rack when working on an assigned lab exercise or with instructor permission
- When rack-mounting or removing devices, use proper technique: support the device from below, align with cage nuts / shelf rails
- Tighten thumbscrews by hand — do not use power tools

---

## Emergency Procedures

| Situation | Action |
|-----------|--------|
| Electrical shock | Do not touch the person — disconnect power at the wall outlet, call 911 |
| Burning smell / smoke | Power off PDU, evacuate if necessary, notify instructor |
| Equipment damage | Report to instructor immediately, do not attempt repair |
| Spill on equipment | Power off affected device immediately, clean and dry before powering on |

---

## Reporting Issues

If you notice any of the following, report to the instructor:

- Loose or damaged cables
- Devices that are unusually hot
- Fans making unusual noise or not spinning
- Console ports that do not respond
- LEDs showing error states (amber/red on switches, no lights on routers)
- Missing or displaced equipment
