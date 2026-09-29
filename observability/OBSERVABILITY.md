---
title: "Observability: The Three Pillars"
layer: guide
audience: [agent, human]
stage: stable
---

# Observability: The Three Pillars

*Metrics, logs, and traces — the golden triangle. Each answers a different question; none answers all three.*

---

## The three pillars

| Signal | Nature | Answers |
|--------|--------|---------|
| **Metric** | A number measured over time (CPU %, request count, latency) | "Is it wrong?" — cheap, aggregate, alertable |
| **Log** | A discrete timestamped record of an event | "What happened?" — detail, but per instance, noisy at scale |
| **Trace** | A request's path across services, spans per hop | "Where in the journey?" — latency attribution, dependency maps |

Metrics spot the problem; traces localize it; logs explain it. An
observability platform that links them lets you click from a spiking
metric to the traces, then to the exact log lines of the slow span.

---

## Beyond the triangle

The three pillars are not the only signals — the right signal depends
on the abstraction layer you observe:

| Signal | Layer | Use |
|--------|-------|-----|
| **Profiling data** | CPU/RAM stack traces | Find the hot function; with cloud billed hourly, this creates cost savings directly |
| **Events** | Platform level (CI/CD, deploys) | "A deploy happened 3 minutes before the incident" — MTTR's best friend; a whole action vs five logs of its stages |

---

## The personas — design from the questions

The platform's design starts from *who asks what*:

| Persona | Typical question |
|---------|------------------|
| **Developer** | "Is my service doing what I expect under load?" |
| **Operator / SRE** | "Which service broke first, and how do I roll back?" |
| **Service manager** | "Are we meeting the SLO this week?" |
| **Product** | "Which feature do users actually reach?" |
| **Manager** | "What is the cost of this platform, and its value?" |

Each persona needs a different slice of the same data — the dashboard
for the operator is not the dashboard for the product owner. Build
persona-first, not metric-first.

---

## The golden signals

Whatever the stack, watch these four first: **latency, traffic,
errors, saturation.** They cover most failure modes, they are cheap to
collect, and they are the base every alerting policy should start from.

## Rules of thumb

- **Alert on symptoms, not causes.** "Requests failing" is a symptom;
  "disk full" is a cause. Users feel symptoms.
- **Link the pillars.** A metric dashboard that cannot jump to traces
  and logs is three silos, not observability.
- **Logs are streams, not files.** Structure them (JSON), tag them
  with service/instance/trace id — the link is what makes them useful.
- **Instrument the boundary.** Outbound calls and queue operations are
  where latency hides; instrument them before the internals.
