## Overview
This section covers installing Active Directory Domain Services (AD DS) and promoting DC01 into a fully functioning Domain Controller. This is the point where the homelab stops being "a Windows Server VM" and becomes a real enterprise environment — identity, authentication, DNS, and Group Policy all handled centrally from one place.

## What Active Directory Actually Does
Active Directory is the backbone of Windows enterprise networks. It provides:

- Centralised identity management
- Authentication (Kerberos + NTLM)
- Computer and user accounts
- Group Policy
- Directory services
- DNS integration

Once AD is installed, DC01 becomes the authority for everything on the network — logons, permissions, policies, and internal name resolution all route through it.

## Why the Domain Is `ASLab.internal`
- `.internal` is an IANA-reserved TLD specifically intended for private, non-routable namespaces like this one
- It gives the isolation benefit of a "fake" internal-only domain without the real-world downsides of an unregistered TLD (no mDNS conflicts, no blocked cert issuance if this ever needs to integrate with something like ADFS or hybrid Azure AD later)
- `ASLab` clearly identifies the environment as mine and as a lab, while staying professional enough to mirror real enterprise naming patterns

This domain is the foundation for every machine, user, policy, and security configuration that gets built from here on — everything downstream (WEF, GPOs, Sentinel workspace naming) inherits this namespace.