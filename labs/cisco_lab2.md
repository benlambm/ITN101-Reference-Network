# Lab 2: Catalyst 1000 Inter-VLAN Routing & Secure Shell (SSH)

**Instructions:** In this lab, you will configure a Cisco Catalyst 1000 switch as your distribution layer router (**SW-DIST**) and connect it to your access switch from Lab 1 (**S1**). You will configure SSH for secure management, establish a VLAN trunk, and enable IP routing to allow communication between isolated VLANs.

> **Pre-Lab Note: The TFTP "Socket Error" Spam**
> Upon booting a factory-reset switch, your console screen will likely be flooded with messages like `%Error opening tftp://255.255.255.255/network-confg (Socket error)`. **Do not panic.** > Because the switch woke up with no saved memory, it is using a feature called AutoInstall to desperately shout into the network, begging a nonexistent TFTP server to give it a configuration file. Simply press **Enter** to get your prompt back, type `no` when it asks to enter the initial configuration dialog, and proceed. Once you save your configuration at the end of this lab, it will stop doing this on future reboots.

## Phase 1: Base Configuration & Secure Access (SW-DIST)

*Note: Connect your console cable directly to the Catalyst 1000 for this phase.*

### Step 1: Global Configuration & Hostname

| **Command** | **Description** | 
| `Switch> enable` | Enter privileged EXEC mode. | 
| `Switch# configure terminal` | Enter global configuration mode. | 
| `Switch(config)# hostname SW-DIST` | Sets the distribution switch hostname. | 
| `SW-DIST(config)# no ip domain-lookup` | Disables DNS timeouts for mistyped commands. | 

### Step 2: Console & SSH Configuration

*First, we will fix the console so those annoying system logs stop interrupting your typing. Then, we secure our VTY lines using SSH, which requires a domain name and cryptographic keys.*

| **Command** | **Description** | 
| `SW-DIST(config)# line console 0` | Enter physical console line configuration. | 
| `SW-DIST(config-line)# logging synchronous` | **Crucial:** Prevents system messages from interrupting your typing. | 
| `SW-DIST(config-line)# exit` | Return to global config. | 
| `SW-DIST(config)# ip domain-name ccna.lab` | Required prerequisite for generating RSA keys. | 
| `SW-DIST(config)# crypto key generate rsa` | Generates encryption keys. **When prompted for modulus size, type `2048` and press Enter.** | 
| `SW-DIST(config)# ip ssh version 2` | Forces the switch to use the more secure SSHv2. | 
| `SW-DIST(config)# username admin secret class` | Creates a local user database entry for login. | 
| `SW-DIST(config)# line vty 0 15` | Enters configuration for remote virtual terminals. | 
| `SW-DIST(config-line)# login local` | Tells the switch to use the local username/password database we just created. | 
| `SW-DIST(config-line)# transport input ssh` | **Crucial:** Allows only SSH. Telnet is now blocked. | 
| `SW-DIST(config-line)# exit` | Return to global config. | 

## Phase 2: Layer 3 Inter-VLAN Routing (SW-DIST)

### Step 3: Enable Routing and Create SVIs (Switch Virtual Interfaces)

| **Command** | **Description** | 
| `SW-DIST(config)# ip routing` | **Crucial:** Wakes up the routing brain of the switch. | 
| `SW-DIST(config)# vlan 10` | Creates VLAN 10 in the database. | 
| `SW-DIST(config-vlan)# name Students` | Names the VLAN. | 
| `SW-DIST(config-vlan)# vlan 20` | Creates VLAN 20 in the database. | 
| `SW-DIST(config-vlan)# name Staff` | Names the VLAN. | 
| `SW-DIST(config-vlan)# exit` | Return to global config. | 
| `SW-DIST(config)# interface vlan 1` | Enters SVI for VLAN 1 (Management). | 
| `SW-DIST(config-if)# ip address 192.168.1.1 255.255.255.0` | Sets the gateway IP for Lab 1's network. | 
| `SW-DIST(config-if)# no shutdown` | Activates the interface. | 
| `SW-DIST(config-if)# interface vlan 10` | Enters SVI for VLAN 10. | 
| `SW-DIST(config-if)# ip address 192.168.10.1 255.255.255.0` | Sets the gateway IP for VLAN 10. | 
| `SW-DIST(config-if)# no shutdown` | Activates the interface. | 
| `SW-DIST(config-if)# interface vlan 20` | Enters SVI for VLAN 20. | 
| `SW-DIST(config-if)# ip address 192.168.20.1 255.255.255.0` | Sets the gateway IP for VLAN 20. | 
| `SW-DIST(config-if)# no shutdown` | Activates the interface. | 

### Step 4: Configure the 802.1Q Trunk Uplink

*Assume GigabitEthernet1/0/24 is the port connecting down to S1. Verify your physical port.*

| **Command** | **Description** | 
| `SW-DIST(config)# interface GigabitEthernet1/0/24` | Select the port connecting to S1. | 
| `SW-DIST(config-if)# switchport mode trunk` | Forces the port to carry multiple VLANs. | 
| `SW-DIST(config-if)# switchport nonegotiate` | Disables DTP to prevent VLAN hopping attacks. | 
| `SW-DIST(config-if)# end` | Return to privileged EXEC mode. | 
| `SW-DIST# copy running-config startup-config` | Save your configuration. | 

## Phase 3: Access Layer Segmentation (S1 - C2960)

*Action: Move your console cable back to S1 (the 2960 from Lab 1).*

### Step 5: Configure S1 Trunk and Access Ports

