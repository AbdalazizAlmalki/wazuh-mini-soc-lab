# Wazuh Mini SOC Lab

A home-built Security Operations Center (SOC) lab simulating real-world threat detection, built and documented end-to-end on an isolated virtual network using **Wazuh**, **VMware Workstation Pro**, **Kali Linux**, and **Windows 11**.

## Overview

This project demonstrates hands-on skills in deploying, securing, and monitoring a small enterprise-style environment using an open-source SIEM (Security Information and Event Management) platform. It covers environment isolation, endpoint monitoring, file integrity monitoring, and detection of a simulated brute-force attack — mapped to the MITRE ATT&CK framework.

## Architecture

| Machine | Role | OS | IP Address |
|---|---|---|---|
| Wazuh-Server | SIEM manager, indexer, dashboard | Amazon Linux (VMware) | 192.168.232.128 |
| Kali-Attacker | Simulated threat actor | Kali Linux (VMware) | 192.168.232.130 |
| Windows11-Victim | Monitored endpoint (Wazuh agent installed) | Windows 11 (VMware) | 192.168.232.129 |

All three virtual machines run on a **Host-only** virtual network (VMnet1, `192.168.232.0/24`), fully isolated from the internet and the home network — verified by failed ping tests to external targets from the attacker machine.

## What This Project Demonstrates

- Deploying and hardening a Wazuh manager, indexer, and dashboard
- Installing and connecting a Wazuh agent to a Windows endpoint
- Changing default credentials and recovering a locked-out Linux account via GRUB recovery mode
- Diagnosing and fixing real network connectivity issues (DHCP, host-only adapters, Windows network-profile firewall rules)
- Configuring and testing **File Integrity Monitoring (FIM)** on a watched directory
- Simulating a **brute-force RDP authentication attack** (Hydra → xfreerdp) against the Windows endpoint
- Analyzing Wazuh's detection output: raw Windows Event ID 4625 logs, alert severity levels, and automatic **MITRE ATT&CK** classification (Password Guessing)
- Safe post-test cleanup and VM snapshot management

## Tools & Technologies

`Wazuh` `VMware Workstation Pro` `Kali Linux` `Windows 11` `Hydra` `xfreerdp` `PowerShell` `MITRE ATT&CK`

## Screenshots

See the [`screenshots/`](./screenshots) folder for the full set of captures covering setup, configuration, the FIM test, and the brute-force attack and its detection in the Wazuh dashboard.

## Incident Report

A full incident report covering the brute-force attack — methodology, detection, MITRE ATT&CK mapping, remediation, and recommendations — is included in [`incident-report.md`](./incident-report.md).

## Disclaimer

This lab runs entirely inside an isolated virtual network created for learning purposes. No real systems, networks, or credentials were targeted.

## Author

**Abdulaziz Al-Malki**
[LinkedIn](https://linkedin.com/in/abdalaziz-yahi)
