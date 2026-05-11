# Wazuh-Based SIEM Project

## Overview

Security Information and Event Management (SIEM) platforms help organizations collect, monitor, and analyze logs from multiple systems in one place. They improve visibility across the network and help security teams detect suspicious activity faster.

In this project, I built a SIEM lab using **Wazuh**, an open-source security monitoring platform. The lab focuses on centralized logging, threat detection, file integrity monitoring, and alert analysis within a virtualized environment.

---

# Project Goals

- Build a centralized SIEM environment using Wazuh
- Monitor endpoint activity in real time
- Detect suspicious behavior using Sysmon logs
- Configure File Integrity Monitoring (FIM)
- Improve visibility into system and network events

---

# Skills Learned

- SIEM deployment and configuration
- Log analysis and event correlation
- Windows event monitoring with Sysmon
- File Integrity Monitoring (FIM)
- Threat detection and alert investigation
- Windows and Linux system administration
- Network segmentation and virtual lab setup

---

# Tools & Technologies

- Wazuh
- Sysmon
- VMware Workstation
- Windows 10
- Windows Server 2016
- Kali Linux
- OPNsense Firewall

---

# Lab Environment

The lab environment was built in VMware using multiple virtual machines:

| Machine | Purpose |
|---|---|
| Windows 10 | Wazuh Endpoint Agent |
| Windows Server 2016 | Domain Controller & Shared Storage |
| Kali Linux | Wazuh Manager/Server |
| OPNsense | Firewall & Network Segmentation |

The following diagram shows how the systems communicate within the lab environment.

## Network Diagram

<img width="736" height="523" alt="DYGRM" src="https://github.com/user-attachments/assets/34306cf3-d109-4822-9666-e49b7a0bc166" />


---

# Installing Sysmon on Windows 10

One of the first steps was installing **Sysmon** on the Windows 10 endpoint.

Sysmon provides detailed Windows event logging that goes beyond standard Windows logs. When integrated with Wazuh, it improves visibility into:

- Process creation
- File creation
- Registry modifications
- Network connections
- Privilege escalation attempts

After installing Sysmon and connecting the endpoint agent to Wazuh, the logs became visible in the Wazuh dashboard.

<img width="1223" height="797" alt="sysmon" src="https://github.com/user-attachments/assets/6bb443de-ec0f-4d13-8afe-9fd177bb0922" />

---

# Suspicious Sysmon Alert Detection

Wazuh generated a high-severity alert based on Sysmon activity.

<img width="903" height="736" alt="single-Sysmon Alert" src="https://github.com/user-attachments/assets/7ba15c29-d8b0-4185-92a6-cbb00ba40338" />

## Alert Analysis

The alert showed that a legitimate Windows binary, `cleanmgr.exe`, created another executable inside a temporary user directory.

### Legitimate Process

```powershell
C:\Windows\System32\cleanmgr.exe
```

### Suspicious File Creation

```powershell
C:\Users\User2\AppData\Local\Temp\...\DismHost.exe
```

This behavior is commonly associated with **Living-Off-the-Land Binary (LOLBin)** techniques, where attackers abuse trusted Microsoft binaries to execute or drop malicious payloads while avoiding detection.

### Detection Indicators

- Trusted Microsoft binary execution
- Executable dropped into a temporary directory
- Unusual parent-child process behavior

This demonstrates how Sysmon and Wazuh can work together to identify suspicious activity that may otherwise appear legitimate.

---

# File Integrity Monitoring (FIM)

Another feature configured in this project was **File Integrity Monitoring (FIM)**.

The goal was to monitor a shared network folder for:

- File creation
- File modification
- File deletion
- File renaming

A shared folder was created on the Windows Server and mapped to the Windows 10 machine.

<img width="625" height="247" alt="Network Share" src="https://github.com/user-attachments/assets/d2578598-1663-42ec-8fb8-d23a99a91280" />

---

# Configuring Wazuh Agent Monitoring

To monitor the shared directory, the `ossec.conf` file was modified on the Windows endpoint located at:

```powershell
C:\Program Files (x86)\ossec-agent\
```

## Steps Performed

1. Enabled **Audit File System** policies in Local Group Policy
2. Enabled auditing on the shared folder
3. Added the monitored path to `ossec.conf`
4. Restarted the Wazuh agent service

<img width="1095" height="357" alt="FIM OSSEC file " src="https://github.com/user-attachments/assets/867b7f9a-4962-42a5-9065-a744adbc3a25" />

---

# File Change Detection

Once monitoring was enabled, any file activity inside the shared folder generated alerts in Wazuh.

This included:

- Creating files
- Editing files
- Renaming files
- Deleting files

<img width="1677" height="171" alt="rename and modifier the file" src="https://github.com/user-attachments/assets/3ec536ac-803b-4815-9a48-805484ab4217" />

Wazuh provided detailed event information, including the user responsible for the file modification.

<img width="912" height="772" alt="modified from W10 By the administrator " src="https://github.com/user-attachments/assets/87a5750c-93a2-4d43-9690-4bc98f410b32" />