| **Command** | **Description** | 
| `S1> enable` | Enter privileged EXEC mode. | 
| `S1# configure terminal` | Enter global configuration. | 
| `S1(config)# vlan 10` | Creates VLAN 10 on S1. | 
| `S1(config-vlan)# name Students` | Names the VLAN. | 
| `S1(config-vlan)# vlan 20` | Creates VLAN 20 on S1. | 
| `S1(config-vlan)# name Staff` | Names the VLAN. | 
| `S1(config-vlan)# exit` | Return to global config. | 
| `S1(config)# interface GigabitEthernet0/1` | Select the uplink port connected to SW-DIST. | 
| `S1(config-if)# switchport mode trunk` | Matches the trunk configuration from SW-DIST. | 
| `S1(config-if)# switchport nonegotiate` | Disables DTP negotiation. | 
| `S1(config-if)# interface FastEthernet0/1` | Select a port for PC 1. | 
| `S1(config-if)# switchport mode access` | Hardcodes the port as an endpoint port. | 
| `S1(config-if)# switchport access vlan 10` | Assigns this port to the Students VLAN. | 
| `S1(config-if)# interface FastEthernet0/2` | Select another port for PC 2. | 
| `S1(config-if)# switchport mode access` | Hardcodes the port as an endpoint port. | 
| `S1(config-if)# switchport access vlan 20` | Assigns this port to the Staff VLAN. | 
| `S1(config-if)# end` | Return to privileged EXEC mode. | 
| `S1# copy running-config startup-config` | Save your configuration. | 

## Phase 4: Post-Connection Discovery & Testing

**1. Command: `show ip route` (Run on SW-DIST)**

* **Question:** Look at the routing table. What does the "C" next to the routes stand for, and how many "C" routes do you see?

* **Answer:** Code "C" means: \___\_**\_   Number of routes: \_**\_

**2. Command: `show interfaces trunk` (Run on both switches)**

* **Question:** What ports are actively trunking, and what VLANs are allowed across the trunk?

* **Answer:** Port: \___\_**\_  Allowed VLANs: \_**\_

**3. Ping Test: Inter-Switch Communication**

* **Action:** From the `SW-DIST#` prompt, type `ping 192.168.1.2`

* **Question:** Were you able to successfully ping S1's management IP from SW-DIST?

* **Answer:** \________________\_

**4. Ping Test: The Layer 2 Isolation (Failure Demo)**

* **Action:** Physically unplug the trunk cable connecting S1 to SW-DIST (or shut down S1's GigabitEthernet0/1 port). Connect PC 1 to FastEthernet0/1 (VLAN 10) and assign it a static IP of `192.168.10.10` / Gateway `192.168.10.1`. Connect PC 2 to FastEthernet0/2 (VLAN 20) and assign it a static IP of `192.168.20.10` / Gateway `192.168.20.1`. Open the command prompt on PC 1 and ping PC 2 (`ping 192.168.20.10`).

* **Question:** Does the ping succeed? Why is a Layer 2 switch completely incapable of moving packets between these two isolated broadcast domains on its own?

* **Answer:** \___________________________________________________________________\_

**5. Ping Test: Inter-VLAN Routing (Success Demo)**

* **Action:** Plug the trunk cable back in (or `no shutdown` the port). Wait 30 seconds for Spanning Tree Protocol to turn the port link light from amber to green. Attempt the same ping from PC 1 to PC 2.

* **Question:** Does it succeed now? Briefly explain how the ICMP packet traveled from PC 1 to PC 2. What device had to forward it?

* **Answer:** \___________________________________________________________________\_

**6. SSH Verification**

* **Action:** From PC 1, open PuTTY. Choose SSH, enter `192.168.10.1` (the SW-DIST gateway), and click Open.

* **Question:** Were you prompted for a username and password? Did the `admin` / `class` credentials work to log in remotely?

* **Answer:** \________________\_

## Appendix: Inter-VLAN ACLs (The "Keep the Rabble Out" Protocol)

*By default, creating Switch Virtual Interfaces (SVIs) allows your router to blindly pass traffic between all connected VLANs. If you have a VLAN 30 for `Servers` (192.168.30.0/24), you likely do not want the `Students` VLAN (192.168.10.0/24) accessing it freely.*

*To prevent this, we use an Access Control List (ACL) applied directly to the SVI to act as a bouncer at the door of the router.*

### Step A1: Create the Extended ACL

| **Command** | **Description** | 
| `SW-DIST(config)# ip access-list extended DENY_STUDENTS_TO_SERVERS` | Creates a named, extended ACL and enters its configuration mode. | 
| `SW-DIST(config-ext-nacl)# deny ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255` | Explicitly drops any traffic originating from the Student subnet destined for the Server subnet. | 
| `SW-DIST(config-ext-nacl)# permit ip any any` | **Crucial:** Allows all other traffic to pass (otherwise the implicit deny drops their internet traffic too). | 
| `SW-DIST(config-ext-nacl)# exit` | Return to global config. | 

### Step A2: Apply the ACL to the Interface

| **Command** | **Description** | 
| `SW-DIST(config)# interface vlan 10` | Enter the virtual interface for the Students VLAN. | 
| `SW-DIST(config-if)# ip access-group DENY_STUDENTS_TO_SERVERS in` | Applies the ACL *inbound*, forcing the switch to check Student packets as they enter the router. | 
| `SW-DIST(config-if)# end` | Return to privileged EXEC mode. | 

*Result: When a student tries to ping or SSH into a server, the switch drops the packet at the threshold and returns an ICMP "Destination Unreachable" message to the student's PC.*