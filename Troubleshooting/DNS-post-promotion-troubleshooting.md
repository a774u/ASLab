## Overview
Right after switching DC01's DNS setting from `8.8.8.8` to self-referencing (`127.0.0.1`), Windows started reporting "No internet connection" in Settings. This documents the investigation into why, and what it actually turned out to be.

## The Symptom
- Windows Settings → Network & Internet → Status showed **"No internet connection"**
- This appeared immediately after the DNS switch, right when I expected everything to be working

## Investigating Step by Step
Rather than assuming DNS itself was broken, I isolated each layer individually:

1. **`nslookup google.com`** (querying DC01 itself, no server specified) — resolved successfully. This ruled out DNS resolution as the cause: DC01's DNS server was able to fully resolve an external name on its own, with **no forwarders configured at all**, because Windows DNS Server ships with **Root Hints** — the addresses of the internet's root DNS servers — enabled by default, letting it do full recursive resolution without needing forwarders as a hard requirement.
2. **`ping 8.8.8.8`** — 0% packet loss, ~17ms replies. Confirmed raw routing/connectivity to the internet was completely fine.
3. **`ping google.com`** — confirmed name-based ping also worked, consistent with the nslookup result.
4. **Edge → `http://google.com`** — loaded normally. Ruled out a browser-level block (I'd suspected IE Enhanced Security Configuration at this point, but it wasn't the cause).

## Root Cause
None of the above were actually broken. The "No internet connection" indicator in Windows Settings is driven by **NCSI (Network Connectivity Status Indicator)** — a separate background probe that checks a specific Microsoft test endpoint (`msftconnecttest.com` / `dns.msftncsi.com`), independent of general connectivity. NCSI is known to be unreliable, particularly on NAT'd VM setups like this one — it's possible for real connectivity and DNS to be entirely fine while NCSI's specific probe still fails or lags behind, showing a false "no internet" state.

Given every other test passed, this was NCSI reporting stale/incorrect status, not a genuine network problem. No fix was needed — the badge caught up to reality on its own shortly after.

## Notes / Lessons Learned
- **A status indicator is a claim, not a fact** — worth verifying independently rather than trusting the first "no internet" badge at face value. This is the same instinct incident investigation depends on generally: check the underlying signal, not just the summary label built on top of it.
- Also added `8.8.8.8` as an actual DNS **Forwarder** (DNS Manager → DC01 Properties → Forwarders) while investigating this — not because it was the fix, but because relying purely on root hints for every external query is less efficient than forwarding to a known-good resolver directly. Worth keeping regardless of this specific issue.