---
title: "IaC: Terraform"
layer: guide
audience: [agent, human]
stage: stable
---

# IaC: Terraform

*You describe resources in HCL; Terraform compares that description with its state file and the real world, shows you the difference as a plan, and applies it. Reading the plan is the job.*

---

## How it works

Terraform itself knows nothing about clouds. **Providers** (plugins)
translate between Terraform and an API: AWS, Hetzner, Proxmox,
Cloudflare DNS, GitHub. You write **resources** against a provider:

```hcl
terraform {
  required_providers {
    hcloud = { source = "hetznercloud/hcloud", version = "~> 1.48" }
  }
}

resource "hcloud_server" "station" {
  name        = "station-fsn-1"
  server_type = "cx23"
  image       = "debian-12"
  location    = "fsn1"
}
```

Terraform records what it created in a **state** file, which maps
`hcloud_server.station` to the real server id. On each run it refreshes
that state from the provider, compares it with the configuration, and
proposes create, update-in-place, replace or destroy actions.

---

## The commands

| Command | Does |
|---------|------|
| `terraform init` | Installs providers and modules, configures the backend. Safe to rerun. |
| `terraform plan` | Refreshes state, diffs against the config, prints proposed actions. Changes nothing. `-out=plan.bin` saves it. |
| `terraform apply` | Makes the changes (from a fresh plan, or exactly the saved one). |
| `terraform plan -refresh-only` | Shows what changed outside Terraform, without proposing to undo it. |
| `terraform destroy` | Removes everything this state manages. |

In a pipeline, save the plan, have a human or a policy check read it,
then apply that exact file. The line to look for is a `-/+` (replace) or
`-` (destroy) you did not expect: a changed attribute that forces a
new server, for example, will destroy the old one.

---

## State

The state file is how Terraform knows what it owns.

- **Keep it remote and locked.** A backend (S3-compatible bucket, HCP
  Terraform, Postgres...) lets a team share it and stops two applies
  running at once. A lost state file leaves real resources that
  Terraform no longer knows about.
- **Assume it contains secrets.** `sensitive = true` only hides a
  value in CLI output; it is still written to state and plan files in
  plain text. To keep a value out of state entirely, use ephemeral
  values (Terraform 1.10+) or write-only arguments (1.11+) where the
  provider supports them, and restrict access to the backend.
- **Drift** shows up as a diff on the next plan: someone changed the
  resource outside Terraform. Decide whether to accept it (update the
  code) or revert it (apply).

---

## Modules and versions

A **module** is a directory of `.tf` files with input variables and
outputs; calling it is the IaC equivalent of calling a function. A
common layout keeps environments thin:

```
modules/station/        # the reusable part
envs/staging/main.tf    # calls modules/station with staging values
envs/prod/main.tf       # same module, prod values, own state
```

- **Environments as directories with separate state**, not long-lived
  git branches that drift apart.
- **Constrain provider versions and commit `.terraform.lock.hcl`.** The
  lock file fixes the exact provider versions; they only move when
  someone runs `terraform init -upgrade`.
- **Pin module versions exactly.** The lock file does not cover
  modules; a loose constraint picks up the newest match whenever modules
  are installed fresh (a new checkout, a CI runner, `init -upgrade`).
- **Keep abstractions shallow.** If a reviewer cannot tell from the
  plan what will happen, the module is hiding too much.

## How it fits the corpus

Terraform is the "what resources exist" layer of the
[IaC principles](IAC_PRINCIPLES.md): it creates machines, networks and
DNS, then hands over to [Ansible](ANSIBLE.md) for what runs on them.
Below it sits [bare-metal provisioning](BARE_METAL_PROVISIONING.md)
for hardware that no cloud API creates.

## Sources

- *Architecting AWS with Terraform*. Erol Kavas. Packt Publishing, 2023. https://www.packtpub.com/en-us/product/architecting-aws-with-terraform-9781803248561
- HashiCorp, "What is Terraform?". https://developer.hashicorp.com/terraform/intro
- HashiCorp, "terraform plan command reference". https://developer.hashicorp.com/terraform/cli/commands/plan
- HashiCorp, "State" and "State: Remote Storage". https://developer.hashicorp.com/terraform/language/state and https://developer.hashicorp.com/terraform/language/state/remote
- HashiCorp, "Manage sensitive data in your configuration". https://developer.hashicorp.com/terraform/language/manage-sensitive-data
- HashiCorp, "Dependency Lock File". https://developer.hashicorp.com/terraform/language/files/dependency-lock
- HashiCorp, "Modules overview". https://developer.hashicorp.com/terraform/language/modules
