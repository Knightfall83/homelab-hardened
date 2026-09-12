# Incident Response: A Zombie Auto-Updater

## Overview
A routine software update on the server failed with a generic package-manager error. Tracing it back revealed software resurrecting itself days after being deliberately removed.

## Why one broken package breaks everything
Linux package managers track exact installation state and refuse to proceed with **any** install or removal while one package is stuck in a broken, half-configured state. A single stuck package had been silently blocking every unrelated update on the system, including scheduled automated ones, for roughly a full day before it was even noticed.

## Root cause: found via investigation, not the first fix
The direct fix (force-remove the stuck package, reconcile the package database) cleared the immediate error, but didn't explain why an application removed days earlier had reappeared at all.

Digging further: a **separate, forgotten third-party auto-updater** for that exact application was still installed, with its own daily scheduled task, configured to auto-install updates with zero confirmation. Removing the application had removed the application — it hadn't removed the independent process whose entire job was making sure that application kept existing. The very next scheduled run detected the "missing" application, redownloaded it, and reinstalled it straight into a broken state.

## The generalizable rule
When removing software that ships or was paired with any kind of independent auto-update mechanism (a scheduled task, a background daemon, a cron job), the application and its auto-updater are **two separate removals**, not one. An updater with nothing left to update will often just silently reinstall what you removed, which from the outside looks exactly like the removal never worked.

## A secondary gotcha
A shared system group two services depended on for file permissions was accidentally deleted mid-cleanup and had to be recreated with its **exact original numeric ID** — Linux file ownership is tracked by ID number, not name, so a same-name-different-ID group would silently break permissions on every file it's supposed to own.

## Verification
- Full filesystem search confirmed zero remaining traces of both the original application and its auto-updater.
- `dpkg --audit` clean; subsequent scheduled updates ran without error.
