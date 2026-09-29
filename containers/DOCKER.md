---
title: "Containers: Docker"
layer: guide
audience: [agent, human]
stage: stable
---

# Containers: Docker

*Images are the immutable template; containers are running processes sandboxed by namespaces and limited by cgroups. The rest is plumbing.*

---

## The kernel primitives

Containers are only possible because Linux supplies the primitives:

| Primitive | What it does |
|-----------|--------------|
| **Namespaces** | Sandbox a resource class: pid (process ids), net (network), mnt (mounts), user, uts (hostname), ipc. A container sees its own world. |
| **cgroups** | Limit and account resources: CPU time, memory, io. Prevents the noisy-neighbor problem — one container consuming the whole host. |
| **Layers** | Filesystem deltas stacked into a single view; the unit of image caching. |

A container is *one or more processes encapsulated by Linux namespaces
and restricted by cgroups* — nothing more. There is no container
process; there is no virtual machine.

---

## The runtime stack

```
Docker Engine        ← API, networking, plugins, the CLI talks to this
    │
Container runtime
    ├── containerd   ← image management, networking, extensibility
    └── runc         ← low-level container creation and management
    │
Linux OS             ← namespaces, cgroups
```

The runtime owns the container lifecycle: pull the image from a
registry, create the container from it, start, stop, remove. The engine
adds the API layer and the developer experience.

---

## Images vs containers

| | Image | Container |
|---|---|---|
| Nature | Immutable template | Running instance |
| Built from | A Dockerfile | An image |
| Has | Filesystem layers, metadata (entrypoint, env, user) | An isolated process tree + writable layer |
| Analogy | A class | An object |

An image's layers cache independently: change the last line of a
Dockerfile and only the layers above the change rebuild. Order the
Dockerfile so that what changes often comes last.

---

## The build contract

```dockerfile
FROM base:latest          # the base image
COPY app /app             # layer: the app
RUN build                  # layer: build artifacts
ENTRYPOINT ["/app/run"]    # what executes
```

Rules of thumb:

- **Pin the base** — by digest in production; `latest` moves.
- **One process per container.** The container dies with its main
  process; multi-process images reinvent the init system poorly.
- **Minimize layers.** Each `RUN` is a layer; chained commands and
  multi-stage builds keep images small and attack surface down.
- **The writable layer is scratch.** State belongs in volumes; a
  container that stores data in its layer loses it on removal.

## Why it matters

Docker's contribution is not the kernel primitives — they predate it —
it is the **supply-chain contract**: one artifact, built once, run
anywhere, described by a Dockerfile. Every service the mesh runs ships
as an image pinned by digest; the digest is the deploy unit and the
rollback target.
