# Opened the Wrong GPO Before Changing Audit Settings

## Overview
When I went to enable failure auditing, the Group Policy editor was open on the Default Domain Policy instead of the Default Domain Controllers Policy. I caught it before changing anything.

## Symptom
The title at the top of the editor's left pane read **Default Domain Policy**, not Default Domain Controllers Policy.

## Cause
Both are named "Default ..." and sit near each other in `gpmc.msc`. The Default Domain Policy is linked to the whole domain, and the Default Domain Controllers Policy is linked to the Domain Controllers OU.

## Why it mattered
A GPO applies to whatever it is linked to. Audit settings put in the Default Domain Policy would reach every domain-joined machine, including a Windows 11 client I add later. That means extra logging I didn't intend, more noise, and higher ingestion cost once client logs are forwarded.

## Fix
Closed the editor without saving, then in `gpmc.msc` expanded `ASLab.internal` → Domain Controllers, right-clicked **Default Domain Controllers Policy** → Edit, and checked the title before continuing.

## Notes / Lessons Learned
- **Check the title at the top of the left pane every time before changing a GPO.**
- **Audit settings for a DC belong on the policy linked to the Domain Controllers OU,** so they don't spread to workstations.
- **An OU is a container for organising objects and targeting policy;** a group is a list of accounts used for permissions. A user sits in one OU but can be in many groups.
