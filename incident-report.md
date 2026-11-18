# Brute-Force RDP Attack — Incident Report

**Date:** November 18, 2026
**Author:** Abdulaziz Al-Malki

On November 18, 2026, a simulated brute-force authentication attack against the Remote Desktop Protocol (RDP) service was carried out against a Windows 11 endpoint inside an isolated home lab, and was detected and correctly classified by a Wazuh SIEM deployment under the MITRE ATT&CK Password Guessing technique.

## Executive Summary

On November 18, 2026, a controlled brute-force authentication attack was launched from a Kali Linux attacker machine against the Remote Desktop Protocol (RDP) service of a Windows 11 endpoint, inside a fully isolated home lab built with Wazuh as the security monitoring platform. The attack generated 13 failed authentication attempts, which Wazuh correctly detected, logged with full forensic detail (source IP, target account, Windows Event ID 4625), and automatically classified under the MITRE ATT&CK "Password Guessing" technique. No real-world systems, networks, or credentials were involved; the exercise was performed entirely within a network-isolated virtual lab for skills demonstration purposes.

## Environment & Scope

| Asset | Role | OS / Platform | IP Address |
| --- | --- | --- | --- |
| Wazuh-Server | SIEM manager, indexer, dashboard | Amazon Linux (VMware) | 192.168.232.128 |
| Kali-Attacker | Attack source | Kali Linux (VMware) | 192.168.232.130 |
| Windows11-Victim | Target endpoint, Wazuh agent installed | Windows 11 (VMware) | 192.168.232.129 |

All three virtual machines ran on VMware Workstation Pro and were placed on a Host-only virtual network (VMnet1, subnet 192.168.232.0/24), fully isolated from the home network and the internet. Isolation was verified before testing: the attacker machine could reach the Wazuh server but failed to reach the internet or the home router.

## Attack Methodology

1. **Service exposure.** Remote Desktop was enabled on the Windows 11 victim (Settings > System > Remote Desktop), opening TCP port 3389 and the related Windows Firewall rule.
2. **Wordlist preparation.** A small password wordlist of common weak passwords (`123456`, `password`, `admin123`, `qwerty`, `letmein`) was created on the Kali attacker machine.
3. **First attempt — Hydra.** Hydra was tried first against the RDP service, but its RDP module (documented upstream as experimental) failed to establish a connection at all, returning a generic connection error with zero attempts completed.
4. **Pivot to xfreerdp.** The attack was switched to `xfreerdp`, the open-source RDP client, scripted in a bash loop to attempt a login with each password in the wordlist using the lightweight `+auth-only` flag.
5. **Obstacle — network profile.** The first xfreerdp attempts failed with "No route to host." Investigation with PowerShell's `Get-NetConnectionProfile` showed the victim's network adapter was classified as **Public**, under which Windows Firewall blocks inbound RDP by default. This was corrected with `Set-NetConnectionProfile -NetworkCategory Private`, after which the connection succeeded.
6. **Execution.** With the firewall obstacle resolved, the xfreerdp loop ran to completion: every password attempt reached the login prompt and was rejected with `ERRCONNECT_LOGON_FAILURE`, confirming 13 genuine authentication attempts reached the target account (`labuser`).

## Detection & Analysis

The Wazuh agent on the Windows 11 endpoint forwarded every failed logon to the Wazuh manager in real time. The **Threat Hunting** dashboard showed 13 authentication-failure events and 41 alerts at severity level 12 or higher within the test window, with no manual tagging required.

| Field | Value |
| --- | --- |
| Windows Event ID | 4625 (An account failed to log on) |
| Target account | `labuser` |
| Source workstation | `kali` |
| Source IP address | 192.168.232.130 |
| Failure reason | `%%2313` — bad username or password |
| Wazuh rule levels | 12+ (high severity) |

Raw events were cross-checked in Wazuh's **Discover** view to confirm every field matched the attack traffic exactly, ruling out false positives.

## MITRE ATT&CK Mapping

| Tactic | Technique | Technique ID |
| --- | --- | --- |
| Credential Access | Brute Force: Password Guessing | T1110.001 |
| Impact | Account Access Removal (related classification shown by Wazuh) | T1531 |

Wazuh applied this mapping automatically from the pattern of repeated RDP authentication failures against a single account, with no manual rule authoring required for this detection.

## Response & Remediation

Immediately after confirming detection, the test was closed down following standard lab hygiene:

- The Remote Desktop service was disabled on the Windows 11 endpoint, closing port 3389.
- Network Level Authentication (NLA), which had been toggled off to allow the test, was restored to its secure default (enabled).
- A VMware snapshot named `brute-force-tested` was taken immediately after cleanup, preserving a known-good, post-test state of the environment.

No persistent access, lateral movement, or credential compromise occurred: every one of the 13 attempts failed, and the lab remained fully isolated from production systems throughout.

## Recommendations

Based on this exercise, the following controls would reduce or prevent this class of attack in a production environment:

1. **Keep Network Level Authentication (NLA) enabled** on every RDP-exposed host; it requires authentication before a full session is established, cutting attacker feedback and resource cost per attempt.
2. **Enforce an account lockout policy** (e.g., lock after 5 failed attempts) so repeated password guessing is automatically blocked rather than only logged.
3. **Require multi-factor authentication (MFA)** for remote access, so a correctly guessed password is not enough on its own.
4. **Avoid exposing RDP directly**; place it behind a VPN or a bastion host, and restrict access by source IP where possible.
5. **Alert on repeated Event ID 4625 from the same source** in the SIEM, as demonstrated here, so analysts are notified before an attacker succeeds.
