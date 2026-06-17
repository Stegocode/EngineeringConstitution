---
name: configuration-and-environment
description: How to handle configuration, paths, secrets, and environment so a project is portable and reproducible — one config source, environment over hardcode, fail-fast validation at boot, secret hygiene, and the clone-and-run test. Use this whenever touching file paths, connection strings, credentials, environment variables, or anything that determines whether someone else can install and run the project.
---

# Configuration & Environment

Expands Constitution principles 7 and 10.

## One config source

Exactly one module reads the environment. Everything else imports values from it.
No other file calls `os.environ` / `process.env` directly. A path or setting is
defined **once** and imported wherever needed — never typed into multiple files.
(The single worst real-world smell: the same absolute path hardcoded independently
in six files. One change, six edits, five of them forgotten.)

## Environment over hardcode

- No machine-specific paths in logic. No literal that begins with `/Users/`,
  `C:\Users\`, `/home/`, or any absolute root inside a source file.
- Credentials, connection strings, hostnames, ports, and tunables come from the
  environment, with sensible defaults only for non-secret values.
- Treat backing services (database, cache, model server, queue) as **attached
  resources** you point at by config — not as fixed local assumptions.

## Fail-fast validation at boot

The first line of every entry point validates config:

- Check all required variables in **one pass** and report every missing one
  together — not one error per restart.
- Include contradiction checks (e.g., a low threshold that exceeds a high one).
- If anything required is missing or inconsistent, stop with an actionable message.
  Never boot half-configured.

## Secret hygiene

- A committed `.env.example` (or equivalent) lists every variable with a one-line
  description. The real `.env` is gitignored and never committed.
- Load secrets only in entry-point scripts, not in library modules.
- Never log a secret. Scrub or redact before any log call; sanitize stack traces
  that might carry a credential.

## The portability test

The whole point of this module is one sentence: **a stranger can clone the repo,
set the documented environment, and run it on a machine that has never seen it.**
If that is not true, config is not done.
