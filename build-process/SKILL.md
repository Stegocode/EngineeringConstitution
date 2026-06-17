---
name: build-process
description: How to approach building any software project from the start, and how to bring an existing project up to standard — the order of operations, what to lock down first, and a section-by-section hardening pass. Use this when starting a new project, planning a build, or refactoring/cleaning up an existing codebase to meet the engineering framework.
---

# Build Process

How to approach a build so the framework is enforced from line one — not
discovered as debt later. Applies the whole constitution; see the per-area modules
for detail.

## Before any code: set the rules, then the spine

1. **Load the constitution.** It is the standard the build is checked against.
2. **Pick the spine.** Decide the three layers for this project (core / services /
   adapters) and the dependency direction. Everything hangs off this. (Module 01.)
3. **Define the canonical data model once.** The core entities and their schema,
   in one place, before features. (Module 03.)
4. **Stand up config first.** One config source, fail-fast validation, an
   `.env.example`. Prove "clone and run" works on an empty shape before building
   on top of it. (Module 02.)
5. **Declare scope boundaries.** Write down assumptions and out-of-scope items
   (single-writer, no auth yet, etc.). (Module 01.)
6. **Put the gate in.** Make the cheap rules mechanical from the first commit:
   one shared write/IO utility, no hardcoded paths, file/function size caps, no
   import cycles. These are greppable and should fail the build if violated. This
   is the difference between clean-by-construction and clean-up-later.

Then build features inside that frame. New code obeys the rules going in; it does
not get cleaned up afterward.

## Starting a new project — the checklist

- [ ] Layered structure with the dependency direction declared
- [ ] Canonical schema with a version and a validator
- [ ] One config module; nothing else reads the environment; `config.validate()` is
      the first line of every entry point
- [ ] `.env.example` committed; `.env` gitignored; no secrets in the repo
- [ ] One shared utility for atomic writes and any 2+-caller helper
- [ ] No company/client/system names in logic — domain vocabulary only
- [ ] Structured logging at boundaries; the handful of operational metrics that matter
- [ ] A debt ledger exists (even if empty) and a `TODO`/`DEBT` convention is set
- [ ] Pure logic isolated and tested with known-answer fixtures
- [ ] Tests run against fakes — no real infrastructure required
- [ ] Size caps and "no `/Users/` literal" enforced mechanically
- [ ] README: what it does, how to run it, the architecture in three sentences

## Hardening an existing project — order matters

Each step is a self-contained pass. Freeze the old behavior as the reference; the
existing tests are the contract.

1. **Structure** — move files to match the layered shape. Don't rename logic yet.
2. **Config** — consolidate every environment read and hardcoded path into one
   config module; add `validate()`; add `.env.example`.
3. **Fail-fast wiring** — call `validate()` first in every entry point.
4. **Names** — strip company/client/system names from logic; push them to config.
5. **Boundaries** — add the owns / must-not / may-import header to every module.
6. **Duplication** — find functions that do the same thing across files; extract one
   shared helper. (The atomic write is almost always the first offender.)
7. **Size** — split every file over the cap, largest concern first.
8. **Schema** — add version + validator; mirror it across any language boundary.
9. **Safety** — grep for `except: pass`, string-built SQL, environment reads outside
   config, secrets in logs. Fix each.
10. **Observability + debt** — add structured logging and the operational metrics;
    open the debt ledger and record what you're deferring.
11. **Tests** — fakes for the infrastructure, known-answer tests for pure logic, at
    least one test that exercises a KILL-threshold abort.
12. **Docs** — README and a short operator runbook (how to run it, how to recover it,
    how to restore from backup).

A build is "done" when the tests pass, the gate is clean, it runs unattended on a
machine that has never seen it, and the debt ledger has no unpaid critical items.
