# Host Firewall Lockdown with ufw

## Overview
The server had no host firewall at all: every listening service (SSH, file sharing, two media servers) was reachable by anything on the network. This doc covers turning that into a default-deny posture, and a real bug the change caused.

## Design
- **Default policy:** deny incoming, allow outgoing, deny routed.
- **Every allow rule scoped to the LAN subnet**, nothing broader, except one later, deliberate exception for a single internet-facing reverse proxy port (see [doc 08](08-secure-exposure.md)).
- Each service gets its own named-port rule rather than a blanket allow.

## What turning it on actually found
- `ufw status` reported inactive, but a stale ruleset already existed underneath with several rules open to the entire internet (`Anywhere`), not just the LAN. Cleaned those out as part of the rollout so the live ruleset matched intent exactly.

## Verification before trusting it
- Opened a **brand new** SSH connection rather than trusting the one already open, in case the change caused a lockout.
- Tested every service from a real LAN client, not just loopback on the server itself.

## The bug it caused: silently dropped casting traffic
- A media-casting feature broke after the firewall went live. Root cause: the casting protocol binds a **random high UDP port** on every restart for one specific discovery step, in addition to its well-known fixed ports. A firewall rule naming a specific port can never cover a target that changes every restart.
- **Fix:** replaced the unpredictable-port traffic class with a single LAN-scoped "allow all inbound UDP from this subnet" rule, while every service that *can* be named precisely stayed locked to its own specific port.
- **Lesson:** a firewall doesn't fail loudly when it's wrong. The only way this surfaced was correlating a user-facing symptom (casting acting up) against firewall logs at the exact same timestamp — a service that's "just being flaky" and a firewall silently eating its packets look identical from the outside.

## Verification
- Confirmed zero UDP drops for the casting service in a clean test window after the fix.
- Live cast to a physical TV confirmed working end to end.
