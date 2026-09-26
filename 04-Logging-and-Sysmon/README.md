This stage transforms the homelab from a basic Active Directory environment into a security‑ready enterprise network. Sysmon provides deep visibility into process creation, network connections, registry changes, and other high‑value telemetry used by SOC analysts and detection engineers.

The goal of this section is to install Sysmon, apply a hardened configuration, and prepare the environment for centralised log collection and SIEM ingestion.

Why Sysmon Matters
Sysmon (System Monitor) is part of Microsoft Sysinternals and provides detailed, high‑signal security telemetry that standard Windows logs do not capture. It enables:

Process creation visibility

Command‑line logging

Network connection tracking

File creation and modification events

Registry monitoring

Driver and image load events

This data is essential for:

Threat hunting

Incident investigation

Detection engineering

Building KQL analytics in Sentinel

Understanding attacker behaviour in your lab

What We’re Setting Up
This section covers:

Installing Sysmon on DC01

Applying the SwiftOnSecurity Sysmon config (industry standard)

Verifying Sysmon is generating logs

Preparing the environment for Windows Event Forwarding (WEF)

Ensuring logs are ready for Defender and Sentinel ingestion