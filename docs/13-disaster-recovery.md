# Disaster Recovery via Volume Shadow Copy

## Overview
A scripting bug during routine cleanup deleted most of the top level of a Windows user profile, including an entire local AI-assistant install and its memory store. This doc covers the root cause and the recovery.

## Root cause
A cleanup command's file path contained a single stray leading character. That malformed path caused path resolution to fail silently, which made the next step in the pipeline **fall back to listing a much broader, unintended location instead of erroring out**. That broader listing is what got piped into a recursive, forced delete.

**The generalizable lesson:** a script that fails silently and falls back to a wider scope, rather than erroring loudly, turns a small typo into a large-blast-radius incident. Validating that a resolved path actually matches what's expected, before feeding it into anything destructive, is the concrete mitigation.

## Recovery
Windows automatically maintains periodic Volume Shadow Copies in the background, independent of any deliberate backup strategy. A shadow copy from earlier the same day, taken before the incident, was still available.

1. Mounted the shadow copy as a normal accessible path.
2. Restored the deleted content via a straight file copy, verified by hand afterward that real file content had come back, not just empty directory structures.
3. Found and recovered a second, smaller gap where the same bad delete had reached partway into a system directory before locking up on an in-use file.
4. **Confirmed the actual gap the snapshot couldn't cover** — anything done between the snapshot's timestamp and the incident — was empty, by directly asking whether any work had happened in that specific window, rather than assuming the recovery was complete.

## Verification
- Manually spot-checked restored files for real content, not just presence.
- Confirmed with the affected party that nothing meaningful happened in the narrow window a shadow copy structurally cannot cover.

## Takeaway
A general-purpose OS feature nobody set up specifically for this purpose turned a potential multi-day data-loss incident into a same-evening recovery. Knowing a mechanism like this exists — and how to actually use it under pressure — is worth more than most dedicated backup tooling nobody actually tests until the day they need it.
