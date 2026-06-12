# Snort-Lab

## Project Overview

This project demonstrates network traffic analysis and intrusion detection within a segmented lab environment. An attacker launched web application attacks against a vulnerable target machine while Snort was deployed as a Network Intrusion Detection System (NIDS) to monitor traffic traversing the target network and generate logs related to malicious activity.

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

## Table of Content

- [Network Configuration](#network-configuration)
- [Router Configuration](#router-configuration)
- [Snort Configuration](#snort-configuration)
- [Log Analysis](#log-analysis)
  - [SCP Packet Capture Analysis](#scp-packet-capture-analysis)
  - [XSS Analysis](#xss-analysis)
  - [Command Injection Analysis](#command-injection-analysis)
- [Indicators of Compromise (IOCs)](#indicators-of-compromise-iocs)
- [Key Takeaway](#key-takeaway)


## Network Configuration 

| Device | IP Address | Default Gateway |
|---|---|---|
| Router | 192.168.80.1 & 192.168.90.1 | N/A | 
| Snort | 192.168.80.254 | 192.168.80.1 |
| Metasploitable 2 | 192.168.80.10 | 192.168.80.1 |
| Kali Linux Attacker| 192.168.90.5 | 192.168.90.1 | 

If the devices are configured correctly they should be able to communicate with each other. **ping** command can be used to test connections.

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
## Snort Configuration

After installing Snort, configure the monitored network in the Snort configuration file.

Open the configuration file:

```bash
sudo nano /etc/snort/snort.conf
```
Configure the protected network

Set the HOME_NET variable to match the network being protected:

ipvar HOME_NET `192.168.80.0/24`

This tells Snort which network should be considered the protected internal network.

Enable Promiscuous Mode

```bash
sudo ip link set enp0s8 promisc on
```

This allows Snort to inspect all traffic on the network segment, rather than only traffic addressed to the sensor.

Start Snort to monitor network traffic.

```bash
sudo snort -A full -c /etc/snort/snort.conf -i enp0s8
```
Snort monitors traffic within the 192.168.80.0/24 network and generates logs in `/var/log/snort`.

## Attack simulation
The following activities were used to generate logs
Using Kali web browser for  Command injection and XSS to DVWA hosted on Metasploitable.
Browse to http://192.168.80.10 from Kali Linux then select DVWA.

**Note:** Snort was stopped and restarted between the XSS and Command Injection tests. This created separate log files for each attack scenario, making it easier to analyse and correlate events during the investigation process.
### XSS

Use payload as <script>alert(“THEPAYLOAD”)</script>

![Reflected XSS payload submitted through DVWA](https://github.com/user-attachments/assets/9faefca9-c6e9-499f-a9f9-a3fabd2b4c26)

![Result of XSS payload](https://github.com/user-attachments/assets/8c8b5068-4083-4e42-8ee9-c6c96052a948)

### Command Injection

The attacker first submitted a standard IP address to verify connectivity, which resulted in a successful ping response with no packet loss.

Further testing involved injecting additional system commands into the input field to observe whether they would be executed on the target system.

Common commands:
- `whoami` - Current user
- `ls` - List files / directories in the current location
- `ip a` - List interfaces and IP addresses
- `cat /etc/passwd` - read local user accounts
- `hostname` - obtain system hostname
- `ps aux` - running processes
-  And more
  
**The following payloads were submitted**

- 127.0.0.1 && whoami 
- 127.0.0.1 && ip a && whoami
- 127.0.0.1 && hostname

  ![whomai](https://github.com/user-attachments/assets/7b9eeeb0-bdd7-4a5d-9e29-491eadfdec80)

 ![whoami && ip a](https://github.com/user-attachments/assets/147d3dd5-c3aa-45ea-807b-53237469514b)


## Log Analysis

The generated Snort logs were transferred from the Ubuntu Server to Kali Linux using SCP for further analysis. Wireshark was used to capture and analyse the network traffic associated with the SSH session.

### SCP Packet Capture Analysis

This capture shows an SSH session established between the Ubuntu Server (client) and Kali Linux (server) during SCP file transfer.

Wireshark displays the SSHv2 handshake process, including key exchange and session establishment. After the handshake completes, all traffic is encrypted and appears as SSH “Encrypted packet” data.

This confirms that the Snort logs were securely transferred and are not visible in plaintext during transit.

![Wireshark showing SCP file transfer](https://github.com/user-attachments/assets/95fdb843-a3b6-4be1-8ac6-d33784ab30a5)

### XSS Analysis

HTTP GET requests can be filtered when analyzing web traffic:

`http.request.method == "GET"`

![Filter for GET request](https://github.com/user-attachments/assets/64a09c41-944d-459e-898e-248c584e45c3)


This helps narrow down visible traffic to only GET requests. However, in environments with high volumes of network activity, this can still result in hundreds or thousands of packets, making manual analysis time-consuming.

To improve detection efficiency, more specific filters can be applied to identify potential XSS-related patterns within request URIs.

Another method to detect XSS payloads is by using a more precise regular expression filter:

`http.request.uri matches "(?i)<script>|%3C|%253c"`

This helps identify common encoded and non-encoded <script> tags.
However, XSS attacks are not limited to <script> tags. Attackers may also use:

-	alert()
-	confirm()
-	prompt()
-	onerror
-	onload
-	onmouseover

OR

`http.request.uri matches "(?i)alert|prompt|confirm|onerror|script|%3C|%253C"`

![using regex for XSS pattern](https://github.com/user-attachments/assets/f85b931c-fee9-40d1-91f2-b33e5aad32f5)


**If these entries are detected, they should be investigated further to determine whether they represent malicious activity.**

### Command Injection Analysis

`http matches "(%26%26|%7c|%3b).*(whoami|id|ls|cat|passwd)"`

This Wireshark display filter detects potential command injection attempts by identifying:

- command chaining operators: &&, |, ;
- Linux commands: whoami, id, ls, cat, passwd

The filter identified malicious HTTP traffic containing command injection payloads submitted through user input.

![Command injection](https://github.com/user-attachments/assets/0305614b-c9c0-4e36-abb1-8010cc170b57)


## Indicators of Compromise (IOCs)

- Source IP: 192.168.90.5
- Destination IP: 192.168.80.10
- HTTP Method: POST
- Vulnerable endpoint: /dvwa/vulnerabilities/exec/
- Content-Type: application/x-www-form-urlencoded
- Malicious payload: 127.0.0.1 && ip a && whoami

## Key Takeaway

- Snort successfully detected malicious web application traffic.
- Command injection payloads were visible in HTTP POST requests.
- XSS payloads were identified through URI analysis.
- SSH/SCP traffic encrypted transfers and protected data in transit.
- Packet analysis helped to identify malicious IP address


 
