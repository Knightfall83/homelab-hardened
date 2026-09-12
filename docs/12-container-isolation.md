# Container Isolation: Moving a Whole Server to Docker

## Overview
Migrated every application-level service on the home server from native, host-installed applications into isolated containers, one real running service at a time, on a server that couldn't afford real downtime.

## Why isolation matters here
Natively installed applications share the same underlying system libraries and package manager. A conflict or update in one can, in principle, reach across and affect another with no relationship to it. Containers give each application its own self-contained environment with only the specific files and ports it actually needs exposed — a smaller, more explicit attack/failure surface per service.

## Sequencing by risk
- **First conversion deliberately chosen as low-risk**: an application with almost no data worth preserving, rebuilt fresh rather than migrated, to work out the process before touching anything that mattered.
- **Middle batch (four services in one sitting):** each with its own specific migration wrinkle — one needed a background-loop workaround for a discovery daemon that doesn't behave the same way inside a container as it does under native systemd; another's existing (and known-broken) configuration was carried over **exactly as-is** rather than silently fixed, since changing undocumented existing behavior during a migration is its own risk.
- **Last and most cautious: the primary media server.** Verified GPU passthrough into an isolated test container *before* touching the real service. Bind-mounted the existing data and config directly rather than copying it, so there was never a second, potentially-diverging copy of the data. Kept the original native installation disabled-but-present as a rollback path through a real multi-day soak period before final removal — a materially more conservative rollback posture than every other service in this batch got, specifically because this one had real, irreplaceable usage history on the line.

## A pre-existing gap the migration surfaced
A hands-on playback test after the final migration revealed that hardware-accelerated video transcoding had never actually been enabled in the application's own settings, containerized or not. Not a bug introduced by the migration — a genuine thorough test simply surfaced a gap that had existed the whole time and nobody had happened to trigger before.

## Verification
- Data integrity confirmed by direct comparison (record counts, file paths) between the pre-migration and post-migration states for every service.
- GPU passthrough verified independently in a disposable test container before the real workload touched it.
- Full reboot verification for every container: correct auto-start, correct health status, and — for the primary media server — a live confirmation that hardware transcoding was actually running on the GPU during playback.
