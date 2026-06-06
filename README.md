# Snort-Lab

Project Overview

This small project demonstrates network traffic analysis and intrusion detection within a segmented lab environment. An attacker launches web application attacks against a vulnerable target machine while Snort is deployed as a Network Intrusion Detection System (NIDS) to monitor traffic traversing the target network and generate logs related to malicious activity.

Generated Snort logs were securely transferred over SSH/SCP to Kali Linux for further investigation using Wireshark. The project highlights network traffic monitoring, IDS logging, packet analysis, and identification of common web attack indicators.

## Tools Used
- **Ubuntu Server** – Configured as a router to forward traffic between segmented networks.
- **Snort** – Deployed on Ubuntu Server as a Network Intrusion Detection System (NIDS) for traffic monitoring and log creation.
- **Metasploitable 2** – Vulnerable target machine hosting DVWA (Damn Vulnerable Web Application).
- **Kali Linux** – Used as the attacker system to simulate web attacks and perform log analysis with Wireshark.
- **Wireshark** – Used for packet capture of SCP file transfers and analysis of Snort logs.
- **SSH/SCP** – Used to securely transfer Snort logs for analysis.
