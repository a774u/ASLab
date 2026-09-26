## Overview
This section covers why I'm using Windows Server 2022, what a Domain Controller actually is, and how I installed the OS for my homelab. The goal is a realistic enterprise environment that can support Active Directory, Defender, Sentinel, and future attack/detection work.

## Why Windows Server 2022
- It's the current enterprise standard
- Supports Active Directory, DNS, and Group Policy natively
- Integrates cleanly with Microsoft Defender and Sentinel
- Matches what real SOCs and IT teams actually run today

## Why Desktop Experience (not Server Core)
Server Core is the leaner, more "production-correct" choice long-term — smaller attack surface, no GUI overhead. I chose Desktop Experience anyway because I'm still building foundational knowledge: tools like Event Viewer, Group Policy Management, and Server Manager are far easier to learn with a full GUI in front of me. The plan is to move to Server Core later once the underlying concepts aren't new anymore.

## What a Domain Controller Actually Is
A Domain Controller is the identity and authentication hub of a Windows network. It manages:

- Users and passwords
- Computers and groups
- Authentication (Kerberos + NTLM)
- Permissions and access control
- Security policies
- Directory services (Active Directory itself)

Almost every attack, investigation, and detection scenario in cybersecurity touches the Domain Controller in some way — so understanding it properly, not just standing it up, is foundational to everything else in this lab.

## Why I Named It "DC01"
- "DC" = Domain Controller, "01" = first server in the environment
- Follows real enterprise naming conventions rather than ad-hoc names
- Keeps the lab organised as more machines get added (DC02, WIN11-01, KALI01, SIEM01, etc.) — deciding this now avoids a messy rename later

## Installation Summary
1. Booted the VM from the Windows Server 2022 ISO
2. Selected Windows Server 2022 Standard **Evaluation** (Desktop Experience)
3. Chose Custom installation, onto the empty virtual disk
4. Waited for file copy + reboot
5. Set the Administrator password
6. Logged into the desktop

## What DC01 Will Become
DC01 is the foundation of the entire homelab. Over time it's set to act as:

- The identity provider for all users and machines
- The authentication authority (Kerberos + NTLM)
- The DNS server for internal networking
- The central point for Group Policy
- The main source of logs for Defender and Sentinel
- The target for attack simulations from Kali
- The core system for learning detection engineering, Windows security, incident investigation, and enterprise networking

## Notes / Lessons Learned
- **Evaluation edition has a clock.** Standard Evaluation typically expires around 180 days, after which services start shutting down. Not a problem now, but a decision I need to make before then: convert to licensed/rearm, or plan to rebuild.
- **Single DC ≠ production pattern.** In a real environment, you'd never want one server acting as your only DC, DNS server, and (later) log-ingestion point — that's a single point of failure, and stacking that many roles on one box is generally avoided once you're past lab scale. Fine for learning purposes here, but worth being explicit that this is a simplification, not the "correct" design.