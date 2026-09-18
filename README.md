# homelab-hardened

A real home lab, hardened one project at a time. This repo documents the actual security and infrastructure work done on my home network and Linux server: what was broken, what I found, how I fixed it, and how I verified the fix actually held.

This isn't a theoretical write-up. Every doc here comes from a project that actually shipped on a real server sitting in my house, with real before/after evidence, not a tutorial copied from somewhere else.

## Why this exists

I'm working toward a cybersecurity degree, and I wanted a public record of hands-on work that backs it up: incident response on my own box, firewall design, DNS-level filtering, patch management, container isolation, and disaster recovery, all done for real, not as a classroom exercise.

## The stack

- **Server:** Ubuntu 24.04, running everything below in Docker containers, with the container engine itself the only major thing still on bare metal.
- **Network:** A consumer router pushed to its actual limits, with WiFi security, DNS, and DHCP all tuned deliberately rather than left on defaults.
- **Monitoring:** A cron-driven health check plus a status dashboard, so problems get caught by a script before they get caught by me noticing something's down.

## Docs

| # | Doc | What it covers |
|---|-----|-----------------|
| 01 | [SSH Hardening and a Cryptominer Incident](docs/01-ssh-hardening-and-incident-response.md) | Key-only auth, the passwordless-root threat model, and finding + investigating an old cryptominer on first recon |
| 02 | [Host Firewall Lockdown with ufw](docs/02-firewall-lockdown.md) | Default-deny inbound, LAN-scoped rules, and a real bug where the firewall silently broke legitimate traffic |
| 03 | [Health Monitoring and a Heartbeat Script](docs/03-health-monitoring.md) | A cron-driven check covering services, disks, updates, and firewall state, and a silent-failure bug in the first version |
| 04 | [Automated Patching Without Losing Control](docs/04-automated-patching.md) | Escalating a Linux server from security-only patches to full unattended updates with auto-reboot, deliberately |
| 05 | [DNS-Level Ad and Tracker Blocking](docs/05-dns-level-blocking.md) | Deploying Pi-hole for the whole network and the two separate services actually squatting on port 53 |
| 06 | [Network Hardening: WiFi Security and a Mesh Lesson](docs/06-network-hardening.md) | Real 10-gigabit tuning, killing a broken QoS shaper, and a WPA3 rollout that took a whole mesh network offline |
| 07 | [Windows Workstation Hardening](docs/07-workstation-hardening.md) | Auditing and hardening the actual daily-driver machine, not just the server |
| 08 | [Secure Exposure: Reverse Proxy and Real TLS](docs/08-secure-exposure.md) | Exposing one internal service to the internet safely, with a real certificate and a scoped firewall exception |
| 09 | [Decommissioning a Service Without Leaving Debris](docs/09-decommissioning-safely.md) | Fully retiring an application, and cleaning up every firewall rule and health check left pointing at it |
| 10 | [Incident Response: A Zombie Auto-Updater](docs/10-zombie-autoupdater.md) | Tracing a broken package manager back to a piece of software that was quietly reinstalling something I'd already removed |
| 11 | [Change Management: A Deliberate Major Upgrade](docs/11-deliberate-major-upgrade.md) | Treating a major-version upgrade as a controlled change instead of letting automation do it by accident |
| 12 | [Container Isolation: Moving a Whole Server to Docker](docs/12-container-isolation.md) | Migrating every application-level service into isolated containers, one at a time, with zero data loss |
| 13 | [Disaster Recovery via Volume Shadow Copy](docs/13-disaster-recovery.md) | Recovering from an accidental mass deletion using a same-day Windows shadow copy |
| 14 | [Least-Privilege Remote Access](docs/14-least-privilege-remote-access.md) | Deliberately running a remote-access service with reduced privileges instead of full admin by default |
| 15 | [DFIR and Threat Hunting with Velociraptor](docs/15-dfir-hunting-with-velociraptor.md) | Adding a hunting layer on top of the existing SIEM, a real GUI bug found and worked around, and two real forensic collections run against the primary server |

## What's not here

A couple of personal-machine bug hunts (an audio driver issue, a GPU crash investigation) live outside this repo. This one is scoped to security and infrastructure work specifically.

## License

Documentation only, no proprietary configs or secrets included. Feel free to read and reference.
