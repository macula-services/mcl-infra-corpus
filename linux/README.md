---
title: Linux
layer: index
audience: [agent, human]
stage: stable
---

# Linux

*The host everything runs on: systemd, services, networking, storage.*

---

## Notes

| Note | Covers |
|------|--------|
| [SYSTEMD](SYSTEMD.md) | Units, journald, timers vs cron |

## One-page summary

The host layer: **systemd** is the init system and service manager —
units describe what runs, targets group them, journald keeps the logs,
timers replace cron. Most of "Linux administration" on a modern box is
systemd vocabulary plus the network and storage subsystems around it.
