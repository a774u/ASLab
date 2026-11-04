# 11 — Account Lifecycle Events: Test User, Group and Widening Security Collection

## Overview
I created a test user and a test group on DC01, added the user to the group and disabled the account, noting the clock time of each action. I then checked whether Windows recorded the resulting events and whether they reached Sentinel. Windows recorded them all, but Sentinel only received the account creation (4720). The cause was that the data collection rule's Security XPath did not list the other event IDs. I fixed it by editing the existing XPath line, then repeated the actions to confirm the events now arrive.

## Why
- **A SIEM can only detect what it receives.** Account creation, group membership changes and account enable/disable are the events that show an account being created or given more access. If they never reach Sentinel, those changes are invisible.
- **A known timeline gives a controlled test.** Writing down the time of each action gives me known answers to check against, the same habit as [entry 07](../07-security-log-collection/README.md).
- **It practises the three switches on a new problem:** (1) Windows records it, (2) the agent ships it, (3) my query finds it.

## Steps

### 1. Created the test user
In Active Directory Users and Computers (`dsa.msc`), in the Users container: New → User. Name Test User1, logon name `testuser1`, strong password, account left enabled, no admin rights (Domain Users only). Time: 11:31. I kept it an ordinary account on purpose, because ordinary accounts are what attackers spray and abuse, and a made-up name is easy to search for in logs.

### 2. Created the test group
New → Group, named `TestGroup1`, scope Global, type Security. Time: 11:40. A security group can have permissions attached, which is why attackers want to join them. A distribution group is only for email.

### 3. Added the user to the group
TestGroup1 → Members tab → Add → `testuser1`. Time: 11:44. Groups are how permissions get attached to people.

### 4. Disabled the account
Right-click Test User1 → Disable Account. Time: 11:48. Disabling blocks logons without deleting anything, which keeps the evidence and the SID.

### 5. Checked switch 1: did Windows record it?
Elevated PowerShell on DC01, counting Security events by ID for a window around my actions:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; StartTime=[datetime]'2026-11-04 11:25'; EndTime=[datetime]'2026-11-04 11:55'} | Group-Object Id | Sort-Object Count -Descending | Select-Object Count, Name
```

A second query hid the noisy logons and showed the time of each remaining event, so I could match them to my clock times:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; StartTime=[datetime]'2026-11-04 11:25'; EndTime=[datetime]'2026-11-04 11:55'} | Where-Object { $_.Id -notin 4624,4672 } | Sort-Object TimeCreated | Select-Object TimeCreated, Id | Format-Table -AutoSize
```

Windows recorded events matching all four actions:

| Action | Time | Events Windows recorded |
|---|---|---|
| Create user | 11:31 | 4720, 4738, 4724, 4722 (plus 4769) |
| Create group | 11:40 | 4727 |
| Add user to group | 11:44 | 4728, 4737 |
| Disable account | 11:48 | 4738, 4725 |

### 6. Checked Sentinel (switches 2 and 3)
Counted Security events by ID for the same window:

```kusto
Event
| where TimeGenerated between (datetime(2026-11-04 11:25:00) .. datetime(2026-11-04 11:55:00))
| where EventLog == "Security"
| summarize count() by EventID
| sort by count_ desc
```

Sentinel showed 4672 (107), 4624 (106), 4769 (2), 4768 (1) and 4720 (1). Those counts matched Windows exactly, so nothing was lost or duplicated for IDs already in the XPath. The IDs 4722, 4724, 4725, 4727, 4728, 4737 and 4738 were missing.

### 7. Fixed switch 2
In the data collection rule (Monitor → Data Collection Rules → `aslab-dcr-dc01` → Data sources → Windows Event Logs) I edited the **existing** Security XPath line in place to add new IDs. I did not add a second line. The line now collects:

```
Security!*[System[(EventID=4624 or EventID=4625 or EventID=4672 or EventID=4719 or EventID=4720 or EventID=4722 or EventID=4724 or EventID=4725 or EventID=4726 or EventID=4727 or EventID=4728 or EventID=4729 or EventID=4732 or EventID=4740 or EventID=4768 or EventID=4769 or EventID=4771 or EventID=4776)]]
```

I left out 4737 and 4738 on purpose (see notes).

### 8. Verified with fresh actions
The agent collects forward only, so I repeated actions after the change and queried for them:

```kusto
Event
| where TimeGenerated > ago(2h)
| where EventLog == "Security"
| where EventID in (4722, 4725, 4727, 4728, 4729, 4732, 4740)
| project TimeGenerated, EventID, RenderedDescription
| sort by TimeGenerated asc
```

Sentinel shows UTC in day/month/year order, and the UK is on GMT, so these match DC01's clock:
- 15:49:47: removed the user from the group, event 4729.
- 15:50:00: added the user back to the group, event 4728.
- 16:12:03: enabled the account, event 4722.
- 16:12:19: disabled the account, event 4725.
- 16:13:12: enabled the account again, event 4722.

The account ended the session enabled. I also pulled the raw EventData of the 4728 (`Event | where EventID == 4728 | take 1 | project EventData`), which came back with the capture time 15:50:00 UTC matching the row above.

## Notes / Lessons Learned
- **The fault was in switch 2.** Windows recorded the events, the agent was healthy and my query was fine (it found the events that were in the XPath). The XPath simply did not list the other IDs.
- **Compare counts per ID between the source and the SIEM.** Equal counts mean the path works for that ID (recorded once, shipped once, found). A higher Sentinel count means duplicates, usually overlapping XPath lines ([duplicate-security-events](../troubleshooting/duplicate-security-events.md)). A whole ID missing points at the XPath. A partial shortfall points at timing, the agent being down, or events that predate the rule.
- **Edit the existing XPath line. Never add an overlapping second line,** or events ship twice.
- **Collection is forward only.** Events from before the edit never arrive, so I had to repeat the actions.
- **Creating one account produced a cluster of events** (4720 plus 4738, 4724 and 4722 within about a second). One click in the console is not one event.
- **4737 and 4738 were left out deliberately.** They fire in bursts and add volume and cost for less detection value. I can add them later.