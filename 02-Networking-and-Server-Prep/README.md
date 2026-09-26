## Overview
This section covers the network configuration required before DC01 could be promoted to a Domain Controller. Setting a static IP and giving the server a proper identity ensures the environment behaves predictably and mirrors how real organisations structure their networks — a DC's whole job depends on other machines being able to find it reliably.

## Why Domain Controllers Need Static IPs
A Domain Controller provides critical services like authentication and DNS. For those to work reliably, clients need to know exactly where the DC is, every time. Static IPs ensure:

- The server's address never changes
- DNS records stay consistent
- Authentication requests don't silently fail
- The network remains stable and predictable

A Domain Controller effectively cannot function properly with a dynamic IP — DHCP-assigned addresses can change on lease renewal, which would break DNS and authentication for the whole domain.

## My IP Scheme
- IP Address: `192.168.10.10`
- Subnet Mask: `255.255.255.0`
- Default Gateway: `192.168.10.2`
- DNS Server: `8.8.8.8` (temporary — see Notes below)

## Static IP Setup (Summary)
1. Opened Control Panel → Network and Sharing Center
2. Opened the Ethernet adapter → IPv4 properties
3. Switched to "Use the following IP address"
4. Entered the static IP, subnet, gateway, and DNS values
5. Applied and saved

This gives DC01 a permanent identity on the network, rather than one that can drift.

## Notes / Lessons Learned
- **DNS is deliberately `8.8.8.8`, not self-referencing, for now.** Once AD DS's DNS Server role is installed and running, DC01 should point at itself (`127.0.0.1`) with `8.8.8.8` configured as a Forwarder instead — but flipping that switch *before* the DNS role exists and is verified just breaks all name resolution, internal and external. Sequencing this correctly (network → AD DS → self-DNS + forwarders) matters more than it looks like it should.
- **The gateway wasn't a guess — it came from a real mismatch I had to debug.** My first attempt used this same `192.168.10.x` scheme, but VMware's NAT network (`vmnet8`) was actually running on a completely different subnet (`192.168.227.0/24`) by default. DC01 sitting on `192.168.10.10` while the actual NAT network lived on `192.168.227.0/24` meant nothing could route anywhere — not a wrong-gateway problem, a wrong-subnet-entirely problem. Fixed by opening VMware's Virtual Network Editor and changing vmnet8's Subnet IP to `192.168.10.0/24` to match my intended scheme, rather than changing my docs to match VMware's default. Worth remembering: **NAT mode ≠ "any IP scheme you want" — the guest's subnet has to actually match the virtual network it's attached to.**