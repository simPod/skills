---
name: php
description:
  Guides agents in idiomatic PHP development, typing, tests, code style, and
  verification. Use when writing, reviewing, refactoring, or debugging PHP code,
  Composer packages, PHPUnit tests, PHPStan issues, PHP_CodeSniffer style,
  value-object boundaries, or PHP benchmark workflows.
license: MIT
compatibility: opencode
metadata:
  version: '0.1.0'
  author: simPod
---

# PHP

## Typing

- Prefer value objects over raw strings, integers, or UUID strings once data
  crosses an external boundary into application or domain code.
- Raw scalars are acceptable at true boundaries such as HTTP, GraphQL, CLI
  input, env/config, and persistence adapters that must serialize or deserialize
  values.
- Repositories, domain services, command/query handlers, and internal helper
  methods should prefer typed value objects in arguments and return types when a
  value object exists.
- Do not add stringly typed shortcuts alongside an existing value-object API
  unless there is a concrete compatibility need.
- Prefer the most concrete correct PHP return type. If a method returns a
  `Generator`, type it as `Generator` rather than broader `iterable`.
- If `composer.json` requires `ext-ds`, prefer `Ds` collections over arrays or
  SPL collections when modeling maps, sets, queues, stacks, or typed sequences.

## Tests

- Follow the repository's own PHPUnit command first; common forms are
  `vendor/bin/phpunit --group unit|integration|e2e` and
  `vendor/bin/phpunit --filter TestName`.
- Prefix PHPUnit data providers with `provide` and match the test name, for
  example `testMessage` -> `provideMessage`.
- Type PHPUnit data provider return values as `Generator` when yielding cases.
- Keep tests focused on observable behavior rather than implementation details.

## Style

- Follow the repository's configured PHP_CodeSniffer rules; common commands are
  `vendor/bin/phpcs` and `vendor/bin/phpcbf`.
- Use `~` as the PHP regex delimiter unless the surrounding project consistently
  uses another delimiter.
- Prefer named arguments for boolean flags in PHP calls.
- When using named arguments, keep them in the same order as declared in the
  function or method signature.
- Prefix PHPDoc array-shape tags with `@phpstan-` because array shapes are
  analyzer-only detail: use `@phpstan-param`, `@phpstan-return`, or
  `@phpstan-var` instead of `@param`, `@return`, or `@var`.
- Split long PHPDoc array shapes across multiple lines instead of keeping dense
  `@phpstan-var array{...}` annotations inline. For example:

  ```php
  /**
   * @phpstan-var array{
   *     level: string,
   *     isActive?: bool,
   * } $options
   */
  ```

- Do not add return types to PHP arrow functions.
- Prefer concrete imports and types over fully dynamic or stringly typed code.
- Prefer direct property access over trivial getters. Modern PHP features such
  as `public private(set)` are encouraged when they express the intended API.
- Call magic methods explicitly instead of relying on implicit object syntax:
  prefer `$x->__toString()` over `(string) $x` and `$x->__invoke($arg)` over
  `$x($arg)` when `$x` is an invokable object.
- On PHP 8.4+, do not wrap `new` expressions in parentheses just to call a
  method or access a member. Prefer `new Set($items)->map(...)` over
  `(new Set($items))->map(...)`.

## Verification

- Check the repository instructions before choosing host tooling, Docker, remote
  dev-server commands, or Composer scripts.
- Do not weaken PHPStan, PHPCS, PHPUnit, security, or build settings to make the
  current task pass.
- Run local PHP checks only when the user asks, the repository requires local
  validation, or a narrow command is the fastest way to diagnose or unblock the
  task.

## Benchmarks

- Use the repository's PHPBench setup when present.
- Common commands are `vendor/bin/phpbench run --report=default`,
  `vendor/bin/phpbench run --ref=baseline --report=table`, and
  `vendor/bin/phpbench run --tag=baseline --store`.
