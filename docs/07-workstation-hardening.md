# Windows Workstation Hardening

## Overview
A hardened home lab isn't just the server. This doc covers auditing and hardening the actual daily-driver Windows machine everything else runs from.

## Findings from a full audit
- Firmware over a year old with security fixes shipped since.
- Multiple background processes with no real function: a CPU-vendor telemetry uploader, a motherboard-vendor "live service" doing usage reporting, two game-launcher autostart entries, unrequested cloud-storage software retrying its own setup at every login.
- A hard-disabled built-in antivirus, disabled at a level below normal user settings.

## Judgment calls: not everything that looks like bloat is bloat
- A large first-party hardware-control suite (fan curves, liquid-cooler pump behavior, RGB) looked like a candidate for a full removal. Investigated component by component instead of removing the whole stack: the telemetry and promotional pieces came out, the actual hardware-control components stayed, since removing those would have cost real functionality (fan/pump control) for no security benefit.
- A process that looked like a memory leak (based on RSS alone) turned out to be two ML models loaded exactly as intended. Verified via process tree inspection before concluding anything was wrong — killing what looked like a duplicate process would have taken down a legitimate service for nothing.

## The antivirus tradeoff, investigated rather than forced
Attempted to re-enable the OS's built-in antivirus for defense-in-depth. It actively resisted: disable flags reverted themselves, and a protected system service refused to start even with elevated rights. **Root cause:** the OS deliberately stands its own built-in AV down when a third-party AV is registered as the active provider — that's by design, not a misconfiguration. The correct supported path is a specific "periodic scanning" toggle meant for exactly this "keep third-party real-time, add a second-opinion scan" scenario, not a registry fight against the OS's own security design.

## Verification
- Confirmed each removed autostart entry stayed gone across a reboot.
- Confirmed the primary antivirus's real-time protection was active and unaffected throughout.
