## VM Specs
- OS: Windows Server 2022 Standard Evaluation
- CPU: 2 vCPUs
- RAM: 4 GB
- Disk: 80 GB
- Network: VMware NAT (vmnet8)
- ISO: Windows Server 2022 installation media

## Why These Specs
- 2 vCPUs + 4 GB RAM is enough headroom for AD, DNS, and basic services at this stage
- 80 GB disk leaves room for logs, policies, and future tooling without needing to resize early
- **NAT** rather than Host-Only was the deliberate choice here, specifically because DC01 needs outbound internet access later — for Windows Update, and critically, for shipping logs to Microsoft Sentinel once that stage of the lab is built. Host-Only would have kept the lab fully isolated, but at the cost of that connectivity.
- The evaluation ISO is the right call for a learning environment — no licensing cost while concepts are still being learned

## Future Changes
As the lab grows, I expect to increase:
- **RAM** — Defender and Sentinel-adjacent workloads (plus running more VMs concurrently) will likely need more than 4 GB
- **Disk space** — logs and SIEM-bound data accumulate fast once ingestion is actually running

## Notes / Lessons Learned
- Originally documented as Host-Only networking by mistake — corrected here after discovering (via a subnet mismatch that blocked static IP configuration) that the VM was actually running on NAT the whole time. Worth double-checking your actual VMware Virtual Network Editor settings rather than assuming, since it's easy for docs to drift from reality.