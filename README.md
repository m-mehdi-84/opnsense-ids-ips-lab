# OPNsense IDS/IPS Lab

A hands-on cybersecurity lab built using Hyper-V and OPNsense to explore firewall rules, intrusion detection, and intrusion prevention in a virtual environment.

This lab demonstrates how to configure LAN firewall rules, enable IDS/IPS in OPNsense, test alert generation, and verify that IPS can actively block traffic.

---

## Lab Overview

This lab simulates a small internal network protected by OPNsense.

Components:

- Hyper-V virtualization
- OPNsense firewall/router
- Windows Server
- Windows 11 client
- Optional LAN2 client used during validation/troubleshooting

---

## Network Topology

                    INTERNET
                        |
                   [ WAN - hn0 ]
                        |
                 +----------------+
                 |    OPNsense    |
                 | Firewall/Router|
                 +--------+-------+
                          |
                    [ LAN - hn1 ]
                          |
                   +---------------+
                   |   LAN-Switch   |
                   +-------+--------+
                           |
              +------------+------------+
              |                         |
      Windows Server               Windows 11
      192.168.10.145              192.168.10.144
      GW: 192.168.10.1            GW: 192.168.10.1

Additional lab interface kept in environment:
- LAN2 available for separate testing/troubleshooting

---

## Network Design

| Network | Subnet | Gateway | Purpose |
|--------|--------|---------|---------|
| LAN | 192.168.10.0/24 | 192.168.10.1 | Main internal lab network |
| LAN2 | 192.168.20.0/24 | 192.168.20.1 | Secondary test network |

---

## Key Features

- LAN firewall rule configuration in OPNsense
- IDS setup using Intrusion Detection
- User defined IDS rule testing
- IPS validation using Netmap mode
- Alert generation and packet blocking
- Hyper-V lab troubleshooting for virtualized IPS

---

## Security Concept

This lab demonstrates three core security layers:

- **Firewall** controls which traffic is allowed or blocked
- **IDS** detects suspicious traffic and generates alerts
- **IPS** goes further by actively dropping matching traffic

The lab also shows an important practical detail:
traffic inside the same LAN does not always pass through OPNsense in a way that matches expectations for inspection and filtering. Because of that, testing directly against OPNsense was used to verify IDS/IPS functionality.

---

## Configuration Summary

### Firewall
- Created a pass rule from `LAN net` to the Windows Server
- Created a block rule from the Windows Server to `LAN net`
- Verified that the rules were configured correctly in OPNsense

### IDS
- Enabled **Intrusion Detection**
- Selected the LAN interface
- Downloaded and applied rules
- Created a **user defined alert rule**
- Verified alert generation by sending ICMP traffic from the client to OPNsense

### IPS
- Changed capture mode to **Netmap (IPS)**
- Changed the user defined rule action from **Alert** to **Drop**
- Added tunable:

`dev.netmap.admode = 2`

- Verified that ICMP traffic to OPNsense was dropped successfully

---

## Verification

| Test | Result |
|------|--------|
| Win11 IP assignment | 192.168.10.144 |
| Server IP assignment | 192.168.10.145 |
| Ping Win11 → Server | Success |
| Ping Server → Win11 | Success |
| IDS alert on traffic to OPNsense | Success |
| IPS drop on traffic to OPNsense | Success |

---

## Screenshots

Add your screenshots here, for example:

### Firewall Rules
- LAN firewall rules
- Rule order and descriptions

### IDS Configuration
- Intrusion Detection settings
- Downloaded rules
- Policy / Rules / User defined alert rule

### IDS Alert Test
- Alert showing ICMP traffic from Win11 to OPNsense

### IPS Configuration
- Netmap (IPS) enabled
- User defined rule changed to Drop
- Tunable `dev.netmap.admode = 2`

### IPS Validation
- Ping failure / request timed out when IPS was active

---

## Documentation

Detailed step-by-step guide is available in:

`docs/lab-documentation.md`

Includes:

- Lab preparation
- Firewall rule configuration
- IDS configuration
- IPS configuration
- Troubleshooting
- Validation and testing

---

## What I Learned

- How firewall rules behave in a flat LAN
- How IDS detects and logs matching traffic
- How IPS actively blocks traffic instead of only alerting
- How virtualized environments may require extra tuning for IPS
- How to troubleshoot OPNsense IDS/IPS in Hyper-V

---

## Future Improvements

- Test IDS/IPS between separate routed networks
- Expand the lab with more clients and services
- Add Suricata rule tuning for more realistic traffic
- Combine IDS/IPS with segmented networks and logging strategy

---

## Author

Muhammad Mehdi  
IT Security Developer Student
