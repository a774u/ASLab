## Overview
Right after demoting DC01 out of the original `aydenlab.local` forest (as part of the domain rename to `ASLab.internal`), Server Manager and the Add Roles wizard were still showing the server as `DC01.aydenlab.local` — even though the demotion had actually completed. This documents figuring out whether that meant the demotion had failed, or something else.

## The Symptom
- After running "Demote this domain controller" and rebooting, re-running Add Roles and Features still displayed **`DC01.aydenlab.local`** as the destination server name
- This looked identical to what "the demotion didn't actually work" would look like

## Investigating
Rather than assuming the demotion had failed, checked the actual domain-membership state directly via the classic System Properties dialog (`sysdm.cpl`, not the modern Settings app — the modern About page doesn't show this clearly):

- **Workgroup: WORKGROUP** — confirmed the demotion had genuinely succeeded; the server was no longer domain-joined at all
- **Full computer name: DC01.aydenlab.local** — the old DNS suffix was still attached to the adapter despite that

## Root Cause
Demoting a domain controller removes it from the domain, but doesn't always clear the **Primary DNS suffix** configured on the network adapter — that field is a separate, independent setting from actual domain membership. Server Manager was just displaying that stale leftover value, not an indication that anything had gone wrong.

## Fix
1. `sysdm.cpl` → Computer Name tab → **Change...** → **More...**
2. Cleared the **Primary DNS suffix of this computer** field entirely
3. Applied, restarted
4. Confirmed **Full computer name** now showed just `DC01` with no suffix, before proceeding with re-promotion into the new forest

## Notes / Lessons Learned
- Domain membership and DNS suffix are two separate pieces of state that can drift out of sync — don't assume a leftover suffix means a failed demotion, but don't ignore it either, since carrying a stale suffix into a freshly-promoted new forest could cause confusing naming issues later.
- The classic `sysdm.cpl` dialog remains the reliable way to check actual domain-join state on Server — the modern Settings app doesn't surface it as directly.