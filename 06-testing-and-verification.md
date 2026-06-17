---
name: testing-and-verification
description: How to test and verify work honestly — deterministic tests for pure logic, testable seams via dependency injection, fakes instead of real infrastructure, pre-registered PASS/PARTIAL/KILL criteria for any batch or sync job, and explicit reporting of what was not measured. Use this whenever writing tests, designing code to be testable, or running any batch/validation/sync pass that needs a defined success bar.
---

# Testing & Verification

Expands Constitution principles 2 and 13.

## Test pure logic deterministically

- The core (pure functions) gets thorough tests with **hand-computed, known-answer
  fixtures**. If you can state the expected output by hand, assert it exactly.
- Pure functions receive clock, randomness, and IDs as **arguments** — never call
  global `now()` / `random()` / `uuid()` inside them. That is what makes them
  deterministic.

## Testable seams (dependency injection)

External dependencies (database, sensor, network, filesystem, time) sit behind an
interface so a **fake** can be substituted in tests.

- Define the dependency as a protocol/interface; ship a real implementation and a
  fake implementation.
- Tests run against the fake — no real database, no network, no device required.
- This is not extra work bolted on; it is *why* the suite is fast and trustworthy,
  and it falls naturally out of the layered spine (adapters are swappable).

## What not to test

- Don't unit-test I/O and UI directly — test the pure logic they call.
- Don't write fragile tests for things the runtime already guarantees. If a test is
  platform-fragile and adds no signal, skip it **and write down why** you skipped
  it. A documented decision to omit a test is rigor; a silent gap is not.

## Pre-registered success criteria

Any batch job, sync loop, or validation pass declares its bar **before** it runs:

- **PASS** — what counts as success.
- **PARTIAL** — degraded but acceptable, and what opens next.
- **KILL** — the failure rate that aborts immediately.

Put these in the job's docstring or spec, not in your head, so the goalposts can't
move after you see the result.

## Report what was not measured

The result of any run states explicitly what it did **not** check, alongside what
it found. A result object carries a `not_measured` field. Unreported scope is not
implied coverage — the most honest line in a report is often the one listing the
gaps.
