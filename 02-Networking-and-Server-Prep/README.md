Overview
This section covers the essential network configuration required before turning DC01 into a Domain Controller. Setting a static IP and giving the server a proper name ensures the environment behaves predictably and mirrors how real organisations structure their networks.

Why Domain Controllers Need Static IPs
A Domain Controller provides critical services like authentication and DNS.
For those services to work reliably, clients must always know exactly where the DC is.

Static IPs ensure:

The server’s address never changes

DNS records stay consistent

Authentication requests don’t fail

The network remains stable and predictable

A Domain Controller cannot function properly with a dynamic IP.

My IP Scheme

IP Address: 192.168.10.10

Subnet Mask: 255.255.255.0

Default Gateway: (not required for host‑only networks)

DNS Server: 127.0.0.1

Setting DNS to 127.0.0.1 means the server will use itself for DNS once Active Directory is installed — this is exactly how real enterprise networks operate.

Static IP Setup (Summary)
Opened Control Panel → Network and Sharing Center

Opened the Ethernet adapter

Opened IPv4 properties

Switched to “Use the following IP address”

Entered the static IP and DNS values

Applied and saved the configuration

This gives DC01 a permanent identity on the network.