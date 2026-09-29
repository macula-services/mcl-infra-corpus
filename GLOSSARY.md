---
title: mcl-infra-corpus — Glossary
layer: glossary
audience: [agent, human]
stage: stable
---

# Glossary

Canonical vocabulary for containers, Linux, IaC, observability, and
cloud. One term, one meaning.

---

## Containers

| Term | Meaning |
|------|---------|
| **Image** | An immutable filesystem snapshot plus metadata (entrypoint, env, user). Built from a Dockerfile. |
| **Container** | A running instance of an image: an isolated process with its own namespaces, cgroups, and filesystem view. |
| **Dockerfile** | The declarative build script: base image, copied files, run commands, entrypoint. |
| **Registry** | Where images are stored and pulled from (`ghcr.io`, Docker Hub). |
| **Layer** | One build step's filesystem delta. Layers cache independently; a changed line invalidates everything below it. |
| **Namespace** | Kernel isolation of a resource class: pids, network, mounts, users. |
| **cgroup** | Kernel resource accounting and limits: CPU, memory, io. |
| **Pod** | Kubernetes' unit of deployment: one or more containers sharing network namespace and volumes, scheduled together. |
| **Deployment** | A Kubernetes controller keeping a desired number of pod replicas running. |
| **Service** | A stable virtual IP + DNS name in front of a set of pods. |
| **ConfigMap / Secret** | Config and sensitive data injected into pods as env or files. |
| **Volume** | Storage mounted into a container; can outlive the container (persistent volume). |
| **Containerfile** | The Podman name for a Dockerfile (OCI-neutral). |
| **OCI** | Open Container Initiative: the image and runtime standard both Docker and Podman implement. |

## Linux

| Term | Meaning |
|------|---------|
| **systemd** | The init system and service manager: units, targets, journald logging, timers. |
| **Unit** | A systemd-managed thing: service, timer, mount, socket, path. |
| **journald** | systemd's structured logging daemon; `journalctl` reads it. |
| **cron / systemd timers** | Scheduled task runners. |
| **LVM** | Logical Volume Manager: pooling physical volumes into resizeable logical ones. |
| **firewalld / nftables** | Host firewall layers. |
| **SELinux** | Mandatory access control labelling processes and files. |
| **rsyslog** | The classic syslog daemon. |

## IaC

| Term | Meaning |
|------|---------|
| **Infrastructure as Code (IaC)** | Machines described as versioned text, applied by a tool, reconciled. |
| **Terraform** | Declarative, provider-based IaC: state file, plan, apply. |
| **State** | Terraform's record of what it manages; drift is the difference between state and reality. |
| **Provider** | A Terraform plugin for one platform (AWS, Proxmox, GitHub…). |
| **Module** | A reusable Terraform bundle of resources with inputs and outputs. |
| **Ansible** | Agentless configuration management over SSH: inventories, playbooks, roles. |
| **Playbook** | A list of plays; a play targets a host group with roles/tasks. |
| **Inventory** | The host list Ansible acts on, static or dynamic. |
| **Idempotent** | Applying twice changes nothing the second time — the IaC contract. |

## Observability

| Term | Meaning |
|------|---------|
| **Metric** | A number measured over time (CPU %, request count). |
| **Log** | A discrete, timestamped record of an event. |
| **Trace** | A request's path across services, with spans per hop. |
| **Dashboard** | A composed view of panels over data sources. |
| **Alert** | A rule over data producing a notification; alerting = query + threshold + route. |
| **SLO** | Service level objective: the reliability target (99.9% of requests < 300 ms). |
| **Golden signals** | Latency, traffic, errors, saturation — the four metrics to watch first. |
| **Prometheus** | Pull-based metrics store with a query language (PromQL). |
| **Grafana** | The visualization and alerting layer over many data sources. |

## Cloud & FinOps

| Term | Meaning |
|------|---------|
| **Multi-cloud** | One system spanning several cloud providers deliberately, for resilience or leverage. |
| **Landing zone** | The pre-provisioned cloud foundation: identity, networking, logging, guardrails. |
| **FinOps** | The practice of making cloud spend a first-class engineering concern: measure, inform, optimise. |
| **Chargeback / showback** | Attributing cloud cost to teams (billed / reported). |
| **Commitment discount** | Reserved capacity or savings plans for predictable workloads. |
| **Rightsizing** | Matching instance size to actual usage. |
| **Spot / preemptible** | Discounted interruptible capacity for fault-tolerant workloads. |
