---
title: "Containers: Kubernetes"
layer: guide
audience: [agent, human]
stage: stable
---

# Containers: Kubernetes

*API objects describe desired state; controllers reconcile reality toward it; etcd remembers the truth. The cluster is a reconciliation loop.*

---

## The control plane

| Component | Where | What it does |
|-----------|-------|--------------|
| **etcd** | Master | The key-value store holding every API object. Nothing talks to etcd except the API server. |
| **API server** | Master | The single door: all reads and writes of cluster state; supports *watching* objects for changes. |
| **Scheduler** | Master | Picks the best node for each new workload, from its view of the whole cluster. |
| **Controller manager** | Master | Watches API objects and moves current state toward desired state — the reconciliation loop. |
| **kubelet** | Every worker | Talks to the container runtime to run containers and reports status back to the API server. |

A **master node** runs the control plane; workers run the kubelet.
In production the control plane is duplicated for HA; etcd is the
piece that must never lose data.

---

## Desired state, reconciled forever

An **API object** describes how a resource should be honored —
written as a YAML manifest, applied via `kubectl`, stored in etcd.
Kubernetes then *reconciles*: the controller manager watches its
objects and makes reality match the manifest.

> A Deployment claims two replicas; one is running; the controller
> creates the second. This comparison runs for the cluster's whole
> life — applications stay in their expected state because the loop
> never stops.

That is the whole model: **declare the desired state, let the
controllers converge**. No imperative steps, no scripts — if reality
drifts, the loop pulls it back.

---

## The basic objects

| Object | Purpose |
|--------|---------|
| **Pod** | The unit of scheduling: one or more containers sharing a network namespace and volumes. A sandbox (`pause`) container boots the namespace the rest join. |
| **Deployment** | Keeps N replicas of a pod template running; rolling updates |
| **Service** | A stable DNS name + virtual IP in front of pods that come and go |
| **ConfigMap / Secret** | Config and sensitive data as env or files |
| **Volume / PVC** | Storage that outlives the pod |
| **Namespace** | A virtual cluster inside the cluster: scoping, quotas, RBAC |

---

## Rules of thumb

- **Never kubectl into the happy path.** If a fix is manual, it is not
  in the desired state, and the loop will undo it — change the
  manifest instead.
- **etcd is the crown jewels.** Back up it, never let anything but the
  API server touch it.
- **Pods are cattle.** They die and are replaced; anything a pod
  must keep lives in a volume or outside the cluster.
- **Watches, not polls.** The watch mechanism is how controllers stay
  cheap; the same idea (subscription) that powers event-sourced
  systems.
