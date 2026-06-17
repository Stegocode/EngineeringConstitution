---
name: enforcement-and-ci
description: How conformance stops being self-reported and becomes mechanical — the committed gate (a conformance script, a CI pipeline, a local hook, an architecture contract, a secret scan) that runs every other module's greppable rules where the builder cannot skip them or self-attest past them. Load this when standing up a new project's skeleton, when adding a rule that needs to bind, or when judging whether a "passing" build was actually proven. This is the answer to "who checks the checker"; without it, every other module runs on trust.
---

# Enforcement & CI

Expands Constitution principles 3 and 13; operationalizes principle 15.

Every other module produces rules. This one makes them run without a human in the
loop. It is the cross-cutting enforcer — the single committed thing that executes
the architecture, config, naming, safety, and schema checks the other modules
define. A rule that lives only as prose is a wish; wired into the gate, it becomes
a fact about every commit.

## Self-report is not enforcement

A builder — human or agent — saying "all checks pass" is a claim, not a proof. The
proof is the gate going green in a place the builder does not control. The two are
not interchangeable: the most common failure is a confident "zero hits" on a grep
that was never run over the file that mattered. Wire each rule so it runs
mechanically, on every change, where it cannot be skipped, reworded around, or
attested away.

## The enforcer is a committed artifact

Not a checklist someone runs — a set of files in the repo:

- **A conformance script.** Pure, dependency-light, runs identically on a
  developer machine and the CI runner. It scans *tracked* files — not just source,
  because the leak is always in the file nobody thought to scan — prints every
  violation as `file:line`, and exits non-zero on any. It owns the greppable rules:
  size caps, no hardcoded paths, names-only-in-config, env-reads-in-one-place, no
  string-built SQL, no `except: pass`.
- **A CI pipeline.** Runs the conformance script, the type check, the architecture
  contract, the tests, and a secret scan — on every push and every pull request,
  all blocking, on a pinned runtime.
- **A local hook.** The same gates fire before a push, so a failure is caught a
  commit earlier.
- **An architecture contract.** The inward-dependency rule (Module 01) enforced by
  a real tool, not a grep: the core may not import services or adapters; services
  may not import adapters. Static, and it fails the build on violation.
- **A secret scan.** Mandatory on any public repo, where a committed credential is
  harvested in minutes — platform push-protection plus a scanner in CI.

## Map every gate to the rule it enforces

The enforcer is complete only when every mechanical rule in the framework has a
line in it. The canonical mapping:

- **Spine, inward dependencies (Constitution 1, Module 01)** → architecture
  contract: core ⇏ services ⇏ adapters.
- **No god files (Constitution 6)** → conformance script: file ≤ 400 lines,
  function ≤ 60.
- **Environment over hardcode (Constitution 7, Module 02)** → conformance:
  environment reads in the config module only; no `/Users/`, `C:\`, or other
  absolute path in any tracked file.
- **Domain vocabulary (Constitution 10)** → conformance: no client, company, or
  system name outside configuration — scanned across *all* tracked files,
  including docs, journals, and commit-adjacent scratch, not only code.
- **Fail closed (Constitution 4, Module 04)** → conformance + lint: no
  `except: pass`, no string-built SQL.
- **Schema is a contract (Constitution 8, Module 03)** → tests assert
  validate-on-load and validate-on-save; the gate runs them.
- **Pure logic is tested (Constitution 2, Module 06)** → the suite runs against
  fakes in CI; no real infrastructure required.
- **No secrets in the repo (Module 07 checklist)** → secret scan + push
  protection; `.env` untracked, `.env.example` committed.

If a rule from another module has no row here, it is not enforced — it is
documentation. Add the row in the same change that adds the rule.

## Make the code conform to the gate — never the reverse

Gates encode the standard; the code earns the green. You fix the code to satisfy a
gate — you never weaken the gate to pass. No inline suppression to silence a real
failure, no rewording a comment to dodge a string match, no editing the pipeline
to skip the step that's red. Choosing which lint rules to *enable* is
configuration; silencing an enabled rule is evasion. If a gate cannot pass
honestly, the answer is to fix the code or stop and say so — a gamed gate is worse
than no gate, because it reports a safety that isn't there.

## Blocking, or it doesn't count

An advisory gate is decoration. Make the CI run a required status check so a red
build actually stops the merge. Until the gate can block, conformance is still a
request.

## One rule, one gate — landed together

When a module or a ticket introduces a constraint, its gate lands in the same
commit. A rule without a gate is a guess wearing a docstring. And the gate reports
its own coverage: state which rules are mechanically enforced and which are still
manual, so unguarded rules are known, not assumed (principle 13, turned on the
checker itself).

## Wire it in at T-01

The enforcer ships with the skeleton, before any feature — this is what "put the
gate in from the first commit" (Module 07) means once it's in files. A project
whose first green build proves the spine, the names, the size caps, and the
absence of secrets is clean by construction. One that adds the gate later is
cleaning up — and will find the gate was needed most for the commits that predate
it.

The enforcer exists when:

- [ ] One conformance script scans all tracked files and fails on any violation
- [ ] CI runs it, the type check, the architecture contract, tests, and a secret
      scan — blocking, on push and PR
- [ ] The same gates run locally before push
- [ ] The CI check is required; a red build cannot merge
- [ ] Every greppable rule in the other modules maps to a gate here
- [ ] The build reports which rules are gated and which remain manual
