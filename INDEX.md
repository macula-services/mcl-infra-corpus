---
title: mcl-infra-corpus — Index
layer: index
audience: [agent, human]
stage: stable
---

# mcl-infra-corpus

*Knowledge corpus for infrastructure: containers, Linux, IaC, observability, cloud, and infra tooling.*

This index maps every note in the corpus. An agent answering a question
recalls from this repo through mcl-rag; this file is the human-readable
map of what is here and where a fuller answer lives.

---

## Domain map

### Containers — `containers/`

The runtime layer services ship as.

| Note | Covers |
|------|--------|
| [README](containers/README.md) | The runtime layer in one page |
| [DOCKER](containers/DOCKER.md) | Namespaces, cgroups, the runtime stack, images vs containers |
| [KUBERNETES](containers/KUBERNETES.md) | The control plane, desired state, the reconciliation loop |

### Linux — `linux/`

The host everything runs on.

| Note | Covers |
|------|--------|
| [README](linux/README.md) | The host layer in one page |
| [SYSTEMD](linux/SYSTEMD.md) | Units, journald, timers vs cron |

### IaC — `iac/`

Infrastructure as declarative text.

| Note | Covers |
|------|--------|
| [README](iac/README.md) | The IaC discipline in one page |
| [IAC_PRINCIPLES](iac/IAC_PRINCIPLES.md) | Declarative state, idempotency, immutability, drift |
| [TERRAFORM](iac/TERRAFORM.md) | Providers, state, plan/apply |
| [ANSIBLE](iac/ANSIBLE.md) | Agentless SSH, inventory, playbooks |
| [BARE_METAL_PROVISIONING](iac/BARE_METAL_PROVISIONING.md) | MAAS: enroll, inventory, commission, deploy, release |

### Observability — `observability/`

Watching what runs.

| Note | Covers |
|------|--------|
| [README](observability/README.md) | The observability surface in one page |
| [OBSERVABILITY](observability/OBSERVABILITY.md) | The three pillars and beyond, personas, golden signals |

### Cloud & FinOps — `cloud/`

Cost and the multi-provider reality.

| Note | Covers |
|------|--------|
| [README](cloud/README.md) | Cost as engineering in one page |
| [FINOPS](cloud/FINOPS.md) | Inform, Optimize, Operate; showback vs chargeback |
| [MULTI_CLOUD](cloud/MULTI_CLOUD.md) | DDD boundaries, API-first, data gravity |

### Tooling — `tooling/`

The tooling language and its patterns.

| Note | Covers |
|------|--------|
| [README](tooling/README.md) | The tooling layer in one page |
| [GO_FOR_INFRA](tooling/GO_FOR_INFRA.md) | Why Go owns the CLI niche; the house patterns |

---

## Reading path — agent

1. [`GLOSSARY.md`](GLOSSARY.md) for the vocabulary.
2. The domain README closest to the question.
3. The specific note, if the README points at one.

## Reading path — human

1. [`README.md`](README.md) → this index → the domain that interests you.
2. Notes are short and standalone; there is no required order.
