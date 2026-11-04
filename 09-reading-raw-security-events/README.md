# 10 — Reading Raw Security Events

## Overview
The rendered description of an event is a summary, and the raw event holds every field. This stage looks at the raw XML of a failed logon (4625) and a failed Kerberos pre-authentication (4771) in Sentinel and reads them field by field. It is the groundwork for parsing the XML into KQL columns.

## Why
- **Reading events is the basis of triage.** Who did it, from where, against which account and why it failed are all in the fields. A query or rule is only as good as my understanding of them.
- **Security events land in the `Event` table with their details packed into one XML text column,** not as separate columns. To filter on a field, I first have to know what the fields are.
- **The same sign-in can leave different events** with different fields, so I need to know which event answers which question.

## Steps

### 1. Pulled the raw EventData
```kusto
Event
| where TimeGenerated > ago(14d)
| where EventID in (4625, 4771)
| project TimeGenerated, EventID, Source, EventData
| take 5
```

The first result showed only 4771s, which proved nothing about whether 4625s existed, because `take` returns arbitrary rows. I then pulled 4625s specifically. Two came back for `take 3`, so only two existed in the window.

### 2. Learned the structure
The XML starts with a DataItem envelope added by the agent (the time is when it was captured and `sourceHealthServiceId` identifies the sending agent). The real event fields sit inside as labelled `Data Name=...` values.

### 3. Read the 4771 fields (Kerberos pre-authentication failed)
- **TargetUserName `Administrator`:** the account the attempt was for.
- **TargetSid ending in -500:** the SID is the account's permanent ID, and RID 500 is always the built-in Administrator, even if renamed.
- **ServiceName `krbtgt/ASLAB`:** asking for the first ticket, which is the sign-in itself. The krbtgt key signs every Kerberos ticket in the domain, which is why Golden Ticket attacks target it.
- **TicketOptions `0x40810010`:** hex flags describing the ticket requested, mostly relevant to advanced detection.
- **Status `0x18`:** wrong password.
- **PreAuthType `2`:** the client proved it knew the password before the DC replied, which blocks AS-REP roasting (type 0 means no pre-authentication).
- **IpAddress `::1` and IpPort `0`:** IPv6 loopback, so the request came from DC01 itself.
- **CertIssuerName, CertSerialNumber and CertThumbprint empty:** a password sign-in, not a certificate or smart card.

### 4. Read the 4625 fields (failed logon)
- **Subject** (SubjectUserSid `S-1-5-18`, SubjectUserName `DC01$`, SubjectLogonId `0x3e7`): the reporter, not the person. `S-1-5-18` is Local System, the trailing dollar sign marks a computer account and `0x3e7` is always the Local System session.
- **Target:** TargetUserSid `S-1-0-0` (a null SID, because no account was resolved when the logon failed), TargetUserName `Administrator` and TargetDomainName `ASLAB`.
- **Status `0xc000006d`** is the generic failure and **SubStatus `0xc000006a`** is the real reason (wrong password). FailureReason `%%2313` is a pointer to Windows' message text.
- **LogonType `2`** (interactive), LogonProcessName `User32` and ProcessName `svchost.exe`: the local sign-in screen.
- **AuthenticationPackageName `Negotiate`:** the layer that picks Kerberos or NTLM, which is why one sign-in produced both a 4771 and a 4625.
- **WorkstationName `DC01`, IpAddress `127.0.0.1` and IpPort `0`:** a local request.

## Notes / Lessons Learned
- **`::1` and `127.0.0.1` mean the same thing but differ as strings.** The 4771 showed the IPv6 loopback and the 4625 showed the IPv4 one, so a filter on one would miss the other.
- **Password spraying vs brute force.** Spraying is one source failing against many different accounts with the same password, which avoids lockouts. Brute force is many passwords against one account. To spot spraying, group failures by source address and count distinct target accounts.
- **Status `0x6` on a 4771 would mean the account does not exist.** A burst of those suggests username guessing, often the step before spraying.
- **A raw event is the source of truth.** If a dashboard and the raw events disagree, trust the raw events.
- **Next skill:** parse the XML into columns in KQL (`parse_xml`, `extend`, `project`) so account, source address and failure code can be filtered and counted.
- Troubleshooting from this stage: [duplicate-security-events](../troubleshooting/duplicate-security-events.md)
