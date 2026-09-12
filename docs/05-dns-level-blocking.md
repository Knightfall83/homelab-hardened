# DNS-Level Ad and Tracker Blocking

## Overview
Per-device ad blocking is only as strong as the weakest device on the network. Deployed a network-wide DNS resolver (Pi-hole) so every device gets the same filtering with nothing to install.

## The port-53 conflict (two problems, not one)
Standing up a service to bind the standard DNS port required freeing it first, and two independent things were holding it:
1. The OS's own local DNS stub listener — disabled its network-facing listener, repointed local resolution to its internal-only path.
2. **A completely unrelated virtualization feature** (leftover from earlier, unused experimentation, zero active VMs) was independently bound to the same port for its own private network. Stopped it rather than deleting it outright, in case a VM is ever actually needed later.

Neither of these was expected going in — both had to be found by directly checking what was listening on the port, not assumed from documentation.

## Rollout
- New resolver deployed as a container, upstreams set to public resolvers, reachable network-wide for both DNS and its own management UI.
- **DHCP-handed DNS:** primary pointed at the new resolver, secondary kept as a public resolver for failover — a deliberate tradeoff (a client can occasionally bypass local filtering during failover) versus running a second redundant instance.
- **Router's own upstream DNS** repointed separately to a fast, privacy-respecting public resolver.

## A configuration gotcha
The command that applies DNS changes on the router didn't actually rewrite one specific internal file used for the router's own outbound lookups. Rather than trigger a full disruptive restart just to fix one file, sent the running service a signal to reread that file directly — same result, zero downtime.

## Verification
- From an actual client device (not the server itself): a normal domain resolves; a known ad-tracking domain returns null; the query shows up in the resolver's live log under the client's real network address, confirming per-device traffic is actually passing through and being filtered, not just theoretically covered.
