---
title: "IaC: Ansible"
layer: guide
audience: [agent, human]
stage: stable
---

# IaC: Ansible

*Agentless configuration over SSH: inventories say where, playbooks say what, tasks are idempotent steps.*

---

## The model

Ansible needs nothing installed on the target: it connects over SSH,
pushes modules, runs them, removes them. No agents to maintain, no
daemon to drift — the control machine is the only requirement.

Three concepts:

| Concept | Answers |
|---------|---------|
| **Inventory** | *Where* — the host list, static or dynamic (cloud APIs) |
| **Playbook** | *What* — plays, each targeting a host group with tasks |
| **Role** | Reusable bundles of tasks, templates, handlers |

```yaml
- hosts: webservers
  roles:
    - nginx
```

---

## Tasks are idempotent steps

Every module is written to the same contract: run it twice, the second
run changes nothing.

```yaml
- name: nginx is installed and current
  ansible.builtin.package:
    name: nginx
    state: present
```

The module checks before it changes — `state: present` means "ensure
this exists", not "install unconditionally". This is Ansible's
idempotency story: per task, not per run. The playbook's log shows
`ok` vs `changed` per task, and a second run that is all `ok` is the
proof the system converged.

---

## Structure that scales

```
site.yml              # top-level playbook: which hosts get which roles
inventories/
  staging/
  production/
roles/
  nginx/
    tasks/main.yml
    templates/nginx.conf.j2
    handlers/main.yml
```

- **Roles are the unit of reuse**; playbooks compose them.
- **Handlers** run once, at the end, when notified (restart a service
  only if config changed).
- **Templates** (Jinja2) render config from variables — the config is
  data, the role is the shape.

---

## Terraform vs Ansible

| | Terraform | Ansible |
|---|---|---|
| Owns | Provisioning: VMs, networks, cloud resources | Configuration: packages, config files, services |
| Mechanism | State file diffed against providers | SSH + idempotent modules |
| Convergence | Plan → apply | Task-by-task |
| When | Resources appear and disappear | Resources are shaped in place |

The common split: **Terraform creates, Ansible configures.** Overlap
them and two tools own the same resource — the fast path to drift.

## Why it matters

Ansible is the lowest-friction way to make a set of hosts *identical*:
one inventory, one playbook, SSH, done. For a fleet of lab boxes it
replaces hand-crafted provisioning notes with a runnable spec — and
the `ok`/`changed` log line is the audit trail of every change.
