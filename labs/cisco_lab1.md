# Cisco Switch: Setup, Config & Discovery

**Instructions:** Connect to the switch via the provided console cable. Enter each command in sequence. Commands in the `monospace` font should be typed exactly as shown. Pay close attention to the prompt (e.g., `Switch>` vs `Switch#`) to ensure you are in the correct mode.

## Phase 1: Hardware Setup & Connection

### Step 1: Gather Your Equipment
* Cisco 2960 switch (powered on — listen for fans, look for lights)
* USB console cable (light blue, RJ-45 to USB-A)
* Ethernet cable (RJ45 Straight-thru)
* Windows laptop with PuTTY app (install from Microsoft Store or download from putty.org)

### Step 2: OPTIONAL Factory Reset (Only Needed If Password Protected)

**Method A: CLI Reset (If you have Admin Access) = EASY**
| **Command** | **Description** |
| :--- | :--- |
| `Switch> enable` | Enter privileged EXEC mode. |
| `Switch# delete vlan.dat` | Press Enter twice to confirm. |
| `Switch# erase startup-config` | Press Enter to confirm. |
| `Switch# reload` | Type "no" if asked to save the initial config, then confirm reload. |

**Method B: Hardware Reset = EASY IF IT WORKS!**
1. Power on the switch.
2. Immediately press and hold the **MODE** button for ~3 seconds until LEDs start blinking.
3. Keep holding for ~7 more seconds until LEDs stop blinking.
4. Release after they stop blinking—the switch should reboot to its factory default state.
5. Connect to the switch via console cable and open a PuTTY session (Serial, COM port, 9600 baud). If you see `Switch>` and the Initial Config dialog, it worked! Enter `no`.

**Method C: Hardware Reset = MORE DIFFICULT**
1. Requires connection to the switch via console cable and open a PuTTY session (Serial, COM port, 9600 baud). See Instructions below in Step 3!
2. Unplug the switch, hold the **MODE** button, then plug it back in — keep holding until you see the `switch:` prompt on the console.
3. Type `flash_init` and press Enter. *(Note: If this returns an error, skip to the next command).*
4. Type `delete flash:config.text` and press Enter, then `delete flash:vlan.dat` and press Enter.
5. Type `boot` and press Enter — the switch reboots to factory defaults.

