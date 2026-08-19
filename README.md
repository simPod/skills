# simpod/skills

AI agent skills for idiomatic PHP development.

This repository contains these skills:

- `opencode-session-db`: OpenCode session lookup, rename, and archive-state management.
- `onepassword-cli`: safe 1Password CLI secret injection.
- `php`: general PHP typing, testing, style, and verification guidance.
- `php-ext-ds-v1`: `ext-ds:^1` API notes.
- `php-ext-ds-v2`: `ext-ds:^2` API notes.

## Install

With `skills`:

```sh
npx skills add simPod/skills --skill opencode-session-db
npx skills add simPod/skills --skill onepassword-cli
npx skills add simPod/skills --skill php
npx skills add simPod/skills --skill php-ext-ds-v1
npx skills add simPod/skills --skill php-ext-ds-v2
```

With `dotagents`:

```sh
npx @sentry/dotagents init
npx @sentry/dotagents add simPod/skills --name opencode-session-db
npx @sentry/dotagents add simPod/skills --name onepassword-cli
npx @sentry/dotagents add simPod/skills --name php
npx @sentry/dotagents add simPod/skills --name php-ext-ds-v1
npx @sentry/dotagents add simPod/skills --name php-ext-ds-v2
npx @sentry/dotagents install
```
