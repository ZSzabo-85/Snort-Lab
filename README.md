# Snort-Lab

## Project Overview

This small project demonstrates network traffic analysis and intrusion detection within a segmented lab environment. An attacker launches web application attacks against a vulnerable target machine while Snort is deployed as a Network Intrusion Detection System (NIDS) to monitor traffic traversing the target network and generate logs related to malicious activity.

Generated Snort logs were securely transferred over SSH/SCP to Kali Linux for further investigation using Wireshark. The project highlights network traffic monitoring, IDS logging, packet analysis, and identification of common web attack indicators.

## Tools Used
- **Ubuntu Server** – Configured as a router to forward traffic between segmented networks.
- **Snort** – Deployed on Ubuntu Server as a Network Intrusion Detection System (NIDS) for monitoring traffic and generating logs.
- **Metasploitable 2** – Vulnerable target machine hosting DVWA (Damn Vulnerable Web Application).
- **Kali Linux** – Used as the attacker to simulate web attacks and perform log analysis with Wireshark.
- **Wireshark** – Used for packet capture of SCP file transfers and analysing Snort logs.
- **SSH/SCP** – Used to securely transfer Snort logs for analysis.

## Attack Simulation:

- Reflected Cross-Site Scripting (XSS)
- Command Injection

## Network Configuration 

| Device | IP Address | Default Gateway |
|---|---|---|
| Router | 192.168.80.1 & 192.168.90.1 | N/A | 
| Snort | 192.168.80.254 | 192.168.80.1 |
| Metasploitable 2 | 192.168.80.10 | 192.168.80.1 |
| Kali Linux Attacker| 192.168.90.5 | 192.168.90.1 | 

If devices configured correctly they should be able to communicate each other. **ping** command can be used to test connections.

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

Start snort to monitor traffic 

```bash
sudo snort -A full -c /etc/snort/snort.conf -I enp0s8
```
Snort monitors 192.168.80.0/24 network and generate logs to /var/log/snort by default if there is no path given.

## Attack simulation
The following activities will be used to generate logs
Using Kali web browser for  Command injection and XSS to DVWA hosted on Metasploitable.
Kali web browser http://192.168.80.10 then select DVWA

**Note:** Snort was stopped and restarted between the XSS and Command Injection tests. This created separate log files for each attack scenario, making it easier to analyse and correlate events during the investigation process.
### XSS

Use payload as <script>alert(“THEPAYLOAD”)</script>

![Reflected XSS payload submitted through DVWA](https://github.com/user-attachments/assets/9faefca9-c6e9-499f-a9f9-a3fabd2b4c26)

![Result of XSS payload](https://github.com/user-attachments/assets/8c8b5068-4083-4e42-8ee9-c6c96052a948)

### Command Injection

The attacker initially submitted a standard IP address to verify connectivity, which resulted in a successful ping response with no packet loss.

Further testing involved injecting additional system commands into the input field to observe whether they would be executed on the target system.

Common commands can be used:
- `whoami` - Current user
- `ls` - List files / directories in the current location
- `ip a` - list interfaces and IP addresses
- `cat /etc/passwd` - read local user accounts
- `hostname` - obtain system hostname
- `ps aux` - running processes
-  And more
  
**The followwing payloads were submitted**

- 127.0.0.1 && whoami 
- 127.0.0.1 && ip a && whoami
- 127.0.0.1 && hostname

  ![whomai](https://github.com/user-attachments/assets/7b9eeeb0-bdd7-4a5d-9e29-491eadfdec80)

 ![whoami && ip a](https://github.com/user-attachments/assets/147d3dd5-c3aa-45ea-807b-53237469514b)


## Log Analysis

The generated Snort logs were transferred from the Ubuntu Server to Kali Linux using SCP for further analysis. Wireshark was used to capture and analyse the network traffic associated with the SSH session.

### SCP Packet Capture Analysis

### SCP Packet Capture Analysis

This capture shows an SSH session established between the Ubuntu Server (client) and Kali Linux (server) during SCP file transfer.

Wireshark displays the SSHv2 handshake process, including key exchange and session establishment. After the handshake completes, all traffic is encrypted and appears as SSH “Encrypted packet” data.

This confirms that the Snort logs were securely transferred and are not visible in plaintext during transit.

![Wireshark showing SCP file transfer](https://github.com/user-attachments/assets/95fdb843-a3b6-4be1-8ac6-d33784ab30a5)

