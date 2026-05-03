# Wazuh-Based SIEM Project

## Objective
To design and implement a Wazuh-based SIEM system for centralized monitoring, real-time threat detection, analysis, and improved incident response capabilities

A Security Information and Event Management (SIEM) system is a critical component of modern cybersecurity infrastructure. It collects, analyzes, and correlates security data from multiple sources to detect threats and respond to incidents in real time.

This project focuses on building a SIEM solution using Wazuh, an open-source security platform that provides unified threat detection, integrity monitoring, incident response, and compliance management.
### Skills Learned
-tochange
- Advanced understanding of SIEM concepts and practical application.
- Proficiency in analyzing and interpreting network logs. 
- Ability to generate and recognize attack signatures and patterns.
- Enhanced knowledge of network protocols and security vulnerabilities.
- Development of critical thinking and problem-solving skills in cybersecurity.

### Tools Used
[Bullet Points - Remove this afterwards]
-tochange
- Security Information and Event Management (SIEM) system for log ingestion and analysis.
- Network analysis tools (such as Wireshark) for capturing and examining network traffic.
- Telemetry generation tools to create realistic network traffic and attack scenarios.

## Steps

I’m going to set up all my virtual machines using VMware. For this project, I will use a Windows 10 machine to install the Wazuh endpoint agent, a Windows Server 2016 machine to host the domain controller, an OPNsense firewall, and a Kali Linux machine to host the Wazuh server. The following network diagram demonstrates how these machines communicate with each other.

*Ref 1: Network Diagram*


<img width="1000" height="507" alt="DYGRM" src="https://github.com/user-attachments/assets/413b1d43-94c7-434b-bcaa-e9c2f0957cc6" />

one of the first we need to install # sysmon on windows 10, when you hook Sysmon into Wazuh, you can detect a lot of high-value activity that standard Windows logs often miss. The combo is popular because Sysmon gives deep visibility, and Wazuh handles correlation, alerting, and dashboards.

after installing sysmon on the endpoing we can see here a list of logs that is showing on the wazuh server

<img width="1223" height="797" alt="sysmon" src="https://github.com/user-attachments/assets/6bb443de-ec0f-4d13-8afe-9fd177bb0922" />

Here, we have a single alert classified as critical.

<img width="903" height="736" alt="single-Sysmon Alert" src="https://github.com/user-attachments/assets/7ba15c29-d8b0-4185-92a6-cbb00ba40338" />

# And here is the explanation of the alert:

“A legit Windows binary dropped another executable in a sketchy location”

This is the core idea behind Living-Off-the-Land (LOLBins):

Attackers use trusted Windows binaries (like cleanmgr.exe)
To write or execute payloads
In locations that blend in (like AppData\Temp)

## Mapping directly to YOUR event

From your log:

# Legit binary
Image: C:\Windows\system32\cleanmgr.exe
Signed Microsoft binary
Normally safe
# Suspicious action
TargetFilename:
C:\Users\User2\AppData\Local\Temp\...\DismHost.exe
Dropped executable
In user Temp directory
Not its usual system path
# Why this combination matters

On its own:

cleanmgr.exe → fine
DismHost.exe → fine

Together in this context:

❗ unexpected parent-child relationship
❗ unexpected file location

**That mismatch is the detection signal**



