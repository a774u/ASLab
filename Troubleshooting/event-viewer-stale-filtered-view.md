# Event Viewer Showed No New 4625 Events

## Overview
After fixing the audit policy, a wrong-password test appeared to produce no 4625 in Event Viewer. The event had been recorded, and the filtered view was out of date.

## Symptom
Event Viewer's Security log, filtered to 4625 and 4771, showed no entry for the attempt I had just made.

## Evidence
Instead of trusting the window, I asked Windows directly for the last 15 minutes of the Security log:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; StartTime=(Get-Date).AddMinutes(-15)} | Group-Object Id | Sort-Object Count -Descending | Select-Object Count, Name
```

The result included one 4625, one 4771 and 31 4624s. The failed logon had been recorded, and the successful sign-in's 4624 showed that logon auditing was working.

## Cause
Event Viewer's list is what the log held when it was last opened or refreshed. A filtered view does not update itself.

## Fix
Refresh the log (F5 or Action → Refresh). The filter stays applied and the new entries appear.

## Notes / Lessons Learned
- **Don't treat a stale view as evidence that something wasn't recorded.** The same view had misled me once before in this lab.
- **`Get-WinEvent` is a quick source of ground truth** for what Windows recorded in a time window, independent of the Event Viewer interface.
- **A successful logon is a useful control.** If the 4624 for my own sign-in is present, logon auditing is working, so a missing failure event is about the attempt or the pipeline, not about auditing being off.
- **Event Viewer shows local time and Sentinel shows UTC,** so match events by converting, not by comparing raw numbers.
