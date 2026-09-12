# Least-Privilege Remote Access

## Overview
Setting up remote access to a local AI assistant from a phone, with a deliberate least-privilege decision baked into how it runs persistently.

## The setup
- Remote access runs as a background service, started automatically at login with zero visible window and zero manual step required.
- Confirmed working end to end: a real message sent from a phone, off the local network, received a real reply from the session running back on the desktop.

## The least-privilege decision
Unlike other persistent background services on the same machine, this one runs with **explicitly reduced, non-administrator privileges**, on purpose. A session reachable remotely, potentially from outside the physical premises entirely, should not automatically carry full administrative control over the machine it connects to by default. If elevated access is ever genuinely needed for something specific, that's a separate, explicit action — not a standing default permission granted to every remote session.

## Scope decision: text only, not voice
Explicitly declined extending the same assistant's voice interface to remote/mobile access, despite it being technically possible. Reasoning: the voice interface is tied to hardware physically connected to the desktop, running two live instances against the same shared state risks data conflicts, and the mobile OS is aggressive about killing background processes — none of which is a good foundation for a feature that needs to just work reliably. Keeping remote access scoped to text only was a deliberate scope-reduction, not a missing feature.

## Verification
- Confirmed the background service auto-starts at every login without manual intervention.
- Confirmed via direct privilege check that the running remote-access process does **not** hold administrator rights.
