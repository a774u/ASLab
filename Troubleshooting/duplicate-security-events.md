# Troubleshooting — Duplicate Security Events in Sentinel

## Overview
After adding event 4719 to the data collection rule, two 4625 rows appeared in Sentinel for one wrong-password attempt, both stamped 22:39:13.120. The cause was two overlapping Security XPath lines in the rule, so the same event was shipped twice. This came up after [stage 07](../07-security-log-collection/README.md).

## Why
- **Duplicates make counts wrong.** A rule such as "5 failures in a minute" would fire at about 3 real failures. That is a false positive, and too many of them cause alert fatigue, where real attacks get ignored.
- **A detection is only as reliable as the data feeding it.** Missing data causes false negatives, which are usually worse because nobody knows.
- **Duplicates add a little to the ingestion bill** ([entry 09](../09-cost-safeguards/README.md)).

## Steps

### 1. Noticed the duplicate
I wanted 4719 (audit policy changed) collected, because Windows recorded it but the first Security XPath did not include it. I added a **second** Security line containing 4719 instead of editing the first. Later, a wrong-password test showed two identical 4625 rows in Sentinel.

### 2. Compared the two rows
The EventData of both rows was identical in every field, including the capture time to a ten-millionth of a second, so this was one event shipped twice and not two separate events.

### 3. Counted how widespread it was
This query counted how many timestamps had one copy and how many had two:

```kusto
Event
| where TimeGenerated > ago(14d)
| where EventID in (4625, 4771)
| summarize Copies = count() by TimeGenerated, EventID
| summarize Timestamps = count() by EventID, Copies
```

For 4771, seven timestamps had one copy and one had two. For 4625, the single timestamp had two. So only some events were duplicated, not all.

### 4. Checked the data collection rule
The rule's data source screen showed **two Security XPath lines**. The second listed the same eight IDs as the first plus 4719, so every ID in the first also matched the second.

### 5. Fixed and verified
I deleted the line without 4719 and saved, leaving one Security line that includes 4719. A fresh wrong-password test showed `Copies = 1`. The old duplicated pair stays in the workspace.

## Notes / Lessons Learned
- **Each XPath entry in a rule is collected independently,** so overlapping entries ship the same event twice. The earlier events had one copy and the 22:39:13 test had two, which fits the second line being added in between. That timing fits but is not proven.
- **This was the second switch (the agent ships it).** Windows recorded each event once and the query was fine, so the fault sat between them.
- **To add an event ID, edit the existing XPath line** (or make sure new entries do not overlap) instead of adding a second line.
- **Comparing per-ID counts between Windows and Sentinel catches this.** A higher Sentinel count shows it straight away ([entry 11](../11-account-lifecycle-events/README.md)).
