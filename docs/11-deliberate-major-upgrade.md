# Change Management: A Deliberate Major Upgrade

## Overview
Automated patching (doc 04) is deliberately scoped to exclude major-version jumps of specific critical software. This doc covers why, and the controlled procedure used when a major upgrade actually happened.

## Reading a risk before it applies to you
Came across a report that an upcoming major version of a key application had a history of breaking installs during its one-time database migration on servers with older starting versions and years of accumulated data. Rather than reacting to the headline, read the actual technical detail: checked the exact starting version and database age on this specific server against the report's stated risk factors, and confirmed this instance was already sitting on the recommended safe starting point.

**Action taken anyway:** placed an explicit version hold on that application's core packages. Routine security patches still apply automatically; a major-version jump specifically requires a deliberate, manual decision. Low measured risk isn't the same as zero risk, and a hold costs nothing.

## Executing the upgrade, weeks later, on purpose
When the upgrade was actually run:
1. **Full backup first**, verified by actually testing the archive's integrity and confirming it contains the critical data files — not just trusting a zero exit code.
2. **Third-party plugins removed ahead of time**, since the original risk report specifically flagged them as a common migration failure point.
3. Migration executed and monitored directly rather than assumed successful.
4. **Full reboot verification** — confirmed the service comes back clean on a cold boot, not just mid-session, before considering the upgrade done.
5. Backup deleted only after **real functional verification** (not just a green status check), then the version hold reapplied at the new version number so the same controlled process happens again next time, rather than a future routine update pulling in the next major version by accident.

## Principle
A major software upgrade on infrastructure that matters is a decision, not something that should ever happen as a side effect of routine automated maintenance.

## Verification
- Backup archive integrity tested before proceeding.
- Post-migration data integrity confirmed (record counts, no orphaned data).
- Full cold-boot verification, not just a live-session check.
