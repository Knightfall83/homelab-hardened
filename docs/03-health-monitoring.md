# Health Monitoring and a Heartbeat Script

## Overview
A server with no way to report its own health is a server you find out about when something's already broken. This doc covers building a cron-driven check that gives the box a heartbeat.

## What it checks
- Root filesystem usage, memory/swap, and per-disk SMART health.
- Failed systemd units.
- The full baseline set of expected services, checked individually.
- HTTP health endpoints for the actual applications, not just "is the process running."
- Firewall status, pending security updates, and whether the running kernel matches the newest one installed on disk (a reboot-required check that's more reliable than the OS's own flag — see [doc 04](04-automated-patching.md)).

## Output
- A single status line, `OK` / `WARN` / `CRIT`, plus detail, written to a flat file.
- Runs on a schedule via cron, appends to a history log capped at a fixed line count.

## A silent-failure bug in v1
The first version checked for a specific named web-server process to confirm one application was healthy, and silently reported nothing when that process name wasn't found, rather than flagging a mismatch. The actual web server on that port turned out to be a different application's bundled instance, running under a different process name entirely.

**Why this matters:** a check that silently no-ops when its assumption is wrong is worse than no check at all — it looks like coverage on a dashboard while actually testing nothing. Fixed by checking for the real process name instead of an assumed one, and treating "expected service not found under any known name" as a reportable condition rather than a silent skip.

## Verification
- Ran the corrected check against every known-healthy state and one deliberately broken state (service stopped) to confirm both paths report correctly, not just the happy path.
