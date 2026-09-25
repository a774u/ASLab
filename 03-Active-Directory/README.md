Overview
This section covers installing Active Directory Domain Services (AD DS) and promoting DC01 into a fully functioning Domain Controller. This is the moment the homelab becomes a real enterprise environment, with identity, authentication, DNS, and Group Policy all handled centrally.

What Active Directory Actually Does
Active Directory is the backbone of Windows enterprise networks. It provides:

Centralised identity management

Authentication (Kerberos + NTLM)

Computer and user accounts

Group Policy

Directory services

DNS integration

Once AD is installed, DC01 becomes the authority for everything in the network — logons, permissions, policies, and internal name resolution.

Why My Domain Is “aydenlab.local”
I chose aydenlab.local because:

It’s clean and professional

It follows real enterprise naming patterns

It clearly identifies the environment as a lab

It avoids conflicts with real internet domains

This domain will be the foundation for all future machines, users, policies, and security configurations.