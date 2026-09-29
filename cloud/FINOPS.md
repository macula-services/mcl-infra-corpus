---
title: "Cloud: FinOps"
layer: guide
audience: [agent, human]
stage: stable
---

# Cloud: FinOps

*Metered infrastructure turns every engineering choice into a spending choice. FinOps is the practice of making that spend visible, reducing it where it buys nothing, and keeping it under control, with engineers, finance and product owning it together.*

---

## Why it exists

With owned hardware, cost is decided once at purchase and then mostly
fixed. With cloud and rented servers it is decided continuously: a
bigger instance, a forgotten volume, a chatty cross-region link all
show up on next month's invoice, and the person who made the choice
usually never sees it. FinOps closes that loop so the people making
technical decisions see their cost and can weigh it against value.

The FinOps Foundation's framework is the common reference. It
describes six principles (among them: teams collaborate, business
value drives decisions, everyone owns their usage, data is accessible
and timely), personas, a crawl/walk/run
maturity model, and three **phases** that teams cycle through
repeatedly and in parallel rather than once in order.

---

## The three phases

### Inform: who spends what

Get cost and usage data into a shape people can act on: allocated to
teams, products and environments, close to real time, with forecasts
and budgets alongside.

- **Allocation** depends on consistent tags or labels and account or
  project structure. It is only as good as the tagging discipline
  behind it.
- **Showback** reports each team's cost back to it; **chargeback**
  actually bills it internally. Showback alone already changes
  behaviour, because people start checking before they upsize.
- Shared costs (support contracts, networking, platform teams) need an
  agreed split rule, or allocation arguments never end.

### Optimize: spend less for the same result

Two levers:

- **Usage**: rightsize over-provisioned machines, switch off idle
  non-production at night, delete orphaned disks, snapshots and IPs,
  pick cheaper storage tiers, cut unnecessary data transfer.
- **Rate**: pay less per unit through commitments (reserved capacity,
  savings plans, committed-use discounts), spot or preemptible capacity
  for interruptible work, and negotiated pricing.

Put an expected saving on every proposal and check it afterwards, so
effort goes where the money is.

### Operate: make it stick

Turn one-off wins into routine: tagging policy enforced in IaC or at
creation time, budgets with alerts, anomaly detection, regular reviews
of the biggest line items, and cost as an input to architecture
decisions rather than an audit afterwards.

---

## Practices and pitfalls

- **Visibility before optimisation.** Without allocation you cannot
  tell which saving matters.
- **Enforce tags in code.** A Terraform or policy check that rejects
  untagged resources beats a monthly clean-up.
- **Beware commitments on shifting workloads.** A three-year commitment
  for a service you will re-architect next year is a cost, not a saving.
- **Unit cost beats total cost.** "Cost per active station" or "per
  thousand requests" shows efficiency; a rising total can be healthy
  growth.
- **Egress is the usual surprise.** Data leaving a provider or crossing
  regions is billed and easy to overlook at design time.

## How it fits the corpus

Every box in the fleet is a metered decision, and placement choices
across providers are covered in [Multi-Cloud](MULTI_CLOUD.md). Tag
enforcement belongs in [Terraform](../iac/TERRAFORM.md), and cost
dashboards sit naturally next to the signals in
[Observability](../observability/OBSERVABILITY.md).

## Sources

- *Efficient Cloud FinOps*. Alfonso San Miguel Sánchez, Danny Obando García. Packt Publishing, 2024. https://www.packtpub.com/en-us/product/efficient-cloud-finops-9781805122579
- FinOps Foundation, "FinOps Framework Overview". https://www.finops.org/framework/
- FinOps Foundation, "FinOps Phases". https://www.finops.org/framework/phases/
- FinOps Foundation, "FinOps Principles". https://www.finops.org/framework/principles/
- FinOps Foundation, "FinOps Maturity Model". https://www.finops.org/framework/maturity-model/
