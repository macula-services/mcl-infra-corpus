---
title: "IaC: Bare-Metal Provisioning"
layer: guide
audience: [agent, human]
stage: stable
---

# IaC: Bare-Metal Provisioning

*Metal as a Service: machines as cattle before the hypervisor. Provision physical servers with the same declarative loop as cloud instances.*

---

## The problem

Cloud instances provision themselves. Physical servers do not: a new
box arrives with nothing on it, and someone must discover it, install
an OS, and configure it — by hand, per machine, historically. That
hand work is exactly the drift and delay IaC exists to eliminate;
bare-metal provisioning brings the same declarative loop to the
hardware itself.

**MAAS (Metal as a Service)** is the reference implementation of the
idea: a provisioning service that discovers machines on the network,
inventories their hardware, and deploys an OS image to them —
repeatably, and then hands the configured machine to the next layer
(Terraform, Ansible, a scheduler).

---

## The loop

1. **Enroll** — the machine boots (PXE) and is discovered by the
   provisioning service.
2. **Inventory** — CPU, RAM, disks, NICs recorded; the machine
   becomes a managed resource.
3. **Commission** — hardware is tested and accepted.
4. **Deploy** — an OS image is laid down and configured
   (networking, storage, credentials) from declarative settings.
5. **Release** — the machine returns to the pool; the cycle repeats.

From then on the machine is an addressable resource, not a rack
position: deploy, redeploy, retire — by API, in CI, like any other
infrastructure.

---

## Where it fits the stack

```
MAAS (or equivalent)      → the metal: discover, image, network
   └─ Terraform           → resources: what exists, provisioned
        └─ Ansible        → configuration: packages, services, state
             └─ workload  → containers, services (the mcl-* layer)
```

Each layer owns one resource class — the same
[one-owner rule](IAC_PRINCIPLES.md) as everywhere else: the
provisioner owns the OS image, IaC owns the resources, config
management owns the config.

## Rules of thumb

- **The pool is the unit.** Machines are interchangeable; redeploying
  one should be a non-event — the pattern that makes hardware
  failures boring.
- **Images are the contract.** A known-good OS image, versioned,
  is the reproducibility anchor — build them, do not hand-configure
  boxes.
- **Hands off the metal.** A machine that was fixed by hand is a
  machine that will surprise the redeploy — every change goes
  through the loop.
- **Provisioning is the first automation.** Everything above it
  (Terraform, Ansible) assumes a machine exists; automate the
  existing first.

## Why it matters

The mesh's fleet lives on physical boxes — stations, lab machines,
whatever hosts a node. Bare-metal provisioning is what makes that
hardware a pool instead of a collection: a station box dies, the
provisioner deploys the image, the layers above redeploy the node,
nobody touches a keyboard. That is the infra-corpus's bottom layer.
