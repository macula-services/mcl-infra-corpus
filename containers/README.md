---
title: Containers
layer: index
audience: [agent, human]
stage: stable
---

# Containers

*The runtime layer: images, containers, and the orchestrator that runs them at scale.*

---

## Notes

| Note | Covers |
|------|--------|
| [DOCKER](DOCKER.md) | Namespaces, cgroups, the runtime stack, images vs containers |
| [KUBERNETES](KUBERNETES.md) | The control plane, API objects as desired state, the reconciliation loop |

## One-page summary

A **container** is one or more processes sandboxed by Linux **namespaces**
and limited by **cgroups**. An **image** is the immutable template; the
**runtime** (containerd + runc) turns it into containers; the **engine**
adds networking, plugins, and an API. **Kubernetes** runs containers in
**pods** on many machines, driven by a control plane that reconciles
**desired state** (API objects in etcd) with **actual state** (kubelets
on every node) — forever.
