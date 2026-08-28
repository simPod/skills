---
name: typescript-testing
description:
  Guides agents in type-safe TypeScript testing with native assertions and
  minimal matcher DSLs. Use when writing, reviewing, refactoring, or debugging
  TypeScript tests, choosing node:test, replacing expect matchers, preserving
  control-flow narrowing, or removing any from test assertions.
license: MIT
compatibility: opencode
metadata:
  version: '0.1.0'
  author: simPod
---

# TypeScript Testing

Write tests as TypeScript programs. Prefer native expressions and assertion
functions that participate in control-flow analysis over matcher DSLs.

## Workflow

1. Read repository instructions, package scripts, test configuration, runtime
   targets, and nearby tests. The test command and runtime constraints are known
   when this step is complete.
2. Keep the configured runner unless the user requests a migration. For new
   Node-only unit tests with no framework requirement, prefer `node:test`.
3. Use Node's strict assertion mode in Node-compatible tests. For browser tests,
   use an assertion function with equivalent TypeScript assertion signatures.
4. Write each test around observable behavior, with setup, one behavior under
   test, and focused assertions. Every relevant branch has an explicit check.
5. Run the narrow test command and the repository's TypeScript check. The work
   is complete when both pass without casts that hide assertion errors.

## Quick Start

```ts
import { strict as assert } from 'node:assert';
import { test } from 'node:test';

test('returns the active user', async () => {
  const user = await findActiveUser();

  assert(user !== undefined);
  assert.equal(user.status, 'active');
  assert.deepEqual(user.roles, ['admin']);
});
```

`assert(condition)` has the signature `asserts condition`, so TypeScript knows
that `user` is defined after the assertion.

## Type Safety

- Accept uncertain values as `unknown`, then narrow them with executable checks.
- Use `assert(value !== undefined)`, `assert(value instanceof Error)`,
  `assert(typeof value === "string")`, and property checks as program logic.
- Use `assert.equal`, `assert.deepEqual`, `assert.match`, `assert.throws`, and
  `assert.rejects` when their diagnostics are useful.
- Keep the expected value statically checked. Node equality assertions infer
  from `expected` while accepting `actual` as `unknown`; they can still compile
  for unrelated types. Use `satisfies`, an explicit domain type, or the typed
  helper in [ASSERTIONS.md](ASSERTIONS.md) when this relationship matters.
- Use `assert.fail()` for unreachable test branches. It returns `never`.
- Return or await every promise under test. Await `assert.rejects`.
- In error validators, narrow the caught `unknown` value before reading it and
  return `true` only after all checks pass.
- Prefer explicit property and element assertions over asymmetric placeholders.
  They retain the static types of both the path and expected value.
- Add a specialized assertion helper only when it gives materially better
  diagnostics or removes repeated, type-safe checks.
- Keep snapshots small and intentional. Prefer explicit assertions for stable
  contracts and important business behavior.
- Preserve framework assertions that provide retrying, polling, soft failures,
  locators, DOM semantics, mock inspection, or snapshots. Convert only immediate
  value assertions after the framework has resolved the value.

## Runner Choice

- Preserve Vitest, Jest, or another runner when the suite needs its browser
  environment, module mocking, fake timers, plugins, or existing integration.
- Native assertions can run inside an existing test runner when the target can
  load Node's assertion module; runner choice and assertion style are separate.
- Use `node:test` for Node-focused tests when it satisfies the repository's
  coverage, mocking, snapshot, watch, and reporting requirements.
- Follow the repository's supported Node and TypeScript execution path. Do not
  assume that `node --test` can execute every TypeScript syntax or module setup.

## Reference

Load [ASSERTIONS.md](ASSERTIONS.md) when converting matcher assertions, checking
unknown values, testing errors, or enforcing compatible equality operands.
