## Overview
This is the step that actually creates the forest and domain, and turns DC01 from "a server with the AD DS role installed" into a real, functioning Domain Controller.

## Steps

### 1. Start the Promotion Wizard
After the AD DS role installs, Server Manager shows a notification flag. Click it and select **"Promote this server to a domain controller."**

### 2. Choose "Add a new forest"
Because this is the first Domain Controller in the environment, there's no existing forest to join — select **Add a new forest** and enter the root domain name: `ASLab.internal`

### 3. Domain Controller Options
- Forest Functional Level: Windows Server 2016
- Domain Functional Level: Windows Server 2016
- DNS Server: Enabled
- Global Catalog: Enabled
- Read-only Domain Controller: Disabled (correct for the first DC — an RODC needs a writable DC to already exist, which isn't the case here)

Windows Server 2016 is the highest functional level that exists — Microsoft hasn't introduced a newer AD functional level since, even on Server 2022 — so this isn't a downgrade, it's simply current.

### 4. Set the DSRM Password
This is a Directory Services Restore Mode password — effectively a break-glass recovery credential, separate from the domain Administrator password, used only if AD needs to be repaired from a recovery boot. Worth storing safely rather than treating as a throwaway field.

### 5. DNS Options
A warning about DNS delegation will likely appear here — this is expected and normal for a first DC in a new namespace, since there's no parent zone to delegate from yet. Click Next.

### 6. Paths
Left the database, log, and SYSVOL paths at their defaults — no reason to deviate for a single-DC lab.

### 7. Review Options
The wizard summarises the configuration before committing:
- New forest: `ASLab.internal`
- NetBIOS name: `ASLAB`
- DNS + Global Catalog enabled

Worth actually reading this screen rather than clicking past it — it's the last chance to catch a typo before it becomes permanent.

### 8. Prerequisites Check
The wizard runs a validation pass. Green checks and yellow warnings are normal and expected; red errors are not — those need resolving before Install becomes safe to click.

### 9. Automatic Reboot
From here the wizard handles everything: configuring AD DS, installing DNS, creating the forest and domain, promoting the server, and restarting automatically. After reboot, DC01 is officially the Domain Controller for `ASLab.internal`.

## Logging In After Promotion
Login now uses domain credentials: `ASLAB\Administrator`. Successfully logging in this way is the actual confirmation that the domain is live and DC01 is functioning as the identity provider — not just that the wizard reported success.

## Notes / Lessons Learned
- The distinction between "role installed" and "promoted" matters more than it seems — a lot of AD troubleshooting guides assume you know which state a server is in, and it's easy to conflate the two as a beginner.