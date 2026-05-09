Wazuh-Based SIEM Project
Overview

Security Information and Event Management (SIEM) platforms help organizations collect, monitor, and analyze logs from multiple systems in one place. They improve visibility across the network and help security teams detect suspicious activity faster.

In this project, I built a SIEM lab using Wazuh, an open-source security monitoring platform. The lab focuses on centralized logging, threat detection, file integrity monitoring, and alert analysis within a virtualized environment.

Project Goals
Build a centralized SIEM environment using Wazuh
Monitor endpoint activity in real time
Detect suspicious behavior using Sysmon logs
Configure file integrity monitoring (FIM)
Improve visibility into system and network events
Skills Learned
SIEM deployment and configuration
Log analysis and event correlation
Windows event monitoring with Sysmon
File Integrity Monitoring (FIM)
Threat detection and alert investigation
Windows and Linux system administration
Network segmentation and virtual lab setup
Tools & Technologies
Wazuh
Sysmon
VMware Workstation
Windows 10
Windows Server 2016
Kali Linux
OPNsense Firewall
Wireshark
Lab Environment

The lab environment was built in VMware using multiple virtual machines:

Windows 10 → Wazuh endpoint agent
Windows Server 2016 → Domain Controller & shared network storage
Kali Linux → Wazuh Manager/Server
OPNsense → Firewall and network segmentation

The following diagram shows how the systems communicate within the lab environment.

Ref 1: Network Diagram

<img width="1000" height="507" alt="DYGRM" src="https://github.com/user-attachments/assets/413b1d43-94c7-434b-bcaa-e9c2f0957cc6" />
Installing Sysmon on Windows 10

One of the first steps was installing Sysmon on the Windows 10 endpoint.

Sysmon provides detailed Windows event logging that goes far beyond default Windows logs. When integrated with Wazuh, it allows the SIEM to detect suspicious activity such as:

Process creation
File creation
Registry modifications
Network connections
Privilege escalation attempts

After installing Sysmon and connecting the endpoint agent to Wazuh, the logs became visible in the Wazuh dashboard.

<img width="1223" height="797" alt="sysmon" src="https://github.com/user-attachments/assets/6bb443de-ec0f-4d13-8afe-9fd177bb0922" />
Suspicious Sysmon Alert Detection

Wazuh generated a high-severity alert based on Sysmon activity.

<img width="903" height="736" alt="single-Sysmon Alert" src="https://github.com/user-attachments/assets/7ba15c29-d8b0-4185-92a6-cbb00ba40338" />
Alert Analysis

The alert showed that a legitimate Windows binary, cleanmgr.exe, created another executable inside a temporary user directory.

Legitimate Process
C:\Windows\System32\cleanmgr.exe
Suspicious File Creation
C:\Users\User2\AppData\Local\Temp\...\DismHost.exe

This behavior is commonly associated with Living-Off-the-Land Binary (LOLBin) techniques, where attackers abuse trusted Microsoft binaries to execute or drop malicious payloads while avoiding detection.

The main detection indicator is the mismatch between:

A trusted Microsoft process
An unusual executable being written into a temporary directory

This demonstrates how Sysmon and Wazuh can work together to identify suspicious activity that may otherwise appear legitimate.

File Integrity Monitoring (FIM)

Another feature configured in this project was File Integrity Monitoring (FIM).

The goal was to monitor a shared network folder for:

File creation
File modification
File deletion
File renaming

A shared folder was created on the Windows Server and mapped to the Windows 10 machine.

<img width="625" height="247" alt="Network Share" src="https://github.com/user-attachments/assets/d2578598-1663-42ec-8fb8-d23a99a91280" />
Configuring Wazuh Agent Monitoring

To monitor the shared directory, the ossec.conf file was modified on the Windows endpoint located at:

C:\Program Files (x86)\ossec-agent\

Before enabling monitoring:

Audit File System policies were enabled in Local Group Policy
Auditing was enabled on the shared folder itself
The monitored path was added to ossec.conf
The Wazuh agent service was restarted
<img width="1095" height="357" alt="FIM OSSEC file " src="https://github.com/user-attachments/assets/867b7f9a-4962-42a5-9065-a744adbc3a25" />
File Change Detection

Once monitoring was enabled, any file activity inside the shared folder generated alerts in Wazuh.

This included:

Creating files
Editing files
Renaming files
Deleting files
<img width="1677" height="171" alt="rename and modifier the file" src="https://github.com/user-attachments/assets/3ec536ac-803b-4815-9a48-805484ab4217" />

Wazuh provided detailed event information, including the user responsible for the change.

<img width="912" height="772" alt="modified from W10 By the administrator " src="https://github.com/user-attachments/assets/87a5750c-93a2-4d43-9690-4bc98f410b32" />
Conclusion

This project demonstrates how Wazuh can be used as a powerful open-source SIEM solution for centralized monitoring and threat detection.

By integrating Sysmon and configuring File Integrity Monitoring, the environment was able to:

Detect suspicious endpoint behavior
Monitor critical file activity
Generate real-time security alerts
Improve visibility across systems

The lab provided hands-on experience with SIEM operations, log analysis, and practical threat detection techniques commonly used in security operations environments.
