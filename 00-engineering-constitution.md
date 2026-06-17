---
name: engineering-constitution
description: The non-negotiable engineering principles for building any software project — the architecture spine, boundary enforcement, fail-closed behavior, no duplication, no god-files, config externalization, schema contracts, idempotency, observability, debt tracking, and pre-registered success criteria. Load this for EVERY project and EVERY build task, always, even when the user has not asked for "standards" — it is the baseline every other decision is checked against.
---

# Engineering Constitution

These fourteen rules apply to every file in every project, every time. They are
the baseline. Reference modules expand on them; nothing overrides them.

1. **One architecture — the spine.** Layered, with dependencies pointing inward:
   a pure **core** (domain logic, no I/O) → **application/services** → **adapters**
   at the edge (database, network, files, UI). The core never imports outward.
   Most other rules fall out of this one.

2. **Pure logic is the unit of correctness.** Anything expressible as
   `input → output` with no side effects lives in its own module and gets a test.
   I/O and UI are not unit-tested; logic and validation are.

3. **Hard enforcement over trust.** Validate at every boundary — arguments,
   deserialized data, DB reads, external responses. Never assume input is valid
   because it came from internal code. Validators are the mechanism that makes
   AI-generated code safe to run.

4. **Fail closed, deny by default.** On any uncertainty — config missing,
   validation ambiguous, external call failed — stop and report. Never proceed on
   a guess. No silent fallbacks, no `except: pass`. A confident wrong action is
   worse than an honest refusal.

5. **Don't repeat yourself across modules.** A second copy of a function is a bug
   waiting to diverge. One shared utility, imported everywhere. Search for an
   existing helper before writing a new one.

6. **No god files.** Cap files at ~400 lines and functions at ~60. Every new piece
   of code has an obvious home in a named module. If it has none, that is the
   signal to create a module — not to grow the nearest large file.

7. **Environment over hardcode.** Paths, credentials, connection strings, and
   tunables come from config/environment, read in exactly one place. Nothing
   machine-specific lives in logic. The test: a stranger can clone it and run it.

8. **Schema is a contract.** Declare a version, validate on load and on save,
   migrate forward automatically. Define each entity once; no module is allowed
   to redefine a record's shape.

9. **Idempotent operations.** Setup, migrations, and business transactions are
   safe to run twice. The same input applied twice must not double the effect.

10. **Domain vocabulary, never proprietary names.** General code reads as if it
    applies to any project in the domain. Client, company, and system-specific
    names live only in configuration values — never in logic, module names, or
    identifiers.

11. **Observe and measure.** Structured logging at every boundary, so a failure is
    legible without a debugger. Lightweight operational metrics, so behavior is
    measurable over time. (Diagnose *and* measure — see module 05.)

12. **Track every deferral.** No silent debt. Every deferred fix is a dated, marked
    entry in a debt ledger. A fix may be fast, but it must still obey the cheap
    rules (one shared utility, config not hardcode) and log its debt on the way in.

13. **Declare success before the run; report what was not measured.** Any batch,
    sync, or validation pass declares PASS / PARTIAL / KILL thresholds before it
    runs. Its result states what was *not* checked, not only what was found.
    Unreported scope is not implied coverage.

14. **Declare your boundaries.** Write the assumptions and out-of-scope items down
    (single-writer, no auth yet, single machine) so they are known, not discovered
    the day reality changes.