<img width="866" height="791" alt="FIM Alert Detail" src="https://github.com/user-attachments/assets/55708448-6d1b-483a-95b7-09ff5a403f90" />

---

# Conclusion

This project demonstrates how Wazuh can be used as a powerful open-source SIEM solution for centralized monitoring and threat detection.

By integrating Sysmon and configuring File Integrity Monitoring, the environment was able to:

- Detect suspicious endpoint behavior
- Monitor critical file activity
- Generate real-time security alerts
- Improve visibility across systems

This lab provided hands-on experience with SIEM operations, log analysis, endpoint monitoring, and practical threat detection techniques commonly used in security operations environments.

---




# Simulating Brute-Force Attacks in the Wazuh SIEM Lab

# Overview

This section demonstrates how brute-force attacks were simulated inside the lab environment and detected using Wazuh.

The objective was to:
- Generate failed login attempts
- Trigger security alerts
- Monitor authentication logs
- Analyze attack behavior inside the SIEM dashboard

This helps simulate real-world attack activity commonly seen in enterprise environments.

---

# Lab Setup

| Machine | Role |
|---|---|
| Windows Server 2016 | Domain Controller |
| Windows 10 | Target Endpoint |
| Kali Linux | Attacker Machine |
| Wazuh Server | SIEM Monitoring |

---

# Step 1 — Enable Failed Login Auditing

On the Windows target machine:

## Open Local Security Policy

```text
secpol.msc
```

Navigate to:

```text
Security Settings → Advanced Audit Policy Configuration → Logon/Logoff
```

Enable:

- Audit Logon → Failure
- Audit Credential Validation → Failure

Apply the changes.

---

# Step 2 — Verify Wazuh Agent Connectivity

Ensure:
- The Wazuh agent is installed
- The endpoint is connected to the Wazuh manager
- Windows Security logs are being collected

You should already see authentication events in the dashboard.

---

# Step 3 — Install CrackMapExec on Kali Linux

Update the system:

```bash
sudo apt update
```

Install CrackMapExec:

```bash
sudo apt install crackmapexec -y
```

---

# Step 4 — Simulate SMB Brute Force Attack

Run a brute-force attempt against the Windows machine.

## Example Command

```bash
crackmapexec smb <TARGET-IP> -u administrator -p passwords.txt
```

Example:

```bash
crackmapexec smb 192.168.1.10 -u administrator -p passwords.txt
```

This generates:
- Multiple failed login attempts
- Windows authentication logs
- Security events collected by Wazuh

---

# Alternative Method — Hydra

Hydra can also be used for brute-force simulations.

## Install Hydra

```bash
sudo apt install hydra -y
```

## Run SMB Brute Force

```bash
hydra -l administrator -P passwords.txt smb://192.168.1.10
```

---

# Step 5 — Monitor Wazuh Alerts

Open the Wazuh dashboard and review generated alerts.

Typical events include:
- Failed logon attempts
- Multiple authentication failures
- Suspicious login behavior
- SMB authentication activity

---

# Common Windows Event IDs

| Event ID | Description |
|---|---|
| 4625 | Failed logon attempt |
| 4624 | Successful logon |
| 4776 | Credential validation |
| 4740 | Account lockout |

---

# Example Detection

Wazuh may generate alerts similar to:

```text
Multiple Windows login failures detected
```

Or:

```text
Possible brute-force attack detected
```

---

# Investigating the Attack

Inside Wazuh, you can investigate:

- Source IP address
- Username targeted
- Number of failed attempts
- Authentication method
- Time of attack
- Severity level

This helps simulate SOC-style investigations.

---

# Optional — Trigger Account Lockout

To make the simulation more realistic:

## Configure Account Lockout Policy

Open:

```text
gpmc.msc
```

Navigate to:

```text
Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies
```

Set:
- Account lockout threshold
- Lockout duration
- Reset counter timing

After enough failed attempts:
- The account locks automatically
- Wazuh generates additional alerts

---

# Using Wireshark During the Attack

Wireshark can capture:
- SMB traffic
- Authentication attempts
- Failed sessions

## Useful Filter

```bash
smb || tcp.port == 445
```

This helps correlate:
- Network traffic
- Authentication logs
- Wazuh alerts

---

# Detection Improvements

You can improve detection by:
- Creating custom Wazuh rules
- Adding Suricata IDS
- Blocking IPs automatically with Active Response
- Alerting on excessive failed logins

---

# Example Resume/Project Points

- Simulated SMB brute-force attacks using CrackMapExec and Hydra
- Monitored Windows authentication events with Wazuh
- Investigated failed login attempts using SIEM dashboards
- Correlated packet captures with endpoint security logs
- Configured account lockout policies and alerting mechanisms

---

# Conclusion

This simulation demonstrated how Wazuh can detect and analyze brute-force authentication attacks in a controlled lab environment. By combining endpoint logging, SIEM monitoring, and packet analysis, the lab provided hands-on experience with attack detection and incident investigation techniques used in real-world SOC environments.


