# Wazuh-Based SIEM Project

## Objective
A Security Information and Event Management (SIEM) system is a critical component of modern cybersecurity infrastructure. It collects, analyzes, and correlates security data from multiple sources to detect threats and respond to incidents in real time.

This project focuses on building a SIEM solution using Wazuh, an open-source security platform that provides unified threat detection, integrity monitoring, incident response, and compliance management.

To design and implement a Wazuh-based SIEM system for centralized monitoring, real-time threat detection, analysis, and improved incident response capabilities
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

one of the first thing we need to install # sysmon on windows 10, when you hook Sysmon into Wazuh, you can detect a lot of high-value activity that standard Windows logs often miss. The combo is popular because Sysmon gives deep visibility, and Wazuh handles correlation, alerting, and dashboards.

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
it's not its usual system path


**That mismatch is the detection signal**

### Conclusion


This alert indicates a potential Living-Off-the-Land Binary (LOLBin) abuse technique where a trusted Microsoft Windows executable (cleanmgr.exe) appears to have created or dropped another executable (DismHost.exe) into a suspicious temporary user directory.

## Monitor a Specific location:

we can set up out wazuh agent to minitor a spesific location as well by modifying OSSEC.CONF  under C:\Program Files (x86)\ossec-agent.

For out lab here we are going to monitor the network drive, already setup the sahred folder on the server and added that location on windows 10 so now we can share files all over our network.

<img width="625" height="247" alt="Network Share" src="https://github.com/user-attachments/assets/d2578598-1663-42ec-8fb8-d23a99a91280" />

first we need to enable Audit File System from the Local group Policy. and auditing from the shared folder on the server

# Edit the OSSEC.CONF file:
 we need to add the file location then restart the WAZUH service 
 
 <directories check_all="yes" realtime="yes" whodata="yes">C:\Shared Drive MHNDAZ</directories>




