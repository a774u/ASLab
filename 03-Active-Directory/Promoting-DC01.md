Promoting DC01 to a Domain Controller
1. Start the Promotion Wizard
After installation, Server Manager shows a yellow notification.
Click it and select:

“Promote this server to a domain controller”

2. Choose “Add a new forest”
Because this is the first Domain Controller in the environment:

Select Add a new forest

Enter the root domain name:

Code
aydenlab.local
3. Domain Controller Options
The wizard will ask for:

Forest Functional Level: Windows Server 2016

Domain Functional Level: Windows Server 2016

DNS Server: Enabled

Global Catalog: Enabled

Read-only Domain Controller: Disabled (correct for first DC)

These defaults are ideal for a homelab.

4. Set the DSRM Password
Choose a strong Directory Services Restore Mode password.
This is used for recovery scenarios.

5. DNS Options
You may see a warning about DNS delegation — this is normal for a first DC.
Click Next.

6. Paths
Leave the database, log, and SYSVOL paths at their defaults.

7. Review Options
The wizard will show a summary of your configuration:

New forest: aydenlab.local

NetBIOS name: AYDENLAB

DNS + Global Catalog enabled

Everything here should match your intended setup.

8. Prerequisites Check
The wizard will run a validation check.
You should see:

Green checks

Yellow warnings (normal)

No red errors

Once complete, click Install.

9. Automatic Reboot
The server will:

Configure AD DS

Install DNS

Create the forest

Create the domain

Promote itself

Restart automatically

After reboot, DC01 is officially the Domain Controller for aydenlab.local.

Logging In After Promotion
You now log in using domain credentials:

Code
AYDENLAB\Administrator
This confirms the domain is active and DC01 is functioning as the identity provider.