---
title: IaC
layer: index
audience: [agent, human]
stage: stable
---

# IaC

*Infrastructure as code: machines described as versioned text, applied by tools, reconciled.*

---

## Notes

| Note | Covers |
|------|--------|
| [IAC_PRINCIPLES](IAC_PRINCIPLES.md) | Declarative state, idempotency, immutability, drift |
| [TERRAFORM](TERRAFORM.md) | Providers, state, plan/apply |
| [ANSIBLE](ANSIBLE.md) | Agentless SSH, inventory, playbooks |

## One-page summary

IaC manages infrastructure through **machine-readable definition
files** instead of consoles and SSH sessions. The two key principles:
**idempotency** (applying twice changes nothing the second time) and
**immutability** (replace, do not mutate — the cure for configuration
drift). Terraform provisions by reconciling a state file against
providers; Ansible configures over SSH, agentless, task by idempotent
task.
