## Overview
This step installs the Active Directory Domain Services role on DC01. Important distinction worth understanding: installing the role does **not** make the server a Domain Controller — it just makes the AD DS capability available. Promotion (the next document) is the step that actually creates the forest, builds the domain, and turns the role into a running service.

## Steps
1. Open Server Manager (launches automatically on login; otherwise from the Start menu)
2. Click **Manage → Add Roles and Features**
3. Choose **Role-based or feature-based installation**
4. Select DC01 as the target server
5. Tick **Active Directory Domain Services**, accept the additional required features it pulls in automatically (these are dependencies AD DS needs to function, not optional extras)
6. Click through the wizard, then **Install**

## Notes / Lessons Learned
- Nothing changes on the network or in how DC01 behaves at this point — it's purely a software installation. The actual behavioural shift happens entirely in the promotion step.