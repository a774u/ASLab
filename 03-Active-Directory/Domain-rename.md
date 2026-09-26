### Overview
After initially building the domain as `aydenlab.local`, I made the decision to rename it to `ASLab.internal` before building anything further on top of it. This section documents why, and the process of tearing down and rebuilding the domain to get there.

### Why the Rename
`.local` is commonly used in tutorials, but it's not actually a safe choice for a real (or realistic) Active Directory domain:

- `.local` is reserved by IANA for **multicast DNS (mDNS/Bonjour)**, which can cause genuine name-resolution conflicts on networks with Apple or zeroconf devices
- Domains using unregistered TLDs like `.local` can't get valid SSL certificates for that namespace, which blocks future integration with things like ADFS or hybrid Azure AD setups
- `.internal` is an **IANA-reserved TLD specifically intended for private, non-routable namespaces** like this one — it gets the isolation benefit of `.local` without the downsides

Since I caught this early — before any GPOs, additional users, or dependent config existed — a clean rebuild was far cheaper than doing it later, and Microsoft's actual domain-rename tooling (`rendom.exe`) is intended for large, established domains that can't be rebuilt, not lab environments like this one.

### What I Did

1. **Attempted to remove the AD DS role directly** via Server Manager → Remove Roles and Features. This was correctly blocked — Windows validated that DC01 was still an active domain controller and refused to let the role be stripped out from under it, pointing me instead to **"Demote this domain controller."**
2. **Ran the proper demotion wizard** (Server Manager → AD DS → right-click DC01 → Demote this Domain Controller), rather than the generic role-removal path. Since DC01 was the last (and only) DC in the forest, the wizard flagged this and offered to remove the domain entirely — selected that option.
3. **Rebooted and confirmed DC01 was back to a standalone Workgroup member**, with the old `aydenlab.local` forest fully gone.
4. **Re-added the AD DS role** the same way as the original build.
5. **Promoted DC01 as a new forest**, this time entering `ASLab.internal` as the root domain name, and set a fresh DSRM (Directory Services Restore Mode) recovery password.
6. **Rebooted and verified** the new domain with `Get-ADDomain`, confirming `ASLab.internal` was live.

### Notes / Lessons Learned
- Sysmon, being host-level rather than domain-level, was completely unaffected by this — no reinstall needed.
- Windows stores and displays DNS domain names in lowercase regardless of how they're typed at creation (`ASLab.internal` → `aslab.internal`) — expected behaviour, not an error.
- Left DNS on DC01 pointed at `8.8.8.8` throughout this process rather than switching to self (`127.0.0.1`), since there was no point enabling self-DNS before the new domain existed to serve. That switch — plus configuring a DNS Forwarder — comes next, now that `ASLab.internal` is up.