---
title: "IaC: Principles"
layer: guide
audience: [agent, human]
stage: stable
---

# IaC: Principles

*Declare the desired state. Apply it any number of times, get the same result. Replace instead of mutate. These three sentences are the whole discipline.*

---

## Declarative desired state

In IaC, a declarative language describes the **desired state** of a
system; the tool derives the steps to bring reality into compliance
with it. You do not write "create the VM, then install the package" —
you write "a VM with this package exists", and the tool computes the
transition.

This is the same shape as Kubernetes' reconciliation loop and the
event-sourcing read-model rebuild: state is declared, the engine
converges.

---

## Idempotency

Applying an operation multiple times produces the same result as
applying it once. In IaC terms: *regardless of the starting state and
the number of times it is executed, the end state remains the same.*

What it buys:

- **Retries are free.** A failed apply is re-run, not repaired.
- **Rollback is thinkable.** The previous state is just another
  declared state.
- **No inconsistent outcomes** from half-applied changes.

Stateful tools (Terraform) reach idempotency by recording what they
manage and diffing against it; Ansible reaches it task by task, where
each task checks before it changes.

---

## Immutability — the cure for drift

**Configuration drift** is what happens when changes are made outside
the code: environments diverge in ways that are hard to reproduce and
harder to debug. Mutable infrastructure invites drift; long-lived
servers accumulate it.

**Immutable infrastructure** replaces rather than alters: a change
produces a new artifact (image, VM), and the old one is discarded.
Nothing is ever patched in place, so every environment is reproducible
by construction — what runs is what the code built.

---

## The practices that hold it together

| Practice | Why |
|----------|-----|
| Everything in source control | Scripts, pipelines, configs — the VCS is the change log |
| One tool owns one resource class | Terraform owns provisioning; Ansible owns configuration; overlap = two sources of truth |
| Secrets never in the repo | State and config must not carry credentials |
| Plan before apply | Read the diff of what *will* change; unexpected lines in the plan are drift caught early |

## Why it matters

IaC is the bridge that lets infrastructure ride the software
toolchain: review, versioning, CI, rollback. The two principles —
idempotency and immutability — are what make that bridge hold:
without them the repo describes an intention, not a state.
