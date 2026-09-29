---
title: "Linux: systemd"
layer: guide
audience: [agent, human]
stage: stable
---

# Linux: systemd

*PID 1 on most Linux distributions: it starts, supervises and orders everything else. You describe each thing in a small unit file; systemd handles dependencies, restarts, logging and schedules.*

---

## Units

Everything systemd manages is a **unit**, and the file suffix says what
kind:

| Suffix | Manages |
|--------|---------|
| `.service` | A process (daemon or one-shot job) |
| `.socket` | A listening socket that starts a service on first connection |
| `.timer` | A schedule that starts another unit |
| `.mount`, `.automount` | Filesystem mounts |
| `.path` | Watches a path and starts a unit when it changes |
| `.target` | A named group of units, used as a synchronisation point (e.g. `multi-user.target`) |

A minimal service for a small HTTP daemon:

```ini
# /etc/systemd/system/pingd.service
[Unit]
Description=Ping responder
Wants=network-online.target
After=network-online.target

[Service]
ExecStart=/opt/pingd/bin/pingd --port 8080
Restart=on-failure
User=pingd

[Install]
WantedBy=multi-user.target
```

`[Unit]` holds ordering and dependencies (`After=` orders, `Wants=` /
`Requires=` pull in). `[Service]` says how to run it. `[Install]` is only
read by `systemctl enable`, which links the unit into the named target
so it starts at boot. Units for a user session live in
`~/.config/systemd/user/` and are driven with `systemctl --user`.

---

## Day-to-day commands

```bash
systemctl status pingd          # state, main PID, recent log lines
systemctl restart pingd
systemctl enable --now pingd    # start at boot and start now
systemctl daemon-reload         # re-read unit files after editing them
systemctl list-units --failed   # what is broken right now
systemctl edit pingd            # drop-in override in pingd.service.d/
```

Edit, then `daemon-reload`, then restart. Forgetting the reload means
systemd keeps using the old definition. Prefer `systemctl edit` drop-ins
over editing a vendor unit in `/usr/lib/systemd/system/`, which a
package upgrade will overwrite.

---

## The journal

`systemd-journald` collects stdout/stderr of every unit plus kernel and
syslog messages, with metadata (unit, PID, boot id) attached, so you can
filter instead of grep:

```bash
journalctl -u pingd             # one unit
journalctl -u pingd -f          # follow
journalctl -u pingd --since "1 hour ago"
journalctl -b -1 -p err         # errors from the previous boot
```

Whether logs survive a reboot depends on `Storage=` in `journald.conf`.
With `auto`, the long-standing default, logs are kept on disk only if
`/var/log/journal` exists and otherwise live in `/run` and are lost on
reboot. Current upstream documents `persistent` as the default but
fixes it at compile time, so distributions differ: check the directory
and the config on a new box rather than assume. Old boots missing
from `journalctl --list-boots` is the symptom.

---

## Timers instead of cron

A timer is a unit that starts another unit, by default the service with
the same name (`backup.timer` starts `backup.service`).

```ini
# backup.timer
[Timer]
OnCalendar=*-*-* 03:00
Persistent=true

[Install]
WantedBy=timers.target
```

- `OnCalendar=` is wall-clock scheduling, like a cron line.
- `OnBootSec=` and `OnUnitActiveSec=` are monotonic: "10 minutes after
  boot", "every 15 minutes since the last run".
- `Persistent=true` (only meaningful with `OnCalendar=`) runs a missed
  job right away if the machine was off at the scheduled time.
- `systemctl list-timers` shows the next and last run of every timer.

Compared with cron you get the job's output in the journal under its own
unit name, a visible failed state, resource limits and dependencies. The
cost is two files instead of one line.

---

## Pitfalls

- `After=network.target` does not wait for a usable network; use
  `network-online.target` with `Wants=` when the service needs it.
- `Restart=always` on a crashing service will loop until the start rate
  limit (`StartLimitBurst=`) trips, then stay failed.
- User units stop when the user logs out unless lingering is enabled
  (`loginctl enable-linger`).

## How it fits the corpus

systemd sits directly under the containers on a fleet box. On the podman
host, Quadlet `.container` files are turned into ordinary service units,
so "is it up, what did it log, will it come back after a reboot" is
answered with the same three commands for everything. See
[Docker](../containers/DOCKER.md) for the container side.

## Sources

- *The Linux DevOps Handbook*. Damian Wojsław, Grzegorz Adamowicz. Packt Publishing, 2023. https://www.packtpub.com/en-us/product/the-linux-devops-handbook-9781803245669
- systemd project site. https://systemd.io/
- systemd.io, "Running Services After the Network Is Up". https://systemd.io/NETWORK_ONLINE/
- systemd.unit(5). https://man7.org/linux/man-pages/man5/systemd.unit.5.html
- systemd.service(5). https://man7.org/linux/man-pages/man5/systemd.service.5.html
- systemd.timer(5). https://man7.org/linux/man-pages/man5/systemd.timer.5.html
- journald.conf(5). https://man7.org/linux/man-pages/man5/journald.conf.5.html
- journalctl(1). https://man7.org/linux/man-pages/man1/journalctl.1.html
- systemctl(1). https://man7.org/linux/man-pages/man1/systemctl.1.html
- Upstream copies of the same man pages (may block automated clients): https://www.freedesktop.org/software/systemd/man/latest/
