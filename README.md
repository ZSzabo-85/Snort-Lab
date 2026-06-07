# Snort-Lab

Project Overview

This small project demonstrates network traffic analysis and intrusion detection within a segmented lab environment. An attacker launches web application attacks against a vulnerable target machine while Snort is deployed as a Network Intrusion Detection System (NIDS) to monitor traffic traversing the target network and generate logs related to malicious activity.

Generated Snort logs were securely transferred over SSH/SCP to Kali Linux for further investigation using Wireshark. The project highlights network traffic monitoring, IDS logging, packet analysis, and identification of common web attack indicators.

## Tools Used
- **Ubuntu Server** – Configured as a router to forward traffic between segmented networks.
- **Snort** – Deployed on Ubuntu Server as a Network Intrusion Detection System (NIDS) for traffic monitoring and log creation.
- **Metasploitable 2** – Vulnerable target machine hosting DVWA (Damn Vulnerable Web Application).
- **Kali Linux** – Used as the attacker to simulate web attacks and perform log analysis with Wireshark.
- **Wireshark** – Used for packet capture of SCP file transfers and analysing Snort logs.
- **SSH/SCP** – Used to securely transfer Snort logs for analysis.

## Attack Simulation:

- Reflected Cross-Site Scripting (XSS)
- Command Injection

## Network configuration 

| Device | IP Address |
|---|---|
| Router | 192.168.80.1 & 192.168.90.1 |
| Snort | 192.168.80.254 |
| Metasploitable 2 | 192.168.80.10 |
| Kali Linux Attacker| 192.168.90.5 |

## Router Configuration
To allow communication between the two networks, enable IP forwarding on the router.
Edit the sysctl configuration file:
```bash
sudo nano /etc/sysctl.conf
```
Enable IPv4 forwarding by changing `net.ipv4.ip_forward=0` to `net.ipv4.ip_forward=1`.

Apply the changes:
```bash
sudo sysctl -p
```


