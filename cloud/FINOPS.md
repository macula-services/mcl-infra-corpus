---
title: "Cloud: FinOps"
layer: guide
audience: [agent, human]
stage: stable
---

# Cloud: FinOps

*Cloud spend as an engineering concern: make it visible (Inform), reduce it (Optimize), keep it that way (Operate). Three pillars, worked in parallel, not phases.*

---

## Why FinOps

Before the cloud, cost lived in procurement: a purchase, amortised,
forgotten. The cloud flipped the model — everything is metered, hourly,
per-resource — and nobody saw the bill until it arrived. FinOps exists
because in the cloud, **every technical decision is a spending
decision**, and the engineer who upscales a database is the one
spending the money.

The paradigm shift: cost moves from a finance problem to an
engineering property, owned jointly by technical, financial, and
project teams.

---

## The three pillars

Worked **in parallel**, not as sequential phases — an organisation
tackles its most pressing pillar first and runs them concurrently.

### 1. Inform — make cost visible

Cost allocation: who spends what, per business unit, project, region,
environment. Two exercises follow:

| Exercise | What it is |
|----------|------------|
| **Showback** | Divide costs across departments and *report* to management |
| **Chargeback** | *Bill* the departments for what they used |

Showback/chargeback sound simple and are not: they depend on naming
conventions and tagging — Operate-pillar work — that most
organisations never applied consistently. Start with a **FinOps
maturity review** as the reference point.

The quick win: once costs are visible, engineers start *thinking*
before upscaling. Awareness is the cheapest optimisation there is.
Inform never ends — visibility keeps adapting to each team's needs.

### 2. Optimize — reduce the spend

With visibility comes the target list: rightsizing, commitment
discounts, spot capacity, deleting the unattached. Every initiative
should carry its **projected savings** — nothing drives optimisation
like the number.

### 3. Operate — keep it that way

The practices that make the other pillars possible and permanent:
naming conventions, tagging policy, budgets and alerts, governance
guardrails. Without Operate, Inform's dashboards decay and Optimize's
savings leak back.

---

## Rules of thumb

- **Visibility first, always.** You cannot optimise what you cannot
  see; the Inform pillar is the first move for everyone.
- **Tag everything, bill by tag.** Tagging policy is the one Operate
  practice every other pillar stands on.
- **Measure savings per initiative.** "We do FinOps" means nothing;
  "rightsizing saved 23% in region X" means something.
- **Make cost a deploy-time signal**, not a monthly surprise — CI that
  rejects an unpinned instance size is FinOps automation.

## Why it matters

The mesh runs on hosted boxes: every station, every node, every
datacenter choice is a metered decision. FinOps is the discipline that
turns "the bill went up" into "we know exactly why, and here is the
trend" — before the bill arrives.
