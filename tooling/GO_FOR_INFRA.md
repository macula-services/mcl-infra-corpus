---
title: "Tooling: Go for Infrastructure"
layer: guide
audience: [agent, human]
stage: stable
---

# Tooling: Go for Infrastructure

*One static binary, no runtime, a stdlib that already speaks the protocols infrastructure uses. That is why the infra CLI niche belongs to Go.*

---

## Why Go owns this niche

| Property | What it buys |
|----------|--------------|
| Static single binary | Ship one file to a box; no runtime, no dependency resolution |
| Cross-compilation | Build for any OS/arch from one machine — `GOOS=linux GOARCH=arm64 go build` |
| Stdlib coverage | Files, HTTP, JSON/YAML-ish, SSH, exec, TLS — the whole infra surface without a dependency tree |
| Fast startup | CLIs feel instant; agents and short-lived tools are cheap |

The consequence: the operational tooling around a fleet — CLI
clients, small daemons, sync agents — is Go-shaped, whatever language
the services themselves are written in.

---

## The patterns every infra tool repeats

### 1. Flags in, structured output out

```
tool --target host --format json
```

- Input: flags or environment, nothing interactive.
- Output: JSON (or YAML) on stdout, logs on stderr. The contract that
  makes tools composable in scripts and pipelines.
- Exit codes mean something: 0 = done, non-zero = failed, distinct
  codes for distinct failures where callers branch on them.

### 2. The work comes from the stdlib

| Task | What does it |
|-------|--------------|
| Filesystem operations | `os`, `path/filepath` — walk, copy, watch |
| Data formats | `encoding/json`, `encoding/csv`, `encoding/xml` |
| Remote data | `net/http` with contexts and timeouts |
| Command execution | `os/exec` — run things, capture output |
| Remote execution | `golang.org/x/crypto/ssh` — the SSH library every fleet tool uses |
| Observability | OpenTelemetry SDK — traces and metrics from day one |

The rule: reach for the stdlib first. An infra tool with fifty
dependencies has lost the property that justified Go in the first
place.

### 3. Contexts and timeouts are not optional

Every network call, every `exec`, every SSH session takes a
`context.Context` with a deadline. Infra tools run unattended against
unreliable targets; the tool that hangs is the tool nobody trusts in
a cron job.

---

## Rules of thumb

- **One tool, one job.** A CLI that does one thing well composes; a
  Swiss-army tool is rewritten by the next team anyway.
- **Idempotent by default.** An infra tool may be run twice; make the
  second run a no-op (the same contract as Ansible tasks).
- **Observability in the tool.** Emit the trace and the metrics from
  day one — the tool's own runtime becomes diagnosable like everything
  else it manages.
- **Test the happy path against a real target.** The stdlib makes it
  easy to fake; the fleet is where the surprises live.

## Why it matters

Every `mcl-*` client, agent, and helper on the mesh is this shape:
flags in, work over the network, structured output out. The Go
patterns here are the house style for the tooling layer — know them
and a new tool reads like a familiar one.
