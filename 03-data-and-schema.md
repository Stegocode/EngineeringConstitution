---
name: data-and-schema
description: How to handle structured data so it stays correct over time — schema as a versioned contract, validation on load and save, forward migrations, idempotent operations (setup and business transactions), and validator parity across language boundaries. Use this whenever defining or changing a data shape, writing persistence, handling records or transactions, or moving data between a backend and frontend.
---

# Data & Schema

Expands Constitution principles 8 and 9.

## Schema is a contract

- Define each entity **once**, in one canonical place. No module is allowed to
  invent its own slightly-different version of a record's shape. (Parallel modules
  that each re-define "the same record a little differently" is the data-layer form
  of duplication, and it diverges silently.)
- Declare a `SCHEMA_VERSION`. Validate structured data on **load** and on **save** —
  not just at the UI.
- A validator is a pure function: data in, list-of-problems out. It lives in the
  core, not the I/O layer.

## Migrations

- Schema changes are **forward migrations**, versioned and ordered — never a hand
  edit of stored data.
- Set up the migration path from version 1, even when only v1 exists, so the
  pattern is there before you need it.
- Migrations validate, never mutate their input silently, and are testable in
  isolation.

## Idempotency

The same operation applied twice must not double its effect. This holds at two
levels:

- **Setup / migrations:** safe to run twice. `CREATE TABLE IF NOT EXISTS`,
  upsert-not-blind-insert, "create raises or no-ops if it already exists."
- **Business transactions:** receiving the same shipment twice does not double the
  inventory; a retried request does not create a duplicate record. Key
  transactions on a stable identifier and check-before-apply, or make the write
  naturally idempotent. For a system of record, data integrity *is* the product.

## Validator parity across boundaries

If one language writes the data and another reads it (e.g., a Python service and a
JavaScript client), both sides validate against the **same rules**. Duplicate the
validation logic deliberately on each side of the boundary rather than trusting a
single network hop — and keep the two copies provably in sync (shared test
fixtures, identical constants). This is the one sanctioned place to "repeat
yourself," because the alternative is trusting unvalidated data across a wire.
