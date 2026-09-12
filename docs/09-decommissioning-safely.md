# Decommissioning a Service Without Leaving Debris

## Overview
Removing an application cleanly is more than uninstalling the package. This doc covers fully retiring one of two redundant media servers, and tracing down every dependent config that had quietly been built around its existence.

## The core constraint
The actual data (a multi-terabyte media library) is not the application. Removal has to leave the library completely untouched while removing every trace of the software that served it: the package, its service, its repository and signing keys, and its configuration/metadata directory.

## Downstream dependencies found and closed
- **A firewall rule** naming the now-removed service's port, pointing at nothing. Deleted.
- **A broad UDP allow rule** that existed for one specific reason: the removed application's casting feature bound an unpredictable random port that no fixed rule could cover, so a wide allow existed to compensate. With that application gone, narrowed the rule back down to only the specific fixed ports the remaining services actually need — closing a gap that no longer had a justification.
- **A health-check entry** for the removed service. Left in place, it would report a false alarm on every single run, forever, for something intentionally gone. Removed and re-verified the check reported clean.

## A deliberate non-fix
Removing the application's dedicated user account also attempted to remove its associated system group — and failed, because another active account still depended on that group's permissions. **Correct outcome, not a bug**: recreating that group with a different internal ID than what other files' permissions expect would have silently broken file access for the *other* service still using it. Left it exactly as-is.

## Principle
A rushed decommission removes the application and calls it done. A real one traces every dependency, firewall rule, and monitoring check that quietly grew up around it and closes each one deliberately — otherwise you're left with stale rules and false alarms that mislead whoever looks at the system next.

## Verification
- Confirmed the media library's file count and total size unchanged before and after.
- Re-ran the health check and confirmed a clean report with the stale check removed.
