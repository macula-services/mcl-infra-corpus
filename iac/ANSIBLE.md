---
title: "IaC: Ansible"
layer: guide
audience: [agent, human]
stage: stable
---

# IaC: Ansible

*Configuration management without an agent: from one control machine, Ansible reaches hosts over SSH, runs small modules that each bring one thing into the wanted state, and reports what it had to change.*

---

## How it works

You install Ansible on a **control node** (Linux, macOS, or Windows via
WSL). The **managed nodes** need no Ansible install, only SSH access and,
for most modules, a Python interpreter. For each task Ansible copies a
module to the host, runs it, collects a JSON result and cleans up.

The vocabulary:

| Term | Meaning |
|------|---------|
| **Inventory** | The hosts and groups to act on; a static INI/YAML file or a dynamic plugin that asks a cloud API |
| **Module** | A unit of work: `package`, `copy`, `template`, `systemd_service`, `user`... |
| **Task** | One module call with its arguments |
| **Play** | A set of tasks applied to a group of hosts |
| **Playbook** | A YAML file of one or more plays, run with `ansible-playbook` |
| **Role** | A packaged, reusable set of tasks, templates, handlers and defaults |
| **Handler** | A task that runs only when notified by a task that changed something |

---

## Desired state per task

Modules are written to describe an outcome, not an action:

```yaml
- name: Configure chrony
  hosts: stations
  become: true
  tasks:
    - name: chrony is installed
      ansible.builtin.package:
        name: chrony
        state: present

    - name: chrony uses our time servers
      ansible.builtin.template:
        src: chrony.conf.j2
        dest: /etc/chrony/chrony.conf
      notify: restart chrony

  handlers:
    - name: restart chrony
      ansible.builtin.systemd_service:
        name: chrony
        state: restarted
```

Each task reports `ok` (already right) or `changed` (had to act). Run
the playbook twice: if the second run is all `ok`, the hosts converged
and the playbook is idempotent. Tasks that shell out
(`ansible.builtin.command`, `shell`) always report `changed` unless you
add `creates:` or `changed_when:`, which is the most common way
idempotency is lost.

Handlers run once, after the play's tasks finish, no matter how many
tasks notified them; `meta: flush_handlers` runs them earlier. So the
service above restarts only when the config file actually changed.

Before touching production, `ansible-playbook --check --diff` shows
what would change without changing it (modules that cannot predict
their effect are skipped).

---

## Layout that scales

```
site.yml                    # maps host groups to roles
inventories/
  lab/hosts.yml
  prod/hosts.yml
  prod/group_vars/all.yml
roles/
  chrony/
    tasks/main.yml
    handlers/main.yml
    templates/chrony.conf.j2
    defaults/main.yml       # overridable variables
```

- Roles are what you reuse; playbooks just assign them to groups.
- Variables live with the inventory, so the same role serves lab and
  production.
- Secrets go in Ansible Vault or an external secret store, never plain
  in `group_vars`.

---

## Ansible next to Terraform

| | Terraform | Ansible |
|---|---|---|
| Best at | Creating and removing resources through APIs | Shaping what runs on existing hosts |
| Knows current state by | A state file plus provider refresh | Checking each host at run time |
| Order | Dependency graph | Top to bottom, as written |

The usual split is "Terraform creates the machine, Ansible configures
it". Letting both manage the same thing (say, firewall rules) gives two
sources of truth that undo each other.

## How it fits the corpus

Ansible is the configuration layer in the [IaC
principles](IAC_PRINCIPLES.md) stack, sitting above
[Terraform](TERRAFORM.md) and [bare-metal
provisioning](BARE_METAL_PROVISIONING.md). Its per-task `ok`/`changed`
output doubles as a change log for a small fleet.

## Sources

- *Practical Ansible*, 2nd edition. James Freeman, Fabio Alessandro Locati, Daniel Oh. Packt Publishing, 2023. https://www.packtpub.com/product/practical-ansible-second-edition/9781805129974
- Ansible Community Documentation, "Getting started with Ansible". https://docs.ansible.com/projects/ansible/latest/getting_started/index.html
- Ansible Community Documentation, "Installing Ansible" (control and managed node requirements). https://docs.ansible.com/projects/ansible/latest/installation_guide/intro_installation.html
- Ansible Community Documentation, "How to build your inventory". https://docs.ansible.com/projects/ansible/latest/inventory_guide/intro_inventory.html
- Ansible Community Documentation, "Ansible playbooks". https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_intro.html
- Ansible Community Documentation, "Roles". https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_reuse_roles.html
- Ansible Community Documentation, "Handlers: running operations on change". https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_handlers.html
- Ansible Community Documentation, "Validating tasks: check mode and diff mode". https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_checkmode.html
