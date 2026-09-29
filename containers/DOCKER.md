---
title: "Containers: Docker"
layer: guide
audience: [agent, human]
stage: stable
---

# Containers: Docker

*An image is a frozen filesystem plus run instructions; a container is an ordinary Linux process started from it inside its own namespaces and cgroup. Docker is the tooling that builds, ships and starts those processes.*

---

## What a container actually is

No virtual machine and no separate kernel are involved. A container is a
normal process tree on the host kernel, given a restricted view of the
system:

| Kernel feature | Role in a container |
|----------------|---------------------|
| **Namespaces** | Give the process its own view of one resource type: process ids (pid), network stack (net), mount table (mnt), users (user), hostname (uts), IPC. Inside, the process sees only its own world. |
| **cgroups** | Cap and meter CPU, memory and block I/O, so one workload cannot starve the host. `docker run --memory` and `--cpus` set these. |
| **Union / overlay filesystem** | Stacks read-only image layers with one thin writable layer on top, so many containers share the same image bytes on disk. |

`ps` on the host shows container processes like any other; that is a
useful sanity check when something looks wrong.

---

## Who does what

```
docker CLI ──REST──▶ dockerd          builds images, manages networks, volumes, the API
                        │
                    containerd        container lifecycle: pull, create, start, stop
                        │
                      runc            sets up namespaces + cgroups, execs the process
                        │
                   Linux kernel
```

The client can talk to a daemon on the same machine or a remote one.
containerd uses runc by default; other OCI runtimes can be swapped in.
Podman reaches the same result without a long-running daemon, which is
why the lab box in this fleet runs it under systemd instead.

---

## Images and the layer cache

An image is built from a `Dockerfile`. Instructions that change the
filesystem (`RUN`, `COPY`, `ADD`) each add a layer; the result is
addressed by a content digest.

Cache rule: when an instruction or its inputs change, that layer **and
every layer after it** is rebuilt, even if the later steps would produce
the same bytes. So put the slow, stable steps first and the frequently
edited ones last:

```dockerfile
FROM debian:12-slim@sha256:<digest>   # pinned base
COPY deps.lock /src/                  # changes rarely
RUN fetch-deps /src/deps.lock         # cached until the lock changes
COPY . /src                           # changes on every commit
RUN build /src
CMD ["/src/bin/app"]
```

---

## Practices that pay off

- **Pin the base by digest.** A tag like `latest` or `12` can be
  repointed by the publisher; a digest cannot.
- **One concern per container.** The container lives as long as its
  main process; bundling several services forces you to ship your own
  supervisor. Scale and restart pieces independently instead.
- **Multi-stage builds.** Compile in a fat build stage, copy only the
  artifact into a small runtime stage. Smaller images, less to patch.
- **Treat containers as disposable.** The writable layer disappears
  with the container. Anything worth keeping goes in a volume or an
  external store.

---

## Pitfalls

- Secrets passed with `ENV` or copied in a `RUN` step stay in the image
  history even if a later step deletes them; use build secrets.
- A cache hit on `RUN apt-get update` can hide stale package lists; keep
  update and install in the same `RUN`.
- Running as root inside the container is still root on the kernel's
  terms unless user namespaces are configured; set `USER`.

## How it fits the corpus

Images are the deploy unit for every service in this fleet: pinned by
digest, the digest is both what runs and the rollback target. The
[Kubernetes](KUBERNETES.md) note covers scheduling many containers;
[systemd](../linux/SYSTEMD.md) covers supervising them on a single box.

## Sources

- *The Ultimate Docker Container Book*, 3rd edition. Dr. Gabriel N. Schenker. Packt Publishing, 2023. https://www.packtpub.com/en-us/product/the-ultimate-docker-container-book-9781804613986
- Docker Docs, "What is Docker?" (architecture, namespaces). https://docs.docker.com/get-started/docker-overview/
- Docker Docs, "Alternative container runtimes" (containerd and runc). https://docs.docker.com/engine/daemon/alternative-runtimes/
- Docker Docs, "Docker build cache". https://docs.docker.com/build/cache/
- Docker Docs, "Building best practices". https://docs.docker.com/build/building/best-practices/
- Docker Docs, "Resource constraints". https://docs.docker.com/engine/containers/resource_constraints/
- Docker Docs, "Volumes". https://docs.docker.com/engine/storage/volumes/
