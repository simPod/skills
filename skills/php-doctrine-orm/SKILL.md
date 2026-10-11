---
name: php-doctrine-orm
description:
  Guides PHP developers in Doctrine ORM entity mapping, repositories, queries,
  persistence, and migrations. Use when working with Doctrine entities, ORM
  attributes, EntityManager, UnitOfWork, DQL, QueryBuilder, associations, or
  PostgreSQL JSONB mappings.
license: MIT
compatibility: opencode
metadata:
  version: '0.1.0'
  author: simPod
---

# PHP Doctrine ORM

## Start Here

- Read repository instructions and load the `php` skill for general PHP
  guidance.
- Check `composer.json` and `composer.lock` for the installed ORM, DBAL, and
  migrations versions. Confirm the database platform before choosing mappings.
- Follow existing mapping, repository, custom type, and transaction patterns.

## Entity Mapping

- Keep entities focused on domain state and behavior. Keep queries and
  EntityManager operations outside entities.
- Align PHP types, ORM types, nullability, and database constraints. Use
  existing value objects and custom DBAL types at persistence boundaries.
- Initialize to-many associations with Doctrine collections and declare their
  element types. Do not replace persistent associations with arrays or `Ds`
  types.
- Update the owning side of associations; synchronize both sides of
  bidirectional associations when the domain operation requires it.
- Choose cascade operations and `orphanRemoval` explicitly. Do not use
  `cascade: ['all']` as a default; removal must match the ownership lifecycle.

## JSONB, Not JSON

- Use `jsonb`, not `json`, for JSON fields in PostgreSQL entities. On DBAL 4.3+
  use `Types::JSONB` in the entity mapping:

  ```php
  use Doctrine\DBAL\Types\Types;
  use Doctrine\ORM\Mapping as ORM;

  #[ORM\Column(type: Types::JSONB)]
  private array $metadata = [];
  ```

- `Types::JSONB` is built in from DBAL 4.3. Do not assume it exists in DBAL 3.x
  or 4.0–4.2. If the project cannot upgrade, its legacy mapping must explicitly
  select PostgreSQL JSONB with `type: Types::JSON, options: ['jsonb' => true]`.
  This option is deprecated from DBAL 4.3; do not use it in new mappings there.
- Never fall back to plain `Types::JSON`. Check migrations use `JSONB` and
  schema comparison does not produce repeat diffs.
- JSONB does not preserve object key order, whitespace, or duplicate keys. Do
  not use it when the exact input text must survive a round trip.
- JSON hydration does not restore PHP object classes. Validate document shape at
  the application boundary; use relational columns for data that needs foreign
  keys or independent relational constraints.

## Queries and Persistence

- Bind query parameters. Never concatenate untrusted values into DQL or SQL;
  allowlist dynamic field names and sort directions.
- Select the data the operation needs. Avoid N+1 queries with deliberate fetch
  joins or projections, not global eager loading.
- Use Doctrine's paginator or a deliberate two-step query for pagination with
  to-many fetch joins; a row limit alone does not reliably limit root entities.
- `persist()` registers an entity; `flush()` writes pending UnitOfWork changes,
  not only the entity most recently passed to `persist()`.
- Keep flush and transaction boundaries at the application operation unless the
  project defines another contract. Use the installed version's transaction API.
- After ORM failure or rollback, check the EntityManager's state. Do not reuse a
  closed manager or assume PHP entity state was rolled back.
- Bulk DQL updates and deletes bypass normal entity change tracking and
  lifecycle events. Plan how affected managed entities will be refreshed or
  detached.
- For large batches, use bounded iteration and flush/clear cycles. `clear()`
  detaches entities; do not keep using them as managed references.

## Migrations and Verification

- Review generated migrations for unintended drops, type changes, defaults, and
  association constraints. Do not run migrations, schema updates, or other
  database writes without explicit permission.
- Follow repository CI and verification rules. Use the real database platform
  for integration coverage of JSONB, custom types, constraints, and query
  semantics; SQLite is not a substitute for PostgreSQL-specific behavior.
- Test public persistence behavior and meaningful round trips. Do not add tests
  that merely repeat mapping declarations or assert private ORM implementation.

## References

- [Doctrine ORM documentation](https://www.doctrine-project.org/projects/doctrine-orm/en/current/index.html)
- [Doctrine DBAL JSONB type](https://www.doctrine-project.org/projects/doctrine-dbal/en/4.3/reference/types.html#jsonb-1)
- [DBAL 4.3 upgrade notes](https://github.com/doctrine/dbal/blob/4.3.0/UPGRADE.md)
