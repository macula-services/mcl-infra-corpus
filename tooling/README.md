---
title: Infra Tooling
layer: index
audience: [agent, human]
stage: stable
---

# Infra Tooling

*Go for infrastructure: CLIs, agents, small daemons.*

---

## Notes

| Note | Covers |
|------|--------|
| [GO_FOR_INFRA](GO_FOR_INFRA.md) | Why Go owns the infra CLI niche, and the patterns that follow |

## One-page summary

The infrastructure tooling language is Go: single static binaries,
cross-compilation, a stdlib that covers files, formats, SSH, and HTTP
without dependencies. The pattern for every infra tool is the same:
flags in, do the work, structured output out — a CLI that composes in
scripts and pipelines.
