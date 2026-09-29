---
title: "IaC: Terraform"
layer: guide
audience: [agent, human]
stage: stable
---

# IaC: Terraform

*Declare resources across providers; Terraform plans the transition and reconciles through a state file. The plan is the contract.*

---

## The model

```hcl
resource "aws_instance" "web" {
  ami           = "ami-123"
  instance_type = "t3.micro"
}
```

The configuration declares **resources** (for a **provider**: AWS,
Proxmox, GitHub, DNS…). Terraform reads the current reality through the
provider, compares it with the configuration and its **state**, and
computes a **plan** — the exact transition to the declared state.

---

## The workflow

| Command | What it does |
|---------|--------------|
| `terraform init` | Downloads providers and modules, initialises the working directory. Safe to repeat. |
| `terraform plan` | Diffs state + config against reality; prints the transition. **Read it.** |
| `terraform apply` | Executes the plan. |
| `terraform destroy` | Tears down what the state manages. |

The discipline is in the plan: *fully understand the plan output before
applying* — an unexpected deletion line in the plan is drift or a
mistake caught before it hits production.

---

## State — the single source of truth

The **state file** records what Terraform manages: resource ids, the
mapping between config and reality. Two rules follow:

- **State is where the power lives.** Losing it orphanes live
  infrastructure; store it remotely (a backend), lock it against
  concurrent applies.
- **Secrets in state leak.** Sensitive outputs must be marked
  `sensitive`, or better, kept out of state entirely.

Drift is simply *reality − state*: changes made outside Terraform show
up in the next plan as differences to reconcile.

---

## Modules and structure

A **module** bundles resources with inputs and outputs — the function
of IaC. The standard layout:

```
environments/staging/main.tf     # thin: which modules, which values
modules/web/                     # the reusable bundle
```

Rules of thumb:

- **Environments are directories, not branches.** Each environment
  dir pins module versions; branches that diverge become drift.
- **Pin provider and module versions.** An unpinned provider upgrades
  under you mid-apply.
- **Resources are nouns, plans are diffs.** If you cannot read the
  plan, your abstraction layer is too clever.

## Why it matters

Terraform is the mesh's answer to "who owns this box's resources and
how did it get this way" — the same question IaC answers everywhere.
The state file is what makes the answer checkable: every resource is
declared, every change is a reviewed diff.
