# simpod/skills

AI agent skills for software development.

This repository contains these skills:

- `opencode-session-db`: OpenCode V2 session lookup, rename, and native
  archive/restore through the API, with guidance for legacy transcript archives.
- `opencode-session-id`: exact current-chat ID lookup through the V2 API.
- `onepassword-cli`: safe 1Password CLI secret injection.
- `php`: general PHP typing, testing, style, and verification guidance.
- `php-doctrine-orm`: Doctrine ORM mapping, queries, persistence, and JSONB
  guidance.
- `php-ext-ds-v1`: `ext-ds:^1` API notes.
- `php-ext-ds-v2`: `ext-ds:^2` API notes.
- `renovate-ship`: list or track open Renovate PRs and MRs in OpenCode,
  summarize release notes, fix CI, or merge a selected GitHub or GitLab update
  on request.
- `test-audit`: test authoring and audit guidance for any repository.
- `testing`: shared policy for test selection, review, analyzer results,
  validation, and CI.
- `typescript-testing`: type-safe TypeScript tests with native assertions.

## Install

With `skills`:

```sh
npx skills add simPod/skills --skill opencode-session-db
npx skills add simPod/skills --skill opencode-session-id
npx skills add simPod/skills --skill onepassword-cli
npx skills add simPod/skills --skill php
npx skills add simPod/skills --skill php-doctrine-orm
npx skills add simPod/skills --skill php-ext-ds-v1
npx skills add simPod/skills --skill php-ext-ds-v2
npx skills add simPod/skills --skill renovate-ship
npx skills add simPod/skills --skill test-audit
npx skills add simPod/skills --skill testing
npx skills add simPod/skills --skill typescript-testing
```

With `dotagents`:

```sh
npx @sentry/dotagents init
npx @sentry/dotagents add simPod/skills --name opencode-session-db
npx @sentry/dotagents add simPod/skills --name opencode-session-id
npx @sentry/dotagents add simPod/skills --name onepassword-cli
npx @sentry/dotagents add simPod/skills --name php
npx @sentry/dotagents add simPod/skills --name php-doctrine-orm
npx @sentry/dotagents add simPod/skills --name php-ext-ds-v1
npx @sentry/dotagents add simPod/skills --name php-ext-ds-v2
npx @sentry/dotagents add simPod/skills --name renovate-ship
npx @sentry/dotagents add simPod/skills --name test-audit
npx @sentry/dotagents add simPod/skills --name testing
npx @sentry/dotagents add simPod/skills --name typescript-testing
npx @sentry/dotagents install
```

### OpenCode `/renovate-ship` command

Install the skill, then copy its bundled
[`commands/renovate-ship.md`](skills/renovate-ship/commands/renovate-ship.md)
into `~/.config/opencode/commands/` (or `.opencode/commands/` for one project).
Installing the skill alone does not register the command.

Run `/renovate-ship` in a GitHub or GitLab repository with authenticated `gh` or
`glab`, respectively, and the OpenCode PR tracker installed. With no arguments,
it only discovers and tracks open Renovate requests. Use `see all` (or
`see all mrs`) to list them without tracking, `see symfony` for release notes,
`fix symfony` to fix CI without merging, or `merge symfony` to wait for CI, fix
in-scope failures, and merge. You can also run
`/renovate-ship see symfony/contracts`.
