---
name: commit
description: Write commit messages following project conventions. Use when committing changes, creating commits, or asking about commit message format.
---

# Commit Message Skill

Write commit messages following the project's Conventional Commits format with commitlint validation.

## Format

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

## Rules

1. **Type** (required) - one of:
   - `feat` - new feature
   - `fix` - bug fix
   - `refactor` - code refactoring (no feature/fix)
   - `chore` - maintenance tasks (deps, configs)
   - `perf` - performance improvement
   - `ci` - CI/CD changes
   - `docs` - documentation only
   - `test` - adding/fixing tests
   - `style` - formatting (no code change)
   - `build` - build system changes
   - `revert` - reverting a commit

2. **Scope** (optional) - kebab-case, describes the area affected:
   - `deps` - dependency updates
   - `symfony` - Symfony configuration
   - `gql` - GraphQL related
   - `auth` - authentication
   - `sentry` - Sentry/monitoring
   - `matrix` - matrix feature
   - `email-report` - email reports
   - `servers` / `server` - servers feature
   - `timeseries` - timeseries/charts
   - `dashboard` - dashboard feature
   - `cross-connects` - cross-connects feature
   - `ipmi-link` / `ipmi-linker` - IPMI linking
   - `device` / `device-polling` - device related
   - `dbal` / `orm` / `pg` - database layer
   - Or any other relevant module/feature name
   - Multiple scopes: `fix(dbal,orm): description`

3. **Description** - lowercase, imperative mood, no period at end

4. **Header max length** - 160 characters

5. **Body** - no line length limit (rule disabled)

6. **Footer** - no line length limit (rule disabled)

## Choose the type by intent

- Use `feat` for requested new or changed product behavior, appearance, or integrations, even when the change
  replaces existing code or changes only a URL, label, icon, or CSS rule.
- Use `fix` when correcting a demonstrated defect against an established requirement or restoring intended
  behavior. Identify that defect from the request, specification, or observed failure; do not invent one.
- Replacing something old does not by itself make the change a fix. Diff size and implementation technique
  do not determine the type. Do not default to `fix` merely because existing behavior changes.
- Use `refactor` for internal restructuring with no product behavior change, and `style` only for non-functional
  source formatting. Follow the user's explicit classification when supplied.
- Prefer a domain scope for product changes; reserve technical scopes for purely mechanical changes in that layer.
- Keep the commit type and PR/MR title aligned. Describe feature motivation without inventing a root cause.

Examples:

- Requested switch to a new ticket system: `feat(tickets): use new ticket URLs`.
- Generated ticket URLs violate the documented URL format: `fix(tickets): correct ticket URL format`.
- Requested new button appearance: `feat(navigation): update button appearance`.
- Rename an internal link helper without changing its output: `refactor(tickets): rename ticket link helper`.

## Examples

```
feat(auth): add OAuth2 support for external providers
```

```
fix(matrix): dedupe tags
```

```
refactor(symfony): migrate monolog to php configs
```

```
chore(deps): update dependency phpstan/phpstan to ^2.1.36
```

```
perf(sentry): increase sampling rate vol.2
```

```
ci: improve patroni perf in docker compose
```

```
refactor: gql queries monClientSensorState and monClientSensorStates
```

```
test: cover link and unlink server with port
```

```
fix(dbal,orm): dont rollback possibly failed transaction
```

```
perf(timeseries): debounce maxDataPoints updates to prevent query cancellation on resize
```

## Common Patterns from History

- Dependency updates: `chore(deps): update dependency <package> to <version>`
- Config migrations: `refactor(symfony): migrate <thing> to php configs`
- GraphQL changes: `refactor(gql): <description>` or `refactor: gql <queries/mutations>`
- Feature additions: `feat(<scope>): <description>`
- Bug fixes: `fix(<scope>): <description>`
- Performance: `perf(<scope>): <description>`
- Tests: `test: <description>` or `test(<scope>): <description>`
- Reverts: Start message with `Revert "original commit message"`
- Multi-scope fixes: `fix(scope1,scope2): <description>`

## Validation

The projects use `@commitlint/config-conventional` with:
- `scope-case`: must be kebab-case
- `header-max-length`: 160 characters
- `body-max-line-length`: disabled (no limit)
- `footer-max-line-length`: disabled (no limit)

## Important

Do NOT add any of the following to commit messages:
- "Generated with [Claude Code]" or similar attribution
- "Co-Authored-By" lines
- Any AI assistant attribution
