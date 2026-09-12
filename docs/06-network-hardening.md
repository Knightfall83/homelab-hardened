# Network Hardening: WiFi Security and a Mesh Lesson

## Overview
A full router tuning pass covering real throughput, WiFi security posture, and DHCP hygiene, plus one change that took the whole house's WiFi offline and the reason why.

## Measured, not assumed, throughput problem
A bandwidth-shaping feature (QoS) was enabled with its bandwidth limits both set to zero. Rather than guess at impact, measured before and after with real speed tests: throughput **more than doubled** with it disabled, and loaded latency improved rather than got worse. A misconfigured shaping feature was quietly halving the connection. Decision to disable it permanently was backed by the actual numbers, not a hunch.

## A wrong assumption caught mid-project
Assumed a specific device had fast networking hardware that would benefit from a faster switch port. Checked directly instead of continuing to plan around the assumption — it only had older, slower hardware, meaning the port change could never have helped it regardless. Correcting this on the spot avoided wasting the rest of the plan chasing a change with zero possible benefit.

## The mesh-wide failure from a WiFi security upgrade
Attempting to move two bands from WPA2 to a stronger WPA3-transitional mode caused wireless client count to collapse network-wide almost immediately.

**Root cause:** the network's secondary access point operates as part of a unified mesh, not as an independent router. A security change intended for the main router automatically propagated to the secondary hardware, which handled the newer standard badly, and knocked out WiFi for every device on the mesh, not just clients near the main unit.

**Standing lesson:** any future wireless security change on a mesh network has to be treated as affecting every device on the mesh network-wide, not just the router it was applied to directly. That's not visible from the settings screen — it only becomes obvious by watching it happen.

## DHCP hygiene
Fixed-address reservations set for every device that other services (firewall rules, reverse proxy configs, monitoring) depend on having a stable IP, so a lease renewal can never silently break something else that assumed the old address.

## Verification
- Multiple speed-test runs, before and after, for the QoS change.
- Physical device connectivity checks across the whole mesh after both the security-change failure and its rollback.
