# OPNsense IDS/IPS Lab Documentation

## Overview

This lab was built in Hyper-V using OPNsense, Windows Server, and Windows 11 to test basic firewall protection, intrusion detection, and intrusion prevention.

The goal was to understand the difference between:

- firewall filtering
- IDS alerting
- IPS blocking

The lab also highlighted an important real-world detail: traffic between devices in the same LAN does not always behave as expected from a firewall or IDS/IPS testing perspective.

---

## Lab Environment

### Virtual Machines
- **OPNsense-VM**
- **Server2025-VM**
- **Win11-VM**
- **Win11-LAN2** (already available in the environment)

### Interfaces
- **WAN** -> Default Switch
- **LAN** -> LAN-Switch
- **LAN2** -> LAN2

### IP Addresses
- **OPNsense LAN:** 192.168.10.1/24
- **Windows Server:** 192.168.10.145
- **Windows 11:** 192.168.10.144

---

## Objective

The main tasks in the lab were:

1. Prepare the environment
2. Configure firewall rules
3. Enable and test IDS
4. Enable and test IPS
5. Analyze the result

---

## Step 1 - Prepare the Environment

The lab environment was first cleaned up because older VLAN configuration from a previous task was still present.

### Cleanup performed
- Removed old VLAN interfaces from OPNsense assignments
- Removed old VLAN tagging from Windows VMs in Hyper-V
- Removed trunk mode from the OPNsense LAN adapter
- Returned the lab to a normal untagged LAN setup

### Validation
- OPNsense LAN confirmed as `192.168.10.1/24`
- Windows Server received `192.168.10.145`
- Windows 11 received `192.168.10.144`
- Ping between systems worked

---

## Step 2 - Configure Firewall Rules

The next step was to configure simple LAN firewall rules in OPNsense.

### Rule 1 - Allow LAN to Server
- **Action:** Pass
- **Interface:** LAN
- **Direction:** In
- **Source:** LAN net
- **Destination:** 192.168.10.145
- **Description:** Allow LAN to Server

### Rule 2 - Block Server to LAN
- **Action:** Block
- **Interface:** LAN
- **Direction:** In
- **Source:** 192.168.10.145
- **Destination:** LAN net
- **Description:** Block Server to LAN

### Result
The rules were created successfully, but testing showed that traffic between the server and client still worked both ways.

### Why this happened
Both systems were in the same subnet (`192.168.10.0/24`), so the traffic did not pass through OPNsense in the way expected for filtering between hosts on the same virtual switch.

---

## Step 3 - Enable and Test IDS

In OPNsense 26.1.5, Intrusion Detection was available under:

- **Services -> Intrusion Detection**
  - Administration
  - Policy
  - Log File

### IDS setup
- Enabled Intrusion Detection
- Selected **LAN**
- Downloaded and applied rules
- Verified Suricata started correctly
- Created a user defined test rule

### User defined IDS rule
A test rule was created to generate an alert for traffic from the client to OPNsense.

- **Enabled:** Yes
- **Source IP:** 192.168.10.144/32
- **Destination IP:** 192.168.10.1/32
- **Action:** Alert
- **Description:** Test ICMP to OPNsense

### IDS test
From the Windows 11 client, ICMP traffic was sent to OPNsense.

```cmd
ping 192.168.10.1 -t
```

### Result
This successfully generated an alert in Intrusion Detection.

### Key observation
Simple ping between the client and the server in the same LAN did not trigger useful alerts. Testing directly against OPNsense was a better way to confirm that IDS was working.

---

## Step 4 - Enable and Test IPS

After confirming IDS worked, the next step was to test IPS.

### IPS setup
- Changed **Capture mode** to **Netmap (IPS)**
- Kept the LAN interface selected
- Changed the user defined rule action from **Alert** to **Drop**

### Updated test rule
- **Source IP:** 192.168.10.144/32
- **Destination IP:** 192.168.10.1/32
- **Action:** Drop
- **Description:** Test ICMP to OPNsense

### Initial result
At first, ping to OPNsense still worked even though IPS mode and Drop were enabled.

### Hyper-V / Netmap fix
To make IPS work properly in the virtualized environment, the following tunable was added:

- **Tunable:** `dev.netmap.admode`
- **Value:** `2`

This forced Netmap emulated mode.

### IPS retest
After applying the tunable and reloading IDS/IPS, the same ping test was repeated:

```cmd
ping 192.168.10.1 -t
```

### Result
This time the ping failed, showing that IPS was actively dropping the traffic.

---

## Validation Summary

| Validation Item | Result |
|----------------|--------|
| OPNsense LAN reachable before IPS drop test | Yes |
| IDS alert generated | Yes |
| IPS blocking confirmed | Yes |
| Firewall rules created successfully | Yes |
| Same-LAN filtering behaved as expected for routed firewalling | No |

---

## Troubleshooting Notes

### 1. Old VLAN configuration affected the new task
The previous VLAN lab left:
- VLAN interfaces
- Hyper-V VLAN tags
- trunk mode on OPNsense LAN

These had to be removed before the IDS/IPS lab could be tested properly.

### 2. Firewall block rule did not stop host-to-host traffic
This happened because both hosts were inside the same LAN and communicated directly on the switch.

### 3. IDS initially showed no alerts
At first, rules were not loaded correctly. After downloading and applying rules, Suricata log entries confirmed that rule reload completed successfully.

### 4. IPS initially did not drop traffic
The fix was to add:

`dev.netmap.admode = 2`

This was required for the Hyper-V virtual environment.

---

## Conclusion

This lab provided practical experience with three different protection layers in OPNsense.

- The **firewall** was used to define traffic rules
- **IDS** was used to detect and alert on matching traffic
- **IPS** was used to actively block matching traffic

The lab also showed that testing matters as much as configuration. In a flat LAN, traffic between hosts does not always pass through the firewall as expected, which affects both filtering and detection. Testing traffic directly against OPNsense made it possible to verify IDS/IPS correctly.

This made the exercise more realistic and gave a better understanding of how security tools actually behave in a virtual lab environment.
