---
title: "Observability: The Three Pillars"
layer: guide
audience: [agent, human]
stage: stable
---

# Observability: The Three Pillars

*Metrics tell you something is off, traces tell you where, logs tell you why. Observability is having all three, correlated, so you can answer a question you did not plan for.*

---

## The three core signals

| Signal | Shape | Good for | Weak at |
|--------|-------|----------|---------|
| **Metrics** | Numeric time series with labels (`http_requests_total{status="500"}`) | Dashboards, alerts, trends; cheap to store and query | Explaining a single failure; high-cardinality labels blow up cost |
| **Logs** | Timestamped records of individual events | Detail about one request or error | Aggregation at scale; volume and cost |
| **Traces** | A tree of timed spans following one request across services | Finding which hop is slow or failing | Everything outside the request path; usually sampled |

A typical investigation: an alert fires on the error-rate metric, an
exemplar or trace id leads to a slow trace, the slow span's trace id
finds the log lines that explain it. That hand-off only works if the
signals share identifiers (service name, instance, trace id).

OpenTelemetry standardises how all three are produced and shipped
(with baggage for passing context along). In the Grafana stack they are
stored by Prometheus or Mimir (metrics), Loki (logs) and Tempo (traces),
with Grafana on top for querying and correlation.

---

## Other useful signals

- **Continuous profiling**: sampled stack traces over time (Grafana
  Pyroscope, OpenTelemetry profiles, still in development). Answers
  "which function is burning the CPU", which on metered hosts is also a
  cost question.
- **Change events**: deploys, config changes, feature-flag flips shown
  as annotations. A large share of incidents start with a change, so
  seeing "deploy at 14:02" next to "errors from 14:03" shortens most
  investigations.

---

## Start from the questions

Different people ask different things of the same data:

- someone who wrote the service: is my change behaving under real load?
- whoever is on call: what broke first, and what do I roll back?
- someone owning a service level: are we inside our error budget?
- someone paying for it: what does this cost, and is it worth it?

Build dashboards and alerts for those questions, not a wall of every
metric that happens to be exported.

---

## The four golden signals

For any request-serving system, the Google SRE book recommends watching
**latency** (separating successful from failed requests), **traffic**,
**errors** and **saturation**. They cover most user-visible failures
and make a sensible first dashboard and alert set.

---

## Practices and pitfalls

- **Page on symptoms, investigate causes.** Alert on what users feel
  (errors, latency); a full disk or a restarting pod is a cause to show
  on a dashboard, and to alert on only when it is certain and imminent.
- **Structure logs.** Key-value or JSON with service, instance and
  trace id; free text cannot be joined to anything.
- **Watch label cardinality.** A user id or request id as a metric
  label creates a series per value and can take down the metrics store.
- **Instrument the edges first.** Inbound requests, outbound calls,
  queues and database calls are where latency and errors show up.

## How it fits the corpus

Metrics, logs and traces are what make the services deployed with
[Docker](../containers/DOCKER.md) and supervised by
[systemd](../linux/SYSTEMD.md) debuggable; journald is the first log
source on every box. The [Go tooling](../tooling/GO_FOR_INFRA.md) note
applies the same idea to the tools themselves.

## Sources

- *Observability with Grafana*. Rob Chapman, Peter Holmes. Packt Publishing, 2024. https://www.packtpub.com/en-us/product/observability-with-grafana-9781803248004
- Betsy Beyer, Chris Jones, Jennifer Petoff, Niall Richard Murphy (eds.), *Site Reliability Engineering*, O'Reilly Media, 2016; chapter "Monitoring Distributed Systems" (free online, CC BY-NC-ND 4.0). https://sre.google/sre-book/monitoring-distributed-systems/
- OpenTelemetry, "Signals". https://opentelemetry.io/docs/concepts/signals/
- Grafana Labs, "Introduction" (Grafana fundamentals). https://grafana.com/docs/grafana/latest/fundamentals/
- Grafana Labs, "About Grafana". https://grafana.com/docs/grafana/latest/introduction/
- Grafana Labs, "Grafana Pyroscope" (continuous profiling). https://grafana.com/docs/pyroscope/latest/
