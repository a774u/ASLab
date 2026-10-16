# 05 — Azure and Sentinel Integration

## Overview
Sysmon on DC01 was already generating security telemetry, but nothing was reading it. This stage builds the pipeline that carries those events to Microsoft Sentinel, where they can be searched with KQL and eventually used for detections.


## Why This Architecture
- **Log Analytics workspace** is the actual database. Sentinel stores nothing itself; it is a security layer (detections, incidents, investigation) switched on over a workspace.
- **Azure Arc** is needed because DC01 is a VMware VM on my own hardware, so Azure has no record of it. Arc registers it as an Azure resource, which gives other Azure services something to attach to. This hybrid pattern is common in real environments with on-prem servers.
- **Azure Monitor Agent (AMA)** does the collecting and sending. Arc provides identity; AMA provides the log shipping. They are separate components.
- **Data Collection Rule (DCR)** defines what to collect and where to send it. One rule can apply to many machines, which is how this scales.

## Steps

### 1. Azure account
Created a free Azure account on my host machine (not inside the VM). The portal is the control plane, and DC01 is only a data source. The account includes a $200 credit for 30 days.

### 2. Resource group
Created `ASLab-RG` in UK South. Every Azure resource must sit in a resource group, and keeping the lab in one makes it easy to remove cleanly later.

### 3. Log Analytics workspace
Created `ASLab-LAW` in `ASLab-RG`, UK South. This is where logs are stored and queried.

### 4. Enabled Microsoft Sentinel
Added Sentinel to `ASLab-LAW`. There was no "create new" step because it layers onto the existing workspace. The portal showed a 31-day Sentinel free trial (up to 10 GB/day for both ingestion and analysis). It also showed that Sentinel is moving into the unified Microsoft Defender portal.

### 5. Onboarded DC01 to Azure Arc
Azure Arc → Machines → Onboard → single server. Settings: `ASLab-RG`, UK South, Windows, SQL Server discovery unchecked (DC01 has no SQL Server and shouldn't), public endpoint connectivity. Ran the generated PowerShell script in an elevated window on DC01 and signed in through the browser flow. DC01 then appeared in Arc as Connected, and `azcmagent show` confirmed Agent Status: Connected.

### 6. Created the Data Collection Rule
Created `aslab-dcr-dc01`, attached DC01 as the resource, set the data source type to **Windows Event Logs**, and set the destination to `ASLab-LAW`. Sysmon logs to its own channel, separate from the standard Windows logs, so it has to be added explicitly as a custom XPath entry: `Microsoft-Windows-Sysmon/Operational!*`. The `!*` means all events from that channel. Without it, Sysmon data keeps being generated locally but never ships. Attaching the rule installs the Azure Monitor Agent on DC01 through Arc.

### 7. Verified data arrived
In the Defender portal's Advanced hunting (Azure Portal Logs also works), with the time range on Last 24 hours: 
Heartbeat
| where Category == "Azure Monitor Agent"
| take 10

Event
| where Source == "Microsoft-Windows-Sysmon"
| take 10

Heartbeat rows confirmed the agent was talking to the workspace, and Sysmon rows confirmed the telemetry was flowing end to end. These were the first KQL queries I ran against my own data.

## Notes / Lessons Learned
- The data collection rule writes directly into the workspace, so Sentinel's data connector page can show "0 connectors" while data is arriving. The connector gallery is a different path.
- The orange banner about the Microsoft Monitoring Agent retiring does not apply here, since this lab uses the newer Azure Monitor Agent.
- Cost awareness: the $200 credit lasts about 30 days (it showed about £150 remaining with £0.00 spent). The Sentinel trial covers roughly the same window, and a single lab DC's log volume is small. Plan: upgrade to Pay-As-You-Go before the credit ends and set a small budget alert so unexpected ingestion shows up quickly.
- Troubleshooting for this stage: [ama-heartbeat-troubleshooting.md](../troubleshooting/ama-heartbeat-troubleshooting.md)