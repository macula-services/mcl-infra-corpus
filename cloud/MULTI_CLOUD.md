---
title: "Cloud: Multi-Cloud"
layer: guide
audience: [agent, human]
stage: stable
---

# Cloud: Multi-Cloud

*Multi-cloud is a deliberate design choice, not an accident of vendors. The principles that make it work are the same ones that make any distributed system work.*

---

## Multi-cloud vs hybrid cloud

| Term | Meaning |
|------|---------|
| **Hybrid cloud** | Private (on-prem) + public cloud, connected |
| **Multi-cloud** | Several public providers used deliberately: resilience, leverage, best-of-breed |

Multi-cloud is not "lift and shift to two providers" — that doubles the
operations cost and buys nothing. It is a design decision: which
workloads live where, and why.

---

## The principles of multi-cloud design

### 1. Domain-driven boundaries

Cut the system by **bounded context** (see the beam corpus's domain
modeling notes), not by provider. A context that owns its data and its
contracts can move between providers — or span them — without dragging
the rest along. The alternative, provider-shaped cuts, makes every
change a cross-provider migration.

### 2. API-first

Contracts over integrations: every context exposes an API, and the
network is the only thing shared. Provider-native services (queues,
object storage) hide behind your own interfaces, so swapping a
provider is a reimplementation of an adapter, not a rewrite.

### 3. Choose cloud-native foundations deliberately

Pick the *technologies* first (container runtime, orchestrator, data
stores), then map them onto providers — not the reverse. The
cloud-native layer is the portability layer.

### 4. Data placement is the hard part

Data gravity is real: egress costs and latency mean data must live
where its consumers live. Replication across providers is the
expensive case — reserve it for what genuinely needs it (DR,
compliance), and design eventual consistency into the rest.

---

## The costs to name

| Cost | Reality |
|------|---------|
| Abstraction overhead | The portability layer is code you write and maintain |
| Egress fees | Moving data between providers is billed — usually the surprise line |
| Skill surface | Two providers = two sets of operations knowledge |
| Weaker per-provider leverage | Spreading spend reduces your discount on any one |

## Rules of thumb

- **Default single-cloud.** Multi-cloud is the answer to a *named*
  problem (resilience, regulation, negotiation), not the default.
- **Abstract at the seams, not everywhere.** Wrap the provider
  services you depend on; do not build a cloud-neutral layer over
  everything — that is a second cloud to maintain.
- **Measure the egress before you promise the resilience.**
  Cross-provider DR that costs more than the outage it covers is a
  plan, not a practice.

## Why it matters

The mesh's fleet lives on hosted boxes and stations across providers
already — multi-cloud is the lens for the choices that are being made
anyway: which box on which provider, what moves, what stays, and what
it costs to have it elsewhere.
