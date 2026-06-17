---
name: safety-and-correctness
description: How to make code safe to run and trust — validate at every boundary, fail closed on uncertainty, crash-safe atomic writes, injection-safe queries, typed and actionable errors, and no silent failures. Use this whenever writing code that mutates state, persists data, accepts input from outside the function, queries a database, or handles an error.
---

# Safety & Correctness

Expands Constitution principles 3 and 4.

## Hard enforcement at every boundary

Validate where data crosses into your code: function arguments, deserialized
payloads, database reads, API responses, file contents. Never assume input is
valid because "it came from our own code." Assertions and validators are not
optimism — they are the mechanism that keeps generated and fast-written code from
doing quiet damage. Trust the contract; verify the data.

## Fail closed, deny by default

When the system cannot proceed correctly — missing config, ambiguous validation,
a failed external call — it stops and reports. It does not guess.

- No silent fallbacks. No `except: pass`. No `except Exception: return None`.
- A code path that cannot handle a case raises or returns an explicit error
  sentinel — it never continues as if nothing happened.
- A correct refusal beats a confident wrong action, every time.

## Atomic, crash-safe writes

Any write to state that must survive a crash uses **write-to-temp, then atomic
rename** (`os.replace` or equivalent): write the new content to a temp file in the
same directory, set permissions, then rename over the target. On failure, delete
the temp file and raise. A crash mid-write then leaves either the old file or the
new one — never a half-written one. Do this in **one** shared helper that
everything imports (Principle 5); never hand-roll it per module.

## Injection-safe data access

All database access uses parameterized queries. Never build SQL (or any query
language) with string formatting or interpolation of values. The same caution
applies to any place untrusted input reaches an interpreter: shell, paths,
templates, deserializers. Reject path traversal and escape sequences at the
boundary, before any filesystem or system call.

## Typed, actionable errors

- Define an error taxonomy once: validation error vs. not-found vs. conflict vs.
  infrastructure failure. Handle each by its type, not by ad-hoc per-module
  guessing.
- Every error message names the rule, the violation, **and** the fix. "Target
  file has uncommitted changes — commit or stash before running" beats "error."
  Good error messages are also your cheapest observability.
