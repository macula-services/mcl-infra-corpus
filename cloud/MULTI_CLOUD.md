---
title: "Cloud: Multi-Cloud"
layer: guide
audience: [agent, human]
stage: stable
---

# Cloud: Multi-Cloud

*Running on more than one provider should be a decision with a named reason, not something that happened because two teams picked different vendors. When it is deliberate, it rests on the same things that make any distributed system portable: clear boundaries, owned interfaces, and careful data placement.*

---

## Terms

| Term | Means |
|------|-------|
| **Hybrid cloud** | Your own hardware or private cloud combined with one or more public providers |
| **Multi-cloud** | Workloads spread across two or more providers on purpose |
| **Poly-cloud / best-of-breed** | Each workload on the provider that suits it best, with little cross-provider traffic |
| **Portable / active-active** | The same workload able to run, or running, on several providers at once |

Most real setups are best-of-breed with a few portable pieces. Fully
portable active-active across providers is rare and expensive.

---

## Good reasons, and weak ones

Good reasons are specific: surviving the loss of a whole provider or
region, a regulator or customer requiring a jurisdiction or a second
supplier, a capability only one provider offers, bargaining power at
large spend, or simply that the best-priced box for this job is at a
different host.

Weak reasons are vague: "avoid lock-in" with no plan for what would
trigger a move, or copying the same stack onto two providers without
changing the design. That doubles the operational work and buys very
little.

---

## Design principles

1. **Cut along business boundaries.** Split the system into bounded
   contexts that own their data and expose a contract (see the domain
   modelling notes in the beam corpus). A context can then live on, or
   move to, any provider. Splitting by provider instead ("everything on
   A talks to everything on B") makes every change a cross-provider one.
2. **Own your interfaces.** Put provider services (queues, object
   storage, secrets) behind interfaces you define, in the place where
   you use them. Moving provider then means rewriting an adapter, not
   the application. Do this for the services you actually depend on,
   not as a blanket abstraction over everything.
3. **Pick portable foundations first.** Containers, a common
   orchestrator or process supervisor, open protocols and IaC tools
   that span providers (Terraform, Ansible) are what make placement a
   choice. Deciding the provider first and the technology second tends
   to bake that provider in.
4. **Place data deliberately.** Data is the hardest thing to move:
   egress is billed, latency grows with distance, and consistency across
   providers is expensive. Keep data next to its main consumers,
   replicate across providers only where the reason justifies it
   (disaster recovery, compliance), and design the rest for eventual
   consistency.
5. **One way to observe and deploy.** A single pipeline, one
   observability stack and one identity model across providers, or the
   second provider becomes a second, poorly-run operation.

---

## Costs to put on the table

| Cost | What it looks like |
|------|--------------------|
| Abstraction | Adapters and portability layers you build and maintain |
| Data transfer | Cross-provider traffic billed per GB, often the biggest surprise |
| Skills | Each provider has its own IAM, networking and failure modes |
| Discounts | Split spend earns smaller commitment discounts on each side |
| Lowest common denominator | Avoiding provider-specific features can mean giving up the best tool |

## Rules of thumb

- **Default to one provider** until a named requirement says otherwise.
- **Price the failover before promising it.** Measure egress and
  standby cost; a DR setup that costs more than the outage it prevents
  is not worth running.
- **Test the move.** Portability that has never been exercised usually
  is not there.

## How it fits the corpus

The fleet already runs on boxes at several hosts, so these principles
apply to everyday placement decisions: which box where, what may move,
and what that costs. The money side is in [FinOps](FINOPS.md); the
tools that make placement repeatable are in
[IaC principles](../iac/IAC_PRINCIPLES.md) and
[Terraform](../iac/TERRAFORM.md).

## Sources

- *Multi-Cloud Handbook for Developers*. Subash Natarajan, Jeveen Jacob. Packt Publishing, 2024. https://www.packtpub.com/en-us/product/multi-cloud-handbook-for-developers-9781804618707
- CNCF Technical Oversight Committee, "Cloud Native Definition". https://github.com/cncf/toc/blob/main/DEFINITION.md
- FinOps Foundation, "FinOps Framework Overview" (cost allocation and rate optimisation across providers). https://www.finops.org/framework/
- HashiCorp, "What is Terraform?" (one workflow across providers). https://developer.hashicorp.com/terraform/intro
