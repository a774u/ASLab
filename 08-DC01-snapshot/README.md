# 08 — DC01 Snapshot

## Overview
After stage 07, DC01 had Active Directory, Sysmon, Azure Arc, the Azure Monitor Agent, a working audit policy and Security log collection. These took weeks to get right, so I took a VMware snapshot of DC01. If an experiment, a bad Group Policy change or an attack simulation breaks something, I can roll back in minutes instead of rebuilding.

## Why
- **A working state is worth saving.** Rebuilding would mean repeating every earlier stage. A snapshot saves the VM's exact state at one moment.
- **Experiments are coming.** Test accounts, a Kali attack VM and password spraying can all change DC01. Being able to undo them makes it safe to try things.
- **A snapshot is a rollback tool, not a backup** (see notes).

## Steps

### 1. Chose the moment
I took the snapshot once everything worked end to end: domain logons, Sysmon and Security events arriving in Sentinel, and audit policy verified. A snapshot of a half-working machine only saves the problem.

### 2. Took the snapshot in VMware Workstation
Select DC01, then VM → Snapshot → Take Snapshot (Ctrl+M), and give it a specific name. The name I used: 'AD + Sysmon + Arc + AMA + audit policy working'. A vague name like "Snapshot 1" tells me nothing in a few months, whereas a name listing what works (AD, Sysmon, Arc, AMA, audit policy) tells me what I would be going back to.

### 3. Checked it in Snapshot Manager
Snapshot Manager lists the named snapshot, with a "You are here" marker beneath it showing the current state.

## Notes / Lessons Learned
- **A snapshot is not a backup.** It lives on the same disk and depends on the original disk files. If that disk fails, both are lost.
- **Reverting rewinds DC01's own logs, but not what Sentinel already holds.** Event Viewer on DC01 goes back to the snapshot moment, while events already shipped to the workspace stay there. That is also why attackers try to clear local logs, and why logs are forwarded off the machine: the copy in Sentinel survives tampering on DC01.
- **Note any revert in write-ups.** After a revert, the timeline in Sentinel can look odd, with events from a stretch of time that no longer exists on DC01.
