---
title: "Linux: systemd"
layer: guide
audience: [agent, human]
stage: stable
---

# Linux: systemd

*The init system and service manager: units describe what runs, targets group them, journald keeps the logs, timers schedule.*

---

## The unit

A **unit** is the thing systemd manages. The kind is the file suffix:

| Suffix | Unit kind |
|--------|-----------|
| `.service` | A long-running process |
| `.timer` | A schedule, activating another unit |
| `.mount` / `.automount` | Filesystems |
| `.socket` | Socket activation |
| `.path` | React to filesystem changes |
| `.target` | A group of units (a boot level, a state) |

```ini
[Unit]
Description=My service
After=network.target

[Service]
ExecStart=/usr/local/bin/myapp
Restart=always

[Install]
WantedBy=multi-user.target
```

`[Install]` says *when*: `WantedBy=multi-user.target` wires the unit
into the normal boot target.

---

## Managing units

```bash
systemctl status myapp      # state + last log lines
systemctl start|stop|restart myapp
systemctl enable|disable myapp     # start at boot, or not
systemctl daemon-reload            # after editing unit files
```

The workflow that matters: edit the unit → `daemon-reload` →
`restart`. Skipping the reload is the classic "why didn't my change
take?" mistake.

---

## journald

systemd's structured logger: every service's stdout/stderr lands in the
journal.

```bash
journalctl -u myapp          # one unit's logs
journalctl -u myapp -f       # follow
journalctl --since today     # time window
journalctl -b -1             # previous boot
```

The journal is indexed and queryable — richer than flat log files, at
the cost of a binary format and (by default) an in-memory journal that
vanishes on boot unless configured persistent.

---

## Timers vs cron

Cron still works; **systemd timers** are the modern answer:

- A timer is a unit, so it is managed, logged, and supervised like
  everything else.
- `OnCalendar=` schedules (cron-like); `OnUnitActiveSec=` and
  monotonic timers survive missed runs and reboots predictably.
- Timer + service = two units: the timer triggers the service; the
  service's logs go to the journal with its own name.

Rule of thumb: for anything a cron line does, a timer unit does with
better failure visibility. Cron remains for the one-off quick job.

## Why it matters

On a fleet box, systemd is the layer below the containers: every
service the mesh runs is a unit (or a Quadlet container unit), and
"is it running, what did it log, will it come back after reboot" is
three `systemctl`/`journalctl` commands answered the same way for
everything.
