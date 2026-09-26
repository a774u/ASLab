## Step 1 — Download Sysmon
Download from Microsoft Sysinternals directly (not a mirror or third-party host) — for a lab specifically about building good security habits, verifying you're running Microsoft's actual signed binary rather than a re-hosted copy is worth doing here rather than skipping:

- Sysmon executable
- Sysmon64 executable
- Sysmon documentation

Place the files in a tools directory on DC01, e.g. `C:\Tools\Sysmon`

## Step 2 — Download the Sysmon Configuration
Used the community-trusted configuration maintained by SwiftOnSecurity rather than writing one from scratch:

- Reduces noise compared to Sysmon's very verbose defaults
- Focuses on high-value telemetry rather than logging everything indiscriminately
- Matches the kind of baseline many real SOCs actually use in production
- Produces clean, structured event output that's easier to build KQL queries against later

Saved as `sysmonconfig.xml` in the same tools directory.

## Step 3 — Install Sysmon#

The `-i` flag installs Sysmon as a Windows service (not just running it once) and applies the config at install time. Running as a service matters because it means monitoring starts automatically on every boot without needing to be manually relaunched — a manually-run instance would stop covering the system the moment it's closed or the machine restarts. You should see confirmation that Sysmon is installed and running as a service.

## Step 4 — Verify Sysmon Events
Event Viewer → Applications and Services Logs → Microsoft → Windows → Sysmon → Operational

Confirmed events appearing, including:

- Event ID 1 — Process creation
- Event ID 3 — Network connection
- Event ID 11 — File creation
- Event ID 13 — Registry modification

This confirms Sysmon is functioning correctly and generating the telemetry the rest of the lab depends on.

## Notes / Lessons Learned
- Sysmon's config isn't "set once" — SwiftOnSecurity's config gets updated periodically as new evasion techniques and noisy defaults get identified. Worth treating this as a file to revisit occasionally rather than a one-time download.
- Sysmon logs live in their own dedicated event channel (`Microsoft-Windows-Sysmon/Operational`), separate from the standard Windows Security log — this matters later, since anything forwarding logs (WEF, Azure Monitor Agent) needs to be told to include this channel explicitly, or Sysmon data won't ship anywhere even though it's being generated correctly.