### Step 3: Find Your COM Port
1. Plug the USB console cable into your laptop.
2. Open Device Manager: Press `Win + X` → select **Device Manager**.
3. Watch for 1-2 minutes as Windows Updates automatically installs the correct Driver (this step requires a working Internet connection!).
4. Expand **Ports (COM & LPT)**. Look for **USB Serial Port (COM#)** — note your COM number.
5. *If no COM port appears:* unplug, replug, and recheck. Older console cables with Prolific chipsets may require legacy drivers.

### Step 4: Connect with PuTTY
1. Connect the RJ-45 end of the console cable to the switch's CONSOLE port (on the back).
2. Open PuTTY and set the following:
   * **Connection type:** Serial
   * **Serial line:** COM# (your port number from Step 3)
   * **Speed:** 9600
3. Click **Open**. Press Enter once or twice until you see the `Switch>` prompt.

## Phase 2: Initial System Discovery

### Step 5: Enter Privileged Mode & Set the Clock

| **Command** | **Description** |
| :--- | :--- |
| `Switch> enable` | Enter privileged EXEC mode (Admin). |
| `Switch# clock set 09:30:00 8 Oct 2026` | Set the date & time (Use the current time and date!). |

### Step 6: Pre-Connection Discovery

**1. Command: `show version`**
* **Question:** What is the exact version number of the Cisco IOS software release running on this switch? What is the Cisco model number given and how many ports does it have?
* **Answer:** Version: \_________________   Model & ports: \______________

**2. Command: `show clock`**
* **Question:** What time does the switch currently think it is?
* **Answer:** \______________

**3. Command: `show ip interface brief`**
* **Question:** Look at the "Status" and "Protocol" columns for your FastEthernet or GigabitEthernet ports. What do they currently say?
* **Answer:** Status: \_________________   Protocol: \______________

**4. Command: `show interfaces status`**
* **Question:** Pick one port from the list. What does the "Status" column say for that specific port? What VLAN is it assigned to? What is its speed & type?
* **Answer:** Status/VLAN: \_________________   Speed & type: \______________

**5. Command: `show mac address-table`**
* **Question:** Are there any DYNAMIC MAC addresses currently listed in the table? (Yes / No – if all are STATIC). What is the total number of MAC addresses given?
* **Answer:** DYNAMIC present?: \_________________   Total number: \______________

## Phase 3: Basic Configuration

### Step 7: Global Configuration & Basics

| **Command** | **Description** |
| :--- | :--- |
| `Switch# configure terminal` | Enters global configuration mode. |
| `Switch(config)# no ip domain-lookup` | Disables DNS lookups so mistyped commands don't freeze the console. |
| `Switch(config)# hostname S1` | Sets the switch hostname (Watch your prompt change immediately!). |
| `S1(config)# banner motd # Authorized Access Only! #` | Sets a Message of The Day banner. |
| `S1(config)# enable secret class` | Sets the admin (enable) password to "class". |

### Step 8: Console & Remote Access Passwords

| **Command** | **Description** |
| :--- | :--- |
| `S1(config)# line console 0` | Enter configuration for the physical Console port. |
| `S1(config-line)# logging synchronous` | Prevents system messages from interrupting your typing. |
| `S1(config-line)# password cisco` | Set console password to "cisco". |
| `S1(config-line)# login` | Require password checking at console login. |
| `S1(config-line)# exit` | Return to global config. |
| `S1(config)# line vty 0 15` | Enter VTY line configuration for remote network access (lines 0-15). |
| `S1(config-line)# password cisco` | Sets Telnet remote password to "cisco". |
| `S1(config-line)# login` | Requires password checking for remote logins. |
| `S1(config-line)# transport input telnet` | Allows telnet connections to the switch. |
| `S1(config-line)# exit` | Return to global config. |

### Step 9: Set Virtual Management IP
*To manage the switch remotely over the network, it needs an IP address assigned to its management VLAN.*

| **Command** | **Description** |
| :--- | :--- |
| `S1(config)# interface vlan 1` | Enter VLAN 1 interface config. |
| `S1(config-if)# description Network Management` | Label the interface. |
| `S1(config-if)# ip address 192.168.1.2 255.255.255.0` | Assign a virtual management IP address. |
| `S1(config-if)# no shutdown` | Turn the interface on. |
| `S1(config-if)# exit` | Return to global config. |
| `S1(config)# ip default-gateway 192.168.1.1` | Set the default gateway (router IP) for the switch. |

### Step 10: Exit & Save Configuration

| **Command** | **Description** |
| :--- | :--- |
| `S1(config)# end` | Jump all the way back to privileged EXEC mode. |
| `S1# copy running-config startup-config` | Save your current configuration to NVRAM so it survives a reboot. Press Enter to confirm. |

## Phase 4: Post-Connection Discovery & Testing

### Step 11: Connect Ethernet & Review Changes
*Action: Plug your Ethernet cable into any of the switch's front FastEthernet or GigabitEthernet ports, and connect the other end to your PC.*

**6. Command: `show ip interface brief`**
* **Question:** Find the port you just plugged into. What do the "Status" and "Protocol" columns say now?
* **Answer:** Status: \_________________   Protocol: \______________

**7. Command: `show interfaces status`**
* **Question:** Look at the port you connected to. What is its current Status, and what is its listed Speed?
* **Answer:** Status: \_________________   Speed: \______________

**8. Command: `show mac address-table`**
* **Question:** You should now see at least one entry. What is the MAC address listed for the port you are plugged into?
* **Answer:** \______________

**9. Command: `show running-config`**
* **Question:** Press the Spacebar to scroll through the output. What is the "hostname" listed near the very top of the configuration?
* **Answer:** \______________

### Step 12: Verification & Telnet Testing
*If necessary, assign your PC a static IP of 192.168.1.10 to be on the same network.*

| **Command** | **Description** |
| :--- | :--- |
| `C:\> ping 192.168.1.2` | Open Command Prompt on your PC and try to ping the switch now. |
| `C:\> telnet 192.168.1.2` | Open a new window in PuTTY on your PC and try to remote into the switch using Telnet. |
| `S1# show users` | If successful, run this on the switch to see your active Telnet session! |

**10. Ping Test from PC**
* **Question:** Were you able to successfully ping the switch's virtual management IP?
* **Answer:** \______________

**11. Bonus Question**
* **Question:** What happens if you ping your PC from the `S1#` prompt on the Switch? Why does it fail?
* **Answer:** \_________________________________________________________

***

## Important Lab Notes

* **Intentionally Insecure:** All passwords configured today are transmitted in plaintext over the network. This is for educational purposes only. Relying on Telnet in a modern production environment is the security equivalent of leaving your car running with the doors open in a bad neighborhood. 
* **Bonus Challenge:** Run Wireshark on your PC while connecting via Telnet. See if you can capture and read your plaintext `cisco` and `class` passwords in the packet capture!