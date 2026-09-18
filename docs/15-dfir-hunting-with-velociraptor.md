# DFIR and Threat Hunting with Velociraptor

## Overview
A SIEM alone tells you something looked wrong. It doesn't let you reach out and ask a specific box a specific question on demand. Added Velociraptor, a digital forensics and incident response (DFIR) platform, as the hunting layer on top of the existing SIEM — deployed on the same low-power ARM board already running the network's centralized syslog collector, with the primary Linux server enrolled as a monitored client.

## Placement, not a new box
The server component is genuinely lightweight (a single Go binary), so rather than stand up new hardware, it went on the same Raspberry Pi 4 already doing log collection duty — one more service justifying that board's existence instead of another idle machine on the network. Firewall on that board went from nonexistent to a real default-deny posture as part of this build: explicit LAN-scoped rules for SSH, the DFIR web console, and the client-enrollment port, plus the pre-existing syslog rule, nothing else open.

## The client is the point
The server's own host wasn't the interesting target — the primary Linux server (the one running every other containerized service on the network) was enrolled as the actual monitored endpoint, using a version-matched agent so client and server never drift apart on protocol compatibility. Endpoint agents were deliberately run as native host installs rather than containerized, unlike almost everything else on this network: a monitoring agent that only sees its own container's filesystem and process list isn't monitoring the host at all, it's monitoring a box nobody attacks.

## A real GUI bug, and the pivot around it
The platform's hunt-creation wizard has a genuine bug: artifact selection visibly succeeds in the UI (the item shows selected, every wizard step unlocks), but the final "launch" action silently drops it, failing server-side with "No artifacts to collect" no matter how the wizard steps are navigated. Confirmed this wasn't a one-off by reproducing it multiple times through different navigation paths before concluding it was a real defect rather than operator error.

Rather than keep fighting a broken wizard, pivoted to the platform's own CLI, which exposes a direct single-client collection command distinct from the multi-client "hunt" object the wizard builds. Functionally identical for a single target: a named detection artifact runs against a specific enrolled client and returns real results over the same secure channel — the wizard bug cost time, not capability.

## Two real collections run
- **Cron persistence sweep** — every system and user crontab entry, plus every script under the standard cron directories, pulled and hashed. Surfaced the server's actual scheduled jobs end to end: routine OS housekeeping, a RAID-parity maintenance job, a dynamic-DNS updater, and a custom health-check script — a genuine full accounting of "what's allowed to run unattended on this box," which is exactly the question a persistence hunt is supposed to answer.
- **SUID/SGID binary sweep** — every setuid/setgid binary on the filesystem, the kind of privilege-escalation surface real intrusion investigations check first. Returned the expected baseline (the usual suspects: `sudo`, `su`, `mount`, `passwd`, a handful of desktop-integration helpers) with nothing unexpected present — a real negative result, not a skipped check.

## Verification
- Confirmed enrollment wasn't just a client-side "connected" log line: queried the server's own client index directly and got the real hostname and OS back for the enrolled machine.
- Both collections traced end to end — dispatched from the server, executed on the live client, real files hashed and returned — not simulated or run locally against the DFIR box's own filesystem.
- Uptime monitor added for the new service alongside every other containerized service on the network, so a future outage gets caught the same way everything else is.
