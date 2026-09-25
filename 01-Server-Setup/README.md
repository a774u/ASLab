01 — Windows Server Setup
Overview
This section covers why I’m using Windows Server 2022, what a Domain Controller is, and how I installed the OS for my homelab. The goal is to build a realistic enterprise environment that supports Active Directory, Defender, Sentinel, and future attack/detection work.

Why Windows Server 2022
It’s the current enterprise standard

Supports Active Directory, DNS, Group Policy

Integrates cleanly with Microsoft Defender + Sentinel

Matches what real SOCs and IT teams use today

Why I Chose Desktop Experience
Easier to learn with a full GUI

Tools like Event Viewer, Group Policy, and Server Manager are more accessible

Ideal for building foundational knowledge before moving to Server Core later

What a Domain Controller Is
A Domain Controller is the identity and authentication hub of a Windows network.
It manages:

Users and passwords

Computers and groups

Authentication (Kerberos + NTLM)

Permissions and access control

Security policies

Directory services (Active Directory)

In cybersecurity, almost every attack, investigation, and detection involves the Domain Controller — so learning it properly is essential.

Why I Named It “DC01”
“DC” = Domain Controller

“01” = first server in the environment

Follows real enterprise naming conventions

Keeps the lab organised as more servers are added (DC02, WIN11-01, KALI01, SIEM01, etc.)

Installation Summary
Booted the VM using the Windows Server 2022 ISO

Selected Windows Server 2022 Standard Evaluation (Desktop Experience)

Chose Custom installation

Installed onto the empty virtual disk

Waited for Windows to copy files and reboot

Set the Administrator password

Logged into the Windows Server 2022 desktop

What DC01 Will Become
DC01 is the foundation of my entire homelab. It will act as:

The identity provider for all users and machines

The authentication authority (Kerberos + NTLM)

The DNS server for internal networking

The central point for Group Policy

The main source of logs for Defender and Sentinel

The target for attack simulations from Kali

The core system for learning detection engineering, Windows security, incident investigation, and enterprise networking