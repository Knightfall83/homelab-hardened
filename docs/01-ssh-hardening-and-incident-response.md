# SSH Hardening and a Cryptominer Incident

## Overview
First real work done on the home server: getting an automated process onto it securely, then using that access to run a proper first-look security sweep.

## Access model
- **Key-only SSH auth**, no password auth. Passwords have to originate somewhere (typed by a human, or stored and injected by a script), and neither is acceptable for an unattended process. SSH also reads passwords directly from the TTY, not stdin, specifically to block script injection.
- Generated a dedicated ed25519 key pair with **no passphrase**, scoped to this one automation purpose, installed via a one-time interactive session (typed once, never stored).
- Gotcha: `ssh-keygen -N ""` in PowerShell sets a literal two-character passphrase, not an empty one. Regenerate through a POSIX shell (Git Bash) for a genuinely empty passphrase.

## The passwordless-root decision
- Automation needing `sudo` on every command defeats the point of being unattended. The fix: a dedicated file under `/etc/sudoers.d/` granting the automation account passwordless, full `sudo`, set up once interactively and never touching a stored password.
- **Explicit threat model accepted:** a passphraseless key + passwordless root means anyone who gets the key file owns the box outright. Judged acceptable for a home-LAN machine with no direct internet exposure. Would not make the same call for anything internet-facing.

## Incident: found a cryptominer on first recon
- First real look at the server turned up an `xmrig` (Monero miner) directory.
- **Investigated before reacting.** Shell history showed a `git clone`, a local build, and exactly one run against a public mining pool, months earlier. No running process, no persistence mechanism (no cron, no systemd unit, no autostart hook), no copies elsewhere.
- **Conclusion: a forgotten personal experiment, not a compromise indicator.** Removed anyway per owner's request; scrubbed the wallet address and pool details from shell history.
- **Takeaway:** finding something that looks bad is step one. Confirming what it actually is — persistence, running state, origin — before treating it as an incident is what separates real incident response from a knee-jerk wipe.

## Verification
- `ssh -o BatchMode=yes <host>` confirms key-only login with zero prompt.
- `sudo -n whoami` returns `root` with no password prompt, confirming the sudoers rule.
- `find /` swept for any other miner-related file, confirming the removal was complete.
