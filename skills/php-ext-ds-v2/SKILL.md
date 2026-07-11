---
name: php-ext-ds-v2
description:
  Guides PHP developers in choosing and using ext-ds v2 data structures
  idiomatically instead of arrays or SPL collections. Use when working with PHP
  ext-ds v2, Ds\\Seq, Ds\\Map, Ds\\Set, Ds\\Heap, Ds\\Pair, Ds\\Key, collection
  performance, or migrations from PHP arrays to typed data structures.
license: MIT
compatibility: opencode
metadata:
  version: '0.2.0'
  author: simPod
---

# PHP ext-ds v2

## Gotchas

- `ext-ds` v2 is only partially API-compatible with v1; shared names like `Map`
  and `Set` exist, but sequence, priority, and key-equality APIs differ.
- php.net docs may still describe v1 APIs; prefer upstream v2 docs or the
  installed extension.
- v2 has copy-on-write semantics for safe iteration during mutation.
- Do not suggest removed v1 types in v2 projects: `Ds\Vector`, `Ds\Deque`,
  `Ds\Stack`, `Ds\Queue`, `Ds\PriorityQueue`, `Ds\Hashable`.
- Data structure constructors accept `iterable`; do not require callers to
  materialize arrays first.

## Type Map

| v2 type   | Replaces / use for                                      |
| --------- | ------------------------------------------------------- |
| `Ds\Seq`  | `Vector`, `Deque`, `Stack`, `Queue`; ordered sequences  |
| `Ds\Map`  | Ordered dictionaries with arbitrary keys                |
| `Ds\Set`  | Ordered unique values                                   |
| `Ds\Heap` | `PriorityQueue`; priority processing, optional comparer |
| `Ds\Pair` | Readonly key-value pair                                 |
| `Ds\Key`  | `Hashable`; custom key equality                         |

## Checks

- Verify installed major with `composer.json`, lock files, CI image, or
  `php --ri ds` before editing code.
- Use `Ds\Seq` for list, stack, queue, and deque use cases in v2.
- Use `Ds\Heap` instead of `Ds\PriorityQueue` in v2.
- Use `Ds\Key` instead of `Ds\Hashable` in v2.
- Prefer native collection operations such as `map()`, `filter()`, `reduce()`,
  and `join()` over converting to arrays for PHP array functions.
- For value-only transforms, prefer constructing the collection from the
  iterable and calling native `map()` directly, for example
  `new Seq($items)->map(...)`. Do not add helper wrappers just to expose unused
  keys; this applies to `Seq`, `Set`, `Map`, and other collections.
- Keep `Ds\*` types inside code that can require `ext-ds`; convert to arrays at
  external boundaries.

## Minimal Examples

Import v2 sequence/map/set/heap types explicitly:

```php
use Ds\Heap;
use Ds\Map;
use Ds\Seq;
use Ds\Set;
```
