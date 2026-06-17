---
name: architecture-and-structure
description: How to structure a project and place code correctly — the layered spine, inward dependency direction, module boundaries, file/function size limits, where new code belongs, and how to declare scope boundaries. Use this whenever designing a project's structure, creating a new module, deciding where a piece of code goes, or reviewing whether files have grown too large or coupled.
---

# Architecture & Structure

Expands Constitution principles 1, 2, 6, and 14.

## The spine: layers with inward dependencies

Three rings. Dependencies point **inward only**.

- **Core / domain** — the rules of the problem, expressed as pure logic. No I/O,
  no database, no network, no framework, no UI. This is the part you test hardest.
- **Application / services** — orchestration. Coordinates the core to perform a
  use case ("receive a shipment", "process an order"). Depends on the core.
- **Adapters / infrastructure** — the edge. Database, files, HTTP, message queues,
  scanners, printers, the UI, third-party APIs. Adapters depend on the core; the
  core must never import an adapter.

**The rule to enforce:** the core never imports outward. An adapter can know about
the core; the core cannot know about adapters. If your domain logic imports your
database module, the layering is inverted — fix it before continuing.

This single arrangement gives you testability (adapters swap for fakes), low
coupling (edges change without touching the core), and a clear home for everything.

## Module boundaries

Every module opens with a docstring/comment declaring three things:

- **Owns** — what this module is responsible for.
- **Must not** — what it is forbidden to do (e.g., "must not touch the DB",
  "must not import UI modules").
- **May import** — its allowed dependencies.

If you cannot write the "must not" line without contradicting the code, the module
has collapsed concerns — split it before adding more.

## Where new code goes

Before writing a new function, find its home. If it fits an existing module, put it
there. If it fits nowhere, that is the signal to create a new focused module — it
is **never** a reason to grow the nearest large file.

## Size limits (a deterministic gate)

- No source file over **~400 lines**.
- No function over **~60 lines**.

These are mechanical and checkable. When a file approaches the cap, a concern has
collapsed in — split before the line, not after.

## Declare scope boundaries

Write down what the design assumes and excludes, in the README or the module
header: single-writer vs. concurrent, auth deferred, single machine, no
internationalization, etc. Stated assumptions are engineering; discovered ones are
incidents. Out-of-scope is a decision, not an oversight — record it as one.
