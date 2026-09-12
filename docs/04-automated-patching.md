# Automated Patching Without Losing Control

## Overview
Three escalating stages of automatic updates on the same Linux server, each one a deliberate decision with a documented tradeoff, not a "set and forget" default.

## Stage 1: security-only (the conservative default)
Ubuntu's unattended-upgrades ships scoped to security patches only by default. A backlog of ordinary (non-security) updates sitting unapplied was the signal it was too conservative for this box.

## Stage 2: everything, with immediate auto-reboot
Widened to every configured origin (not just the OS's own repos), with auto-reboot firing **immediately** when a change needs one, rather than on a fixed nightly window.

**Explicit risk accepted:** no remote power-on capability on this hardware. An automatic reboot that fails to come back up leaves the box dark until someone physically presses the power button. Accepted specifically because this is a home-LAN box sitting a few feet away, not a remote or production system — this tradeoff does not generalize to internet-facing infrastructure.

**Health-check gap closed alongside this:** the OS's own "reboot required" flag doesn't always get set correctly. Patched the health check (doc 03) to directly compare the running kernel version against the newest one installed on disk, closing a blind spot the standard flag missed.

## Stage 3: making it actually invisible
A desktop notification layer kept nagging about updates even though the underlying automation was working correctly the whole time. **First instinct (wrong, corrected same session):** kill the notifier and block it from ever starting. That's suppressing a symptom, not fixing anything.

**Real fix:** restore the notifier's normal silent-fetch behavior (an unrelated earlier setting change had put it into a manual "click to download" mode) and tighten the actual update-check interval from roughly once a day to every three hours. With the real pipeline visibly keeping pace, the notifier has nothing left to complain about.

**Principle:** if the fix for a warning is making the warning stop appearing rather than making the underlying condition false, nothing has actually been fixed.

## Verification
- `unattended-upgrade --dry-run -d` confirmed exactly which packages and origins are in scope before any live run.
- Post-change: `apt list --upgradable` empty, health check clean, confirmed across a real reboot.
