---
name: observability-telemetry-and-debt
description: How to make a system legible and honest over time — structured logging so failures are diagnosable without a debugger, lightweight operational metrics so behavior is measurable, audit trails for state changes, and a debt ledger that tracks every deferred fix instead of carrying it in your head. Use this whenever adding logging or metrics, instrumenting an operation, or deferring/hot-fixing something you intend to clean up later.
---

# Observability, Telemetry & Debt

Expands Constitution principles 11 and 12. These three are grouped because they
are how a system stays honest after it ships.

## Observability — diagnose the failure

Structured logging at every boundary. Each significant operation records **what it
did and why**, in a form you can read without opening a debugger.

- Log at the edges (requests in, calls out, writes, state transitions), not on
  every line.
- Use structured fields, not prose strings, so entries can be filtered and
  individual fields redacted.
- A failure must be diagnosable from the logs alone. This is the rule that lets a
  person maintain the system without reading the source — treat it as a
  first-class requirement, not a nice-to-have.
- Never log secrets (see module 02).

## Telemetry — measure the operation over time

Lightweight. The system records its own key numbers: throughput, durations, error
counts, the handful of metrics that describe whether it is doing its job.

- This is three counters and a log line, **not** a monitoring stack. Do **not**
  build a time-series database or dashboards for v1 — that is later, and it is
  exactly the over-engineering to refuse.
- The payoff is triple: the numbers are your proof of value, your evidence when
  asking for investment, and your operational signal.

## Audit trails

Any action that changes external state writes an audit record before and after the
mutation: what changed, by what, with verifiable hashes where it matters. Audit is
the *what-happened* (compliance, traceability); logging above is the
*why-it-broke*. You want both.

## The debt ledger — track every deferral

No silent debt. The default failure mode is fast fixes that pile up because nobody
wrote them down.

- Every deferred fix becomes a **dated, marked entry** (a `TODO`/`DEBT` with an
  owner and a date) in a tracked ledger — not a thought in your head.
- A fix may be fast, but on the way in it must still obey the cheap rules (use the
  shared utility, take config from one place, no hardcoded path) and log its debt.
- Critical debt blocks "done": a cycle cannot be considered closed while it is
  unpaid. This is what turns honesty from a cleanup habit into a guardrail.
