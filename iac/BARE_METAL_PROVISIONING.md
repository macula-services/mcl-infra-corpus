---
title: "IaC: Bare-Metal Provisioning"
layer: guide
audience: [agent, human]
stage: stable
---

# IaC: Bare-Metal Provisioning

*A cloud VM exists the moment an API call returns. A physical server arrives blank. Bare-metal provisioning puts an API in front of real hardware so machines can be discovered, tested, installed and wiped the same way cloud instances are created and destroyed.*

---

## The gap it fills

Everything else in the IaC stack assumes a machine with an OS and an
SSH login already exists. On physical hardware someone has to get it
there: find the box on the network, check the hardware, install an
operating system, set up disks and network. Done by hand, each server
becomes a one-off and rebuilding it after a failure is a project.

A provisioning service automates that first step. **MAAS** (Metal as a
Service, from Canonical) is the best-known open-source example; others
in the same space include Tinkerbell, Foreman and OpenStack Ironic. The
mechanics are similar: network boot (PXE/iPXE), a small in-memory
environment for inspection, an image written to disk, and control of
the machine's power through its BMC (IPMI, Redfish).

---

## A machine's life cycle in MAAS

| State | What is happening |
|-------|-------------------|
| **New** | The machine network-booted on a MAAS-managed subnet (or was added by hand) and is known by MAC address and architecture. |
| **Commissioning** | MAAS boots it into an ephemeral image and records CPU, memory, disks and NICs; optional hardware tests run. |
| **Ready** | Passed commissioning; available in the pool. (Failures land in a **Failed** state instead.) |
| **Allocated** | Reserved for a user or project, not yet installed. |
| **Deploying / Deployed** | The chosen OS image is installed with the requested storage and network layout, plus cloud-init user data; the machine is now running it. |
| **Releasing** | Handed back; disks can be erased, and it returns to Ready. |

Side states: **Rescue mode** boots an ephemeral environment on a
deployed machine for troubleshooting, and **Broken** takes faulty
hardware out of the pool.

All of this is available through the web UI, the `maas` CLI and a REST
API, so allocation and deployment can be driven from scripts, from CI,
or from Terraform via a MAAS provider.

---

## Where it sits

```
provisioning (MAAS or similar)  hardware → running OS
  └─ Terraform                  which machines/resources exist
       └─ Ansible               packages, config, services on each host
            └─ workloads        containers and services
```

Each layer owns one thing, following the
[one-owner rule](IAC_PRINCIPLES.md). The provisioner owns "which image
is on the disk"; it should not also be where application config lives.

---

## Practices and pitfalls

- **Treat the pool as the unit.** If any Ready machine can take any
  role, a dead server means "deploy another one", not an outage.
- **Version your images.** A known OS image plus cloud-init is what
  makes two deployments identical. Custom images can be built with
  Packer templates for MAAS.
- **No hand fixes on deployed machines.** Anything changed by hand is
  lost on the next redeploy, or worse, it is the only reason the
  machine works.
- **Network and BMC setup is the hard part.** PXE needs DHCP control on
  the provisioning subnet, and power control needs reachable,
  credentialed BMCs. Consumer mini PCs often have no BMC, which limits
  automation to manual power-on.
- **Erase on release** if machines move between tenants or projects.

## How it fits the corpus

This is the bottom of the infra stack: it turns physical boxes into a
pool that [Terraform](TERRAFORM.md) and [Ansible](ANSIBLE.md) can build
on. The same declare-and-converge idea from the
[IaC principles](IAC_PRINCIPLES.md) applies, one layer lower.

## Sources

- Canonical, MAAS documentation. https://canonical.com/maas/docs/stable/
- Canonical, MAAS documentation, "The machine life-cycle". https://canonical.com/maas/docs/stable/explanation/the-machine-life-cycle/
- Canonical, *Server provisioning: What Network Admins and IT Pros need to know* (eBook), listed on the MAAS resources page. https://canonical.com/maas/resources
- Canonical, Packer templates for MAAS images. https://github.com/canonical/packer-maas
- Terraform Registry, MAAS provider (canonical/maas). https://registry.terraform.io/providers/canonical/maas/latest
