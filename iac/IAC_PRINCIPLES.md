---
title: "IaC: Principles"
layer: guide
audience: [agent, human]
stage: stable
---

# IaC: Principles

*Infrastructure as Code means the repository, not somebody's shell history, says what the infrastructure is. Three ideas make that true in practice: declare the end state, make applying it repeatable, and replace things rather than patch them.*

---

## Declare the end state

There are two ways to write automation:

- **Imperative:** a list of steps. "Create a VM. Install nginx. Open
  port 443." Run it twice and you may get two VMs.
- **Declarative:** a description of the result. "One VM named `web1`
  exists, nginx is installed, 443 is open." The tool works out which
  steps are needed from wherever reality currently is.

Declarative code reads as documentation of the system and survives
partial failures, because the next run just computes a new set of
steps. Terraform and Kubernetes manifests are declarative; Ansible
playbooks are ordered steps, but each step is written declaratively
(`state: present`).

---

## Idempotency

An operation is idempotent when running it once or ten times leaves
the system in the same state. For IaC that means: apply, apply again,
and the second run changes nothing.

Why it matters:

- A failed or interrupted run is fixed by running it again.
- Scheduled re-applies become safe, and a non-empty second run is a
  signal that something outside the code changed things.
- Reviews compare two descriptions of the system, not two histories.

Tools get there in different ways. Terraform keeps a state file and
diffs it against the configuration and the live resources. Ansible
modules check the current value before changing it and report `ok` or
`changed` per task. A hand-written script is idempotent only if every
line checks first; `mkdir -p` is, `useradd` is not.

---

## Replace, don't patch

**Drift** is the gap between what the code says and what is running,
created every time someone fixes a box by hand. Long-lived servers
collect it until nobody can rebuild them.

**Immutable infrastructure** avoids it: a change produces a new
artifact (an image, a VM template), new instances are started from it,
and the old ones are thrown away. Nothing is edited in place, so every
running instance matches a build that can be reproduced. The price is
a build pipeline and a way to keep state (databases, volumes) outside
the replaceable part.

Few setups are purely immutable. A common compromise: immutable images
for the workload, declarative config management for the host underneath.

---

## Working rules

| Rule | Reason |
|------|--------|
| Keep all of it in version control | History, review and rollback come for free |
| One tool owns each kind of resource | Two tools managing the same thing will fight and drift |
| No secrets in the repo or in plain state | IaC artifacts get copied, cached and shared |
| Always look at the plan or dry run first | Surprises in the diff are cheap there and expensive after |
| Re-apply regularly | Drift found early is small |

## How it fits the corpus

[Terraform](TERRAFORM.md) and [Ansible](ANSIBLE.md) are the two
implementations covered here, and [bare-metal
provisioning](BARE_METAL_PROVISIONING.md) extends the same loop down to
physical machines. The "declare, then converge" shape is the same one
[Kubernetes](../containers/KUBERNETES.md) uses for workloads.

## Sources

- *Architecting AWS with Terraform*. Erol Kavas. Packt Publishing, 2023. https://www.packtpub.com/en-us/product/architecting-aws-with-terraform-9781803248561
- HashiCorp, "What is Terraform?". https://developer.hashicorp.com/terraform/intro
- HashiCorp, "State". https://developer.hashicorp.com/terraform/language/state
- Ansible Community Documentation, "Ansible playbooks". https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_intro.html
