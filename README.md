[README_Enterprise_Firewall_DMZ_Suricata_Wazuh.md](https://github.com/user-attachments/files/29994832/README_Enterprise_Firewall_DMZ_Suricata_Wazuh.md)
# Enterprise Firewall, DMZ, Suricata IDS and Wazuh SIEM Lab

## Overview

I built this practical cybersecurity home lab to understand how network segmentation, firewall rules, intrusion detection and central log collection work together in an enterprise-style environment.

The lab was created in Oracle VirtualBox using pfSense, Windows 10, Ubuntu Server and Kali Linux. I configured separate LAN and DMZ networks, deployed an Apache web server in the DMZ, tested firewall isolation, generated controlled reconnaissance traffic with Nmap, detected suspicious activity with Suricata IDS and configured pfSense syslog forwarding toward a Wazuh SIEM environment.

## Lab Architecture


VirtualBox NAT / Internet
          |
     pfSense Firewall
          |
   +------+------+
   |             |
   LAN           DMZ
192.168.10.0/24  192.168.20.0/24
   |             |
Windows 10       Ubuntu Server
Kali + Wazuh     Apache Web Server




| System | Role | IP Address |
|---|---|---|
| pfSense | Firewall / Router | LAN: 192.168.10.1, DMZ: 192.168.20.1 |
| Windows 10 | Internal LAN client | 192.168.10.100 |
| Kali Linux | Security testing and Wazuh host | 192.168.10.101 |
| Ubuntu Server | DMZ Apache web server | 192.168.20.100 |
 What I Implemented

- Configured pfSense with separate WAN, LAN and DMZ interfaces.
- Enabled DHCP for the LAN and DMZ networks.
- Installed Apache2 on Ubuntu Server and hosted a test web page in the DMZ.
- Created firewall rules to block DMZ-to-LAN traffic while allowing required DNS and web traffic.
- Verified network segmentation using ICMP tests and pfSense firewall logs.
- Used Nmap from Kali to identify exposed services on the Ubuntu DMZ server.
- Configured Suricata IDS with Emerging Threats Open rules.
- Generated controlled scan traffic and reviewed Suricata Priority 1 alerts.
- Ran a Wazuh single-node Docker environment on Kali.
- Configured Wazuh to listen for pfSense syslog events over UDP port 514.
- Configured pfSense remote logging and validated the syslog transport path with tcpdump.
- Captured a readable pfSense `filterlog` event containing the blocked DMZ-to-LAN traffic.

 Security Testing

 Nmap Service Discovery


nmap -sV 192.168.20.100


Nmap identified TCP port 80 as open and detected the Apache HTTP service.

A more detailed controlled scan was also performed:


sudo nmap -sS -sV -O -T4 -p- 192.168.20.100


This traffic was generated only inside my private VirtualBox lab.

### Suricata IDS Detection

Suricata monitored the lab network and detected scan-related traffic from the Kali system to the Ubuntu DMZ web server.

Observed alert details included:

- Source: `192.168.10.101`
- Destination: `192.168.20.100`
- Destination Port: `80`
- Priority: `1`
- Classification: `Web Application Attack`

The alert showed that the IDS detected suspicious network activity. It did not prove a successful compromise.

### DMZ Isolation Test

Traffic from the Ubuntu DMZ server to the Windows LAN client was blocked by pfSense.


Source:      192.168.20.100
Destination: 192.168.10.100
Action:      Block


The test produced 100% packet loss and a matching pfSense firewall log.

## Wazuh and Syslog Integration

The Wazuh manager was configured to accept syslog from the pfSense LAN IP:


<remote>
  <connection>syslog</connection>
  <port>514</port>
  <protocol>udp</protocol>
  <allowed-ips>192.168.10.1</allowed-ips>
</remote>


I configured pfSense remote logging to send syslog to:

```text
192.168.10.101:514
```

Packet capture confirmed automatic UDP 514 traffic from pfSense to the Kali/Wazuh host. An ASCII packet capture also showed the real pfSense `filterlog` block event.

## Important Limitation

The pfSense-to-Wazuh syslog transport path was validated at the network layer, but I did not confirm a matching pfSense firewall alert in the Wazuh dashboard.

Custom Wazuh decoder or alert-rule tuning is future work.

## Tools and Technologies

- pfSense
- Suricata IDS
- Wazuh SIEM
- Nmap
- tcpdump
- Apache2
- Ubuntu Server
- Kali Linux
- Windows 10
- Docker
- Oracle VirtualBox

## Skills Demonstrated

- Network segmentation
- Firewall rule configuration
- DMZ design
- Linux networking
- Network reconnaissance
- IDS monitoring
- SIEM and syslog integration
- Packet analysis
- Security troubleshooting
- Technical documentation

## Project Report

A detailed technical report with screenshots, an action log, troubleshooting decisions, a risk register and an evidence matrix is included in this repository.

## Ethical Scope

All testing was performed inside my own isolated VirtualBox home lab using private IP addresses and systems I configured. No third-party or public systems were scanned.
