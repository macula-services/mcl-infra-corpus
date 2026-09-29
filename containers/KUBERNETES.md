---
title: "Containers: Kubernetes"
layer: guide
audience: [agent, human]
stage: stable
---

# Containers: Kubernetes

*You store a description of what should be running; controllers keep comparing that description with what is running and close the gap. Everything else is detail on top of that loop.*

---

## The moving parts

**Control plane** (one or more control-plane nodes; older material calls
these "master" nodes):

| Component | Job |
|-----------|-----|
| `kube-apiserver` | The only entry point. Every read, write and watch of cluster state goes through it. |
| `etcd` | Consistent key-value store behind the API server; holds every object. |
| `kube-scheduler` | Chooses a node for each Pod that has none yet. |
| `kube-controller-manager` | Runs the built-in controllers (Deployments, ReplicaSets, Nodes, Jobs...). |
| `cloud-controller-manager` | Optional; talks to a cloud provider for load balancers, routes, node lifecycle. |

**Every node:**

| Component | Job |
|-----------|-----|
| `kubelet` | Makes sure the Pods assigned to this node are running, via the container runtime; reports status. |
| Container runtime | containerd, CRI-O or similar; actually starts containers. |
| `kube-proxy` | Optional; programs node networking so Service addresses reach Pods. |

---

## The loop

1. You submit an object, usually a YAML manifest via `kubectl apply`.
   It has a `spec` (what you want).
2. The API server validates it and stores it in etcd.
3. A controller watching that kind of object sees the change, compares
   `spec` with the observed `status`, and acts: creates Pods, deletes
   Pods, asks for a load balancer.
4. The loop never ends. If a node dies and takes a Pod with it, the
   controller notices the count is short and makes another one.

Example: a Deployment asks for 3 replicas. You delete one Pod by hand.
Within seconds a new Pod appears, because the desired count is still 3.
The only way to get 2 is to change the Deployment.

Controllers learn about changes by **watching** the API server rather
than polling it, which keeps the loop cheap even in large clusters.

---

## Objects you meet first

| Object | What it gives you |
|--------|-------------------|
| **Pod** | Smallest deployable unit: one or more containers that share a network namespace (one IP, talk over `localhost`) and can share volumes. |
| **Deployment** | N identical Pods from a template, with rolling updates and rollback. |
| **StatefulSet** | Pods with stable names and per-Pod storage, for databases and similar. |
| **Service** | A stable name for a changing set of Pods. The default `ClusterIP` type adds a virtual IP; a headless Service (`clusterIP: None`) returns Pod IPs through DNS instead. |
| **ConfigMap / Secret** | Configuration and credentials, mounted as files or env vars. Secrets are only base64-encoded unless encryption at rest is enabled. |
| **PersistentVolumeClaim** | A request for storage that outlives any single Pod. |
| **Namespace** | A scope for names, quotas and access rules inside one cluster. |

---

## Working with it, and what goes wrong

- **Change the manifest, not the cluster.** A `kubectl edit` or
  `kubectl scale` by hand is overwritten the next time the manifest is
  applied, or silently diverges from what is in git.
- **Back up etcd.** Losing it loses the cluster's memory of every
  object. Only the API server should talk to it.
- **Pods are replaceable.** Anything a Pod must keep belongs in a
  volume or outside the cluster; its IP and name will change.
- **Set resource requests and limits.** The scheduler places Pods by
  their requests; without them it packs blindly and nodes get
  overloaded.
- **It is a lot of machinery.** For a handful of boxes, containers
  under systemd or docker may be the better trade; this fleet made that
  choice.

## How it fits the corpus

Kubernetes applies the [IaC principle](../iac/IAC_PRINCIPLES.md) of
declared state and convergence to running workloads. The unit it
schedules is the image described in [Docker](DOCKER.md). The watch
mechanism is the same subscription idea that event-sourced read models
use.

## Sources

- *The Kubernetes Workshop*. Zachary Arnold, Sahil Dua, Wei Huang, Faisal Masood, Melony Qin, Mohammed Abu Taleb. Packt Publishing, 2020. https://www.packtpub.com/en-us/product/the-kubernetes-workshop-9781838820756
- Kubernetes Docs, "Kubernetes Components". https://kubernetes.io/docs/concepts/overview/components/
- Kubernetes Docs, "Controllers". https://kubernetes.io/docs/concepts/architecture/controller/
- Kubernetes Docs, "Objects In Kubernetes". https://kubernetes.io/docs/concepts/overview/working-with-objects/
- Kubernetes Docs, "Pods". https://kubernetes.io/docs/concepts/workloads/pods/
- Kubernetes Docs, "Service". https://kubernetes.io/docs/concepts/services-networking/service/
- Kubernetes Docs, "Secrets" (unencrypted in etcd by default). https://kubernetes.io/docs/concepts/configuration/secret/
- Kubernetes Docs, "Operating etcd clusters for Kubernetes". https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/
