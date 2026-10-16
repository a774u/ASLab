# 06 — Windows Audit Policy with Group Policy

## Overview
Sentinel can only search what Windows writes down. This stage checks which security events DC01 records, closes a gap (failed Kerberos and NTLM logons were not being audited), and then sets the audit policy explicitly through Group Policy so the result no longer depends on defaults I can't see.

## Why
- **Audit policy decides what Windows records.** Each subcategory can log Success, Failure, both, or nothing ("No Auditing"). An event that was never recorded can't be recovered by any query later, so this has to be right before any detection work.
- **Failures matter most for detection.** Password guessing looks like many failed logons, so a DC that only logs successful credential checks can't show it. It is like a door camera that only records when someone gets in.
- **Group Policy is the right tool.** A DC takes its settings from Group Policy and re-applies them regularly, so a local edit can be overwritten. One policy change reaches every DC, which is how it is done in real environments.
- **OUs target the policy.** A GPO only applies where it is linked. The Default Domain Controllers Policy is linked to the Domain Controllers OU, which holds DC01, so DC-specific audit rules go there and not on the whole domain.

## Steps

### 1. Checked the current audit policy
In an elevated PowerShell on DC01:

```powershell
auditpol /get /category:"Logon/Logoff","Account Logon","Account Management"
```

Before any change, the relevant rows were:

| Subcategory | Setting | Events |
|---|---|---|
| Logon | Success and Failure | 4624, 4625 |
| Special Logon | Success | 4672 |
| User Account Management | Success | 4720 and related |
| Security Group Management | Success | group changes |
| Kerberos Authentication Service | Success only | 4768 |
| Kerberos Service Ticket Operations | Success only | 4769 |
| Credential Validation | Success only | 4776 |

The gap was the three Account Logon rows at the bottom: only successes were recorded, so failed Kerberos and NTLM authentication would leave no trace.

### 2. Opened the correct GPO
Ran `gpmc.msc`, expanded `ASLab.internal` → Domain Controllers, and edited **Default Domain Controllers Policy**. I first opened the Default Domain Policy by mistake and caught it from the title bar before changing anything (see [wrong-gpo-edited](../troubleshooting/wrong-gpo-edited.md)).

### 3. Turned on failure auditing for Account Logon
Computer Configuration → Policies → Windows Settings → Security Settings → Advanced Audit Policy Configuration → Audit Policies → Account Logon. For Credential Validation, Kerberos Authentication Service and Kerberos Service Ticket Operations: tick **Configure the following audit events**, then tick Success and Failure.

The first tick matters. Without it the policy has no opinion on that setting and the other ticks are ignored.

### 4. Applied and checked

```powershell
gpupdate /force
auditpol /get /category:"Account Logon"
```

All three rows changed from Success to **Success and Failure**.

### 5. Found a side effect and set everything explicitly
Failed logons then stopped producing 4625 events (full write-up: [audit-policy-wiped-logon-auditing](../troubleshooting/audit-policy-wiped-logon-auditing.md)). Running `auditpol /get /category:*` showed that only the three Account Logon rows I had configured were auditing anything. Logon, Special Logon, User Account Management and everything else said No Auditing.

I fixed it by configuring the subcategories my detections depend on explicitly, in the same GPO:

| Category | Subcategory | Setting | Events |
|---|---|---|---|
| Logon/Logoff | Audit Logon | Success and Failure | 4624, 4625 |
| Logon/Logoff | Audit Special Logon | Success | 4672 |
| Account Management | Audit User Account Management | Success and Failure | 4720 and related |
| Account Management | Audit Security Group Management | Success | group membership changes |
| Policy Change | Audit Policy Change | Success | 4719 |
| Account Logon | Credential Validation, Kerberos Authentication Service, Kerberos Service Ticket Operations | Success and Failure | 4776, 4768/4771, 4769 |

After `gpupdate /force`, `auditpol /get /category:*` showed exactly these rows enabled and everything else still at No Auditing.

## Notes / Lessons Learned
- **Account Logon and Logon/Logoff are different things.** Account Logon is authentication: a DC checking credentials and issuing Kerberos tickets. Logon/Logoff is sessions: a machine creating a logon session after authentication.
- **A setting left at Not Configured is fragile.** I assumed unconfigured subcategories would fall back to Windows defaults. After the GPO defined Advanced Audit Policy, they showed No Auditing instead. Set every subcategory a detection depends on explicitly.
- **4719 is worth auditing.** It records changes to the audit policy itself, which is how switching off logging would be noticed.
- **Deliberately left off:** Detailed Tracking (process creation, 4688) because Sysmon already records process creation in more detail; Object Access and Privilege Use because they are very noisy on a DC. DS Access is for later, when I reach AD attack techniques such as DCSync.
- **Editing the Default Domain Controllers Policy is fine for a lab.** Real environments usually create a separate GPO linked to the Domain Controllers OU so the defaults stay untouched.
- Next stage: [07 — Security log collection](../07-security-log-collection/07-security-log-collection.md)
