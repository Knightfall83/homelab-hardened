# Secure Exposure: Reverse Proxy and Real TLS

## Overview
Exposing exactly one internal service to the public internet, deliberately and narrowly, rather than opening broad access or trusting a third-party tunnel with sensitive traffic.

## Choosing the approach for the actual scale
Evaluated three standard options against a real constraint (10-20 casual, non-technical viewers):
- **Mesh VPN:** rejected — every viewer needing individual client setup doesn't scale past a handful of technical users.
- **Third-party tunnel:** rejected — free-tier terms of service explicitly discourage the sustained bandwidth this use case needs.
- **Direct port-forward + reverse proxy with its own TLS termination:** chosen — scales to any number of casual viewers with just a link, at the cost of exposing one specific port directly.

## Getting a certificate without opening the usual ports
The standard HTTP-based certificate validation method wasn't available (the standard web ports were already committed to a different service). Used DNS-based validation through the dynamic-DNS provider's API instead, which requires no port at all for the validation step itself.

**Configuration gotcha:** the reverse proxy tries to claim the standard HTTP port by default for a redirect behavior, even when the actual service runs on a different port entirely. Fixed with one explicit flag disabling that default.

## Scoped firewall exception
This is the **one deliberate exception** to an otherwise fully LAN-scoped firewall (see [doc 02](02-firewall-lockdown.md)): a single port, opened specifically for this one reverse-proxy service, with nothing else in the ruleset opened wider than the LAN.

## Verification (the part that actually matters)
A test that only checks from inside your own network can pass even when it wouldn't work for a real external user, because routers don't always allow looping back to their own public address from inside. Verified genuine external reachability using a multi-location internet testing service hitting the public hostname from several actual external networks, confirming both TCP connectivity and a valid certificate from outside, not just a green check from the LAN.

The final, most important test wasn't technical at all: a real, non-technical user successfully connecting from a phone on cellular data, with no VPN, no special client, and a valid cert with no browser warning.
