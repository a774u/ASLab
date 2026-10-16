# Empty KQL Results for 4625 Despite Events in Event Viewer

## Overview
A query for 4625 events in Sentinel returned "No results found" even though Event Viewer on DC01 listed 17 of them. Two things looked like possible causes (the time zone, and when the events happened). This entry records how I separated them.

## Symptom
```kusto
Event
| where EventLog == "Security" and EventID == 4625
| take 10
```
returned no results for the last 24 hours, while Event Viewer showed 4625s on DC01.

## Evidence
- The newest 4625s in Event Viewer were stamped 2:35 PM. The Group Policy screenshots were taken at 2:56 to 2:57 PM, so those failures came before the audit policy change, and the data collection rule was edited after that.
- Other Security events (4624, 4672, 4768, 4769, 4771, 4776) were arriving through the same XPath line, so the line itself was working.
- After I changed DC01's time zone, the same two 4625 events displayed as 10:35 PM instead of 2:35 PM. The events were unchanged and only the display had moved.

## Cause
- **The agent collects forward only.** Events written before the rule included that ID were never going to be shipped.
- **The time zone was not the cause.** Windows stores event times in UTC and converts them for display. A wrong time zone changes how times look, not whether events exist or ship. (A wrong *clock* is a different problem: Kerberos rejects clocks more than about 5 minutes apart.)

## Fix
A clean test with no timing ambiguity: a wrong password at the sign-in screen after the rule was saved, then wait about 10 minutes and query. At the time I did this, no 4625 appeared, which led to a second, separate cause: see [audit-policy-wiped-logon-auditing](audit-policy-wiped-logon-auditing.md). I also set DC01's time zone to UK time so what I see on the machine is easier to match against Sentinel's UTC.

## Notes / Lessons Learned
- **When a query returns nothing, check three points in order:** was the event recorded (audit policy and Event Viewer or `Get-WinEvent`), was it shipped (the rule and the time it was edited), and does the query find it (time range, filters).
- **Start at the source and work outward.** If Windows never recorded the event, no later step can find it.
- **`take 10` is a sample, not proof.** Ten 4624s said nothing about whether 4625s existed. A `summarize count() by EventID` covers all the data.
- **A plausible explanation can still be the wrong one.** The timing theory was reasonable and partly true for the early events, but it did not explain the 4625s that kept going missing later.
