# Failed Logons Stopped Producing 4625 After a GPO Change

## Overview
After enabling failure auditing for Account Logon in a GPO, wrong-password attempts still produced 4771 events but no longer produced any 4625. The GPO change had left Logon auditing switched off.

## Why this mattered
4625 is the standard failed-logon event. Silently losing it would have left a gap in every detection built on logon failures, and nothing would have looked broken.

## Symptom
- Three wrong passwords at the DC01 sign-in screen (11:22 PM local) produced three 4771s and no 4625.
- Earlier 4625s existed in Event Viewer, but the newest were from before the GPO change.

## Evidence
- Event Viewer, filtered to 4625: the newest entries were from before the GPO was edited. Every attempt after it produced only 4771 and 4776.
- `auditpol /get /category:"Logon/Logoff"` showed **No Auditing on every subcategory**, including Logon and Special Logon. Before the change, Logon was Success and Failure.
- `auditpol /get /category:*` showed that only the three Account Logon rows I had configured were auditing anything. User Account Management and Security Group Management, which had been Success, were also at No Auditing.

## Cause
My best explanation is that once the GPO defined Advanced Audit Policy, every subcategory I hadn't configured fell to No Auditing instead of keeping its earlier value. The `auditpol` output is the evidence for this; I haven't confirmed the exact mechanism. I had assumed unconfigured settings would fall back to Windows defaults, and they did not.

## Fix
Configured every subcategory my detections depend on explicitly in the Default Domain Controllers Policy (Logon, Special Logon, User Account Management, Security Group Management, Audit Policy Change, and the three Account Logon rows), then ran `gpupdate /force` and checked with `auditpol /get /category:*`. Full table in [06 — Audit policy](../06-audit-policy/06-audit-policy.md).

Verified with one wrong password at the sign-in screen: Windows recorded one 4625 and one 4771, and both reached Sentinel.

## Notes / Lessons Learned
- **Set the audit subcategories you rely on explicitly.** A setting left at Not Configured depends on defaults that can change when something else in the policy changes.
- **A total count can hide a gap.** The 4624 and 4672 counts still looked healthy after logon auditing stopped, because they included the hours before the change. Counting per hour would have shown the drop straight away:

```kusto
Event
| where EventLog == "Security" and EventID in (4624, 4625, 4672)
| summarize count() by bin(TimeGenerated, 1h), EventID
| sort by TimeGenerated asc
```

- **Check the evidence right after the change, not just the setting you edited.** Running `auditpol /get /category:*` straight after applying the GPO would have exposed this immediately.
- **The timing was the clue.** The 4625s stopped immediately after the policy change, with the same action each time and nothing else different.
