# Engineering Constitution

A project-agnostic engineering standard — a constitution and eight reference modules — designed to be loaded into Claude Code at build time. The constitution is the baseline; the modules expand on it one concern at a time.

## The framework

| File | Owns | Load when |
|---|---|---|
| `00-engineering-constitution.md` | Non-negotiable principles + architecture spine | Always |
| `01-architecture-and-structure.md` | Layering, module boundaries, dependency direction, size limits, scope | Designing structure or placing new code |
| `02-configuration-and-environment.md` | One config source, env over hardcode, fail-fast validation, secret hygiene, portability | Touching config, paths, or credentials |
| `03-data-and-schema.md` | Schema-as-contract, versioned validation, forward migrations, idempotency | Defining or changing data shapes |
| `04-safety-and-correctness.md` | Boundary enforcement, fail-closed behavior, injection safety, typed error taxonomy | Writing logic that mutates state or accepts external input |
| `05-observability-telemetry-and-debt.md` | Structured logging, operational counters, audit trails, debt ledger | Adding logging or metrics, or deferring a fix |
| `06-testing-and-verification.md` | Deterministic tests, testable seams, PASS/PARTIAL/KILL criteria, reporting gaps | Writing tests or any batch/sync job |
| `07-build-process.md` | New-project checklist and hardening pass for existing projects | Starting a build or bringing one up to standard |
| `08-enforcement-and-ci.md` | Committed gates that run every other module's rules mechanically — conformance script, CI pipeline, local hooks, architecture contract, secret scan | Standing up a new project skeleton, wiring a rule into a gate, or verifying a build was actually proven |

Each file also ships as a Claude Code skill under `skills/<name>/SKILL.md` — same content, packaged for `@import`.

---

## Setup for Claude Code

### 1. Clone to a stable local path

```powershell
git clone https://github.com/Stegocode/EngineeringConstitution.git C:\path\to\EngineeringConstitution
```

Choose a path that won't move — `@import` lines in CLAUDE.md are absolute.

### 2. Wire the constitution into CLAUDE.md

Open `%USERPROFILE%\.claude\CLAUDE.md` (create it if it does not exist) and add:

```
@import C:\path\to\EngineeringConstitution\00-engineering-constitution.md
```

Replace `C:\path\to\EngineeringConstitution` with the path you cloned to. This is the baseline and loads for every session, every project, always.

### 3. Load individual skills the same way

Add `@import` lines to `%USERPROFILE%\.claude\CLAUDE.md` (global) or a project-level `.claude\CLAUDE.md`. Use the same absolute path prefix:

```
@import C:\path\to\EngineeringConstitution\skills\configuration-and-environment\SKILL.md
```

All available skills — add any or all:

```
@import C:\path\to\EngineeringConstitution\skills\architecture-and-structure\SKILL.md
@import C:\path\to\EngineeringConstitution\skills\build-process\SKILL.md
@import C:\path\to\EngineeringConstitution\skills\configuration-and-environment\SKILL.md
@import C:\path\to\EngineeringConstitution\skills\data-and-schema\SKILL.md
@import C:\path\to\EngineeringConstitution\skills\enforcement-and-ci\SKILL.md
@import C:\path\to\EngineeringConstitution\skills\observability-telemetry-and-debt\SKILL.md
@import C:\path\to\EngineeringConstitution\skills\safety-and-correctness\SKILL.md
@import C:\path\to\EngineeringConstitution\skills\testing-and-verification\SKILL.md
```

Load the constitution always. Load skills on demand — the one or two relevant to the work in progress, not all eight at once.

---

## The one rule that governs this rulebook

No concern appears in two modules. Each module owns exactly one concern. If a rule could live in two places, it goes in one and the other points to it. If this set ever sprawls, the fix is the same as for any god-file: split or consolidate.
