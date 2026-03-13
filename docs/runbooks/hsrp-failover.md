# Runbook: HSRP Failover Demo & Recovery

**Applies to:** R1, R2
**Estimated time:** 5 minutes (demo), 1 minute (recovery)

> This is one of the most memorable demos in the ITN 101 course.
> Students see redundancy work in real-time with a continuous ping.

---

## HSRP Baseline State

| VLAN | Virtual IP (.1) | Active Router | Standby Router | Active Priority |
|------|----------------|---------------|----------------|-----------------|
| 10 (Students) | 192.168.10.1 | R1 (.2, pri 110) | R2 (.3, pri 100) | 110 |
| 20 (Faculty) | 192.168.20.1 | R2 (.2, pri 110) | R1 (.3, pri 100) | 110 |
| 40 (Wireless) | 192.168.40.1 | R1 (.2, pri 110) | R2 (.3, pri 100) | 110 |

Both routers have `standby <group> preempt` enabled, so the higher-priority router reclaims Active status when it comes back online.

---

## Demo Procedure: VLAN 10 Failover

### Setup

1. On a VLAN 10 PC (PC-A or PC-B), open Command Prompt
2. Start continuous ping to the HSRP virtual gateway:
   ```
   ping -t 192.168.10.1
   ```
3. Confirm pings are succeeding (replies from 192.168.10.1)

### Trigger Failover

4. On R1, console or SSH in:
   ```
   R1# configure terminal
   R1(config)# interface GigabitEthernet0/0/1.10
   R1(config-subif)# shutdown
   R1(config-subif)# end
   ```

### Observe

5. Watch the ping output — **1 to 3 pings will fail** (timeout or "Request timed out")
6. Pings resume automatically as R2 assumes Active role
7. On R2, verify the state change:
   ```
   R2# show standby brief
   ```
   R2 should now show **Active** for Group 10.

### Recovery

8. On R1, re-enable the interface:
   ```
   R1# configure terminal
   R1(config)# interface GigabitEthernet0/0/1.10
   R1(config-subif)# no shutdown
   R1(config-subif)# end
   ```
9. Because preemption is enabled, R1 reclaims Active status (priority 110 > 100)
10. Verify:
    ```
    R1# show standby brief
    ```
    R1 should show **Active** for Group 10 again.

---

## Talking Points for Students

- **Why only 1–3 dropped pings?** HSRP hello/hold timers. Default hold time is 10 seconds, but failover is typically faster in practice.
- **What is preemption?** Without `preempt`, the Standby router would stay Active even after R1 recovers. Preempt forces the higher-priority router to reclaim the role.
- **Why load balance across VLANs?** R1 is Active for VLANs 10 and 40; R2 is Active for VLAN 20. Both routers are always forwarding traffic — no idle standby hardware.
- **What does the client see?** The client only knows its default gateway is `.1`. It never knows which physical router is answering. That's the whole point.

---

## Troubleshooting

### HSRP Not Forming
```
show standby brief
show standby GigabitEthernet0/0/1.10
```
Check: Are both routers trunking to SW-DIST? Is VLAN 10 allowed on the trunk?

### Preemption Not Working
Verify both sides have `standby <group> preempt` configured:
```
show running-config | section standby
```

### Failover Takes Too Long
Default HSRP timers: hello 3s, hold 10s. For faster failover (not in the default golden config):
```
standby 10 timers msec 200 msec 750
```
Use cautiously — aggressive timers can cause flapping on unreliable links.
