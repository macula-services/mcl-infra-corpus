---
title: "Tooling: Go for Infrastructure"
layer: guide
audience: [agent, human]
stage: stable
---

# Tooling: Go for Infrastructure

*A Go program compiles to one self-contained executable that you copy to a box and run. That, plus a standard library that already covers HTTP, TLS, JSON and process control, is why so much operations tooling is written in Go.*

---

## Why Go fits this niche

| Property | What you get |
|----------|--------------|
| Single binary | No interpreter or package install on the target; copy and run. Pure-Go builds are statically linked. |
| Easy cross-compilation | `GOOS=linux GOARCH=arm64 go build` on a laptop produces a binary for an ARM box. |
| Broad standard library | HTTP client and server, TLS, JSON, CSV, templates, process execution, structured logging (`log/slog`) |
| Fast start, modest memory | Fine for short-lived CLIs, cron-style jobs and small agents |
| Simple concurrency | Goroutines and channels for fanning out over many hosts |

Many of the tools in this space (Docker, Kubernetes, Terraform,
Prometheus) are written in Go, so their client libraries are Go-first
as well.

---

## The shape of a good infra tool

**Input and output contract**

```
stationctl status --host beam01 --format json
```

- Configuration from flags and environment variables; nothing
  interactive, so it runs the same from a terminal, CI or a timer.
- Results on stdout in a machine-readable format (JSON), diagnostics on
  stderr. Scripts can pipe the one and log the other.
- Exit code 0 for success, non-zero for failure, and distinct codes
  when callers need to tell failures apart.

**Deadlines everywhere**

Every network call, subprocess and remote session should carry a
`context.Context` with a timeout, and the tool should cancel cleanly on
Ctrl-C (`signal.NotifyContext`). Unattended tools meet unreachable
hosts; one that hangs forever blocks the job that called it.

```go
ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
defer cancel()
out, err := exec.CommandContext(ctx, "systemctl", "is-active", "pingd").Output()
```

**Where the work comes from**

| Need | Package |
|------|---------|
| Files and paths | `os`, `io/fs`, `path/filepath` (stdlib) |
| JSON, CSV, XML | `encoding/json`, `encoding/csv`, `encoding/xml` (stdlib) |
| HTTP APIs | `net/http` with a client timeout (stdlib) |
| Running commands | `os/exec` (stdlib) |
| Structured logs | `log/slog` (stdlib) |
| SSH to hosts | `golang.org/x/crypto/ssh` (Go project's extended library, not stdlib) |
| YAML | a third-party module; the stdlib has none |
| Traces and metrics | OpenTelemetry Go SDK (third-party) |

Prefer the standard library and the `golang.org/x` modules; every extra
dependency is something to audit and update across the fleet.

---

## Practices and pitfalls

- **One tool, one job.** Small tools compose in scripts; a tool that
  does everything is hard to test and gets rewritten.
- **Make reruns safe.** A tool may be retried after a timeout; the
  second run should find the work done and change nothing, the same
  contract as an [Ansible](../iac/ANSIBLE.md) task.
- **Watch cgo.** Some packages (`net`, `os/user`) can use the C library
  on Linux, which makes the binary dynamically linked. Build with
  `CGO_ENABLED=0` when you need a truly static binary; cgo is off by
  default when cross-compiling anyway.
- **Instrument the tool.** Emit its own logs, metrics or traces so a
  failed run can be diagnosed like any other service; see
  [Observability](../observability/OBSERVABILITY.md).
- **Test against a real target too.** Interfaces make faking easy, but
  the surprises live in real SSH servers, real APIs and real timeouts.

## How it fits the corpus

The `mcl-*` clients, agents and helpers follow this shape: flags in,
work over the network with deadlines, structured output out. Knowing
the pattern makes a new tool read like a familiar one.

## Sources

- *Go for DevOps*. John Doak, David Justice. Packt Publishing, 2022. https://www.packtpub.com/en-us/product/go-for-devops-9781801819343
- *Learning Go*, 2nd edition. Jon Bodner. O'Reilly Media, 2024. https://www.oreilly.com/library/view/learning-go-2nd/9781098139285/
- Go documentation. https://go.dev/doc/
- Go, "Installing Go from source" (GOOS and GOARCH values). https://go.dev/doc/install/source
- Go packages: `context` https://pkg.go.dev/context , `os/exec` https://pkg.go.dev/os/exec , `log/slog` https://pkg.go.dev/log/slog , `os/signal` https://pkg.go.dev/os/signal
- Go, `cmd/cgo` (cgo defaults and cross-compilation). https://pkg.go.dev/cmd/cgo
- `golang.org/x/crypto/ssh`. https://pkg.go.dev/golang.org/x/crypto/ssh
- OpenTelemetry, "Go". https://opentelemetry.io/docs/languages/go/
