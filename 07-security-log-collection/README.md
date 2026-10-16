# 07 — Security Log Collection into Sentinel

## Overview
With the audit policy fixed (stage 06), DC01 was recording the right security events but not sending them anywhere. This stage adds the Windows Security log to the data collection rule, then checks the events arrive in Sentinel and learns to read them. These were my first real KQL queries against logon data.

## Why
- **Three separate switches must all be on** for an event to appear in Sentinel: Windows must record it (audit policy), the agent must be told to ship it (data collection rule), and the query must actually find it. When data is missing, the job is to work out which switch is off.
- **The rule ships only what it lists.** Sysmon needed its own entry for this reason, and Security does too.
- **Filter by event ID instead of collecting the whole log.** A DC's Security log is very noisy. Naming specific IDs keeps the data useful and the ingestion cost low once the free trial ends.

## Steps

### 1. Added a Security XPath to the rule
Azure portal → Monitor → Data Collection Rules → `aslab-dcr-dc01` → Data sources → Windows Event Logs → Custom. Alongside the Sysmon entry, added:

```
Security!*[System[(EventID=4624 or EventID=4625 or EventID=4672 or EventID=4720 or EventID=4768 or EventID=4769 or EventID=4771 or EventID=4776)]]
```

What each ID gives me:

| Event ID | Meaning |
|---|---|
| 4624 / 4625 | Successful / failed logon |
| 4672 | Privileged logon |
| 4720 | User account created |
| 4768 / 4771 | Kerberos ticket requested / pre-authentication failed |
| 4769 | Service ticket requested (relevant to Kerberoasting later) |
| 4776 | NTLM credential validation |

The rule is the central instruction sheet. Changing it here updates the agent on DC01 automatically, and any machine I attach later gets the same list.

### 2. Generated test events
Signed out of DC01, typed a wrong password once for `Administrator`, then signed back in correctly.

### 3. Checked the ground truth on DC01 first
Before blaming the pipeline, I asked Windows directly what it had recorded in the last 15 minutes:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; StartTime=(Get-Date).AddMinutes(-15)} | Group-Object Id | Sort-Object Count -Descending | Select-Object Count, Name
```

This showed one 4625 and one 4771 (plus many 4624s), so Windows had recorded the failure.

### 4. Counted events by ID in Sentinel
`take 10` returns any ten matching rows, so ten 4624s proved nothing about whether a 4625 existed. A count covers all the data:

```kusto
Event
| where EventLog == "Security"
| summarize count() by EventID
```

An early result showed 4624 (43), 4672 (41), 4769 (5), 4776 (5), 4771 (4) and 4768 (3). 4720 was absent because no account had been created yet.

### 5. Confirmed the failed logon end to end

```kusto
Event
| where EventLog == "Security" and EventID in (4625, 4771)
| project TimeGenerated, EventID, RenderedDescription
| sort by TimeGenerated desc
```

Both a 4625 and a 4771 arrived at 22:39:13 UTC, matching the wrong password I typed. That proved all three switches for failed logons.

### 6. Read what the events say
**4771** (Kerberos pre-authentication failed): Account Name `Administrator`; Service Name `krbtgt/ASLAB` (asking for the first ticket); Client Address `::1` (IPv6 for this machine itself); Failure Code `0x18` (wrong password); Pre-Authentication Type 2 (password-based).

**4625** (failed logon): the Subject is `DC01$`, the machine reporting the failure, not the person. The guessed account is under "Account For Which Logon Failed". Logon Type 2 is an interactive sign-in. Status `0xC000006D` is the generic failure and Sub Status `0xC000006A` is the real reason (right username, wrong password; a non-existent username would give `0xC0000064`). Source address `127.0.0.1` and logon process `User32` show it came from the local sign-in screen.

## Notes / Lessons Learned
- **A failure can leave two events.** A wrong password at the sign-in screen produces a 4625 from the logon process and a 4771 from the Kerberos service. For password guessing on a DC, 4771 is the stronger signal, and the failure code and client address show what kind of failure it was and where it came from.
- **Where the event is written depends on the event.** The DC writes the credential check (4771, 4776). The machine where someone sits writes the logon failure (4625). With DC01 as my only source, I see only the DC's side for now.
- **Security events land in the `Event` table with details packed in XML,** not in separate columns. Parsing that XML into columns is the next KQL skill.
- **The agent collects forward only.** Adding an ID to the rule does not bring back events recorded earlier.
- **Sentinel shows UTC; Event Viewer shows DC01's local time.** Convert before comparing timestamps.
- **Not yet collected:** 4719 (audit policy changed) is recorded on DC01 but not in the XPath, so it never reaches Sentinel. To add it, edit the existing Security line instead of adding a second one.
- Troubleshooting from this stage: [empty-kql-results-timing-and-timezone](../troubleshooting/empty-kql-results-timing-and-timezone.md), [event-viewer-stale-filtered-view](../troubleshooting/event-viewer-stale-filtered-view.md), and [audit-policy-wiped-logon-auditing](../troubleshooting/audit-policy-wiped-logon-auditing.md)
