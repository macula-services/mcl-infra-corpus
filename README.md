---
title: mcl-infra-corpus
layer: index
audience: [agent, human]
stage: stable
---

# mcl-infra-corpus

*Knowledge corpus for infrastructure: containers, Linux, IaC, observability, cloud, and infra tooling. Markdown only — this repo is ingested by [mcl-rag](https://github.com/macula-services/mcl-rag), the mesh's shared memory.*

This repository holds reference knowledge an agent on the Macula mesh can
recall: how to run things in containers, how to stand them up with IaC,
how to watch them once they run, and what they cost. It contains no
runtime code. It is the infrastructure domain corpus of the mesh's
federated retrieval.

> **mcl-rag ingestion notes.** The sync loop fast-forwards this repo and
> re-embeds changed `**/*.md` files every 120 s. A commit is a deploy.
> Every note is chunked as written, so keep headings tight and
> paragraphs short.

---

## Start here

| You are | Read |
|---------|------|
| Agent, first recall | [`INDEX.md`](INDEX.md) → domain map |
| Human, first contact | [`INDEX.md`](INDEX.md) → read top-down |
| Looking for a term | [`GLOSSARY.md`](GLOSSARY.md) |
| Containers questions | [`containers/`](containers/README.md) |
| Linux questions | [`linux/`](linux/README.md) |
| IaC questions | [`iac/`](iac/README.md) |
| Observability questions | [`observability/`](observability/README.md) |
| Cloud questions | [`cloud/`](cloud/README.md) |
| Infra tooling questions | [`tooling/`](tooling/README.md) |

---

## Layout

| Domain | Where | Purpose |
|--------|-------|---------|
| **Containers** | `containers/` | Docker, Kubernetes, the runtime layer services ship as |
| **Linux** | `linux/` | The host: systemd, networking, disk, packages |
| **IaC** | `iac/` | Terraform, Ansible: infrastructure as declarative text |
| **Observability** | `observability/` | Metrics, logs, dashboards, alerting |
| **Cloud** | `cloud/` | Multi-cloud patterns, FinOps |
| **Tooling** | `tooling/` | Go for infra: CLIs, agents, small daemons |

---

## Conventions

- **Front-matter** on every note: `title`, `layer`, `audience`, `stage`.
- **One pattern per note.** Notes are lookup targets for RAG, not books.
- **Version-neutral where possible.** This domain ages; write the
  durable concept, pin versions only where the difference matters.
- **`stage`** is `draft`, `stable`, or `superseded`. Superseded notes
  link to their replacement instead of being deleted.
- **License:** MIT. Deposit only knowledge you may license as MIT.
