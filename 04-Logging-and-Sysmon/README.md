## Overview
This stage transforms the homelab from a basic Active Directory environment into a security-ready enterprise network. Sysmon provides deep visibility into process creation, network connections, registry changes, and other high-value telemetry that standard Windows logging doesn't capture on its own.

The goal of this section is to install Sysmon, apply a hardened configuration, and prepare the environment for centralised log collection and SIEM ingestion later.

## Why Sysmon Matters
Sysmon (System Monitor) is part of Microsoft Sysinternals and provides detailed, high-signal security telemetry that standard Windows Event Logs don't capture by default. It enables:

- Process creation visibility
- Command-line logging
- Network connection tracking
- File creation and modification events
- Registry monitoring
- Driver and image load events

This data is essential for:

- Threat hunting
- Incident investigation
- Detection engineering
- Building KQL analytics in Sentinel
- Understanding attacker behaviour within the lab

## What We're Setting Up
This section covers:

- Installing Sysmon on DC01
- Applying the SwiftOnSecurity Sysmon config (a widely used community baseline)
- Verifying Sysmon is generating logs
- Preparing the environment for Windows Event Forwarding (WEF) — not built yet, but this is the stage that lays the groundwork for it
- Ensuring logs are in a shape ready for Defender and Sentinel ingestion once those stages are built