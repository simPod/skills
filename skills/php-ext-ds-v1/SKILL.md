---
name: php-ext-ds-v1
description:
  Guides PHP developers in choosing and using ext-ds v1 data structures
  idiomatically instead of arrays or SPL collections. Use when working with PHP
  ext-ds v1, Ds\\Vector, Ds\\Map, Ds\\Set, Ds\\Stack, Ds\\Queue, Ds\\Deque,
  Ds\\PriorityQueue, collection performance, or migrations from PHP arrays to
  typed data structures.
license: MIT
compatibility: opencode
metadata:
  version: '0.1.0'
  author: simPod
---

# PHP ext-ds v1

## Gotchas

- `ext-ds` v1 is only partially API-compatible with v2; shared names like `Map`
  and `Set` exist, but sequence, priority, and key-equality APIs differ.
- Do not suggest v2-only types in v1 projects: `Ds\Seq`, `Ds\Heap`, `Ds\Key`.
- v1 uses separate sequence-like types; do not collapse them into `Seq`.
- Data structure constructors accept `iterable`; do not require callers to
  materialize arrays first.

## Type Map

| v1 type            | Use for                                                |
| ------------------ | ------------------------------------------------------ |
| `Ds\Vector`        | Ordered list with integer indexes                      |
| `Ds\Deque`         | Double-ended sequence                                  |
| `Ds\Map`           | Ordered dictionary with non-string/non-int key support |
| `Ds\Set`           | Unique values                                          |
| `Ds\Stack`         | Last-in-first-out processing                           |
| `Ds\Queue`         | First-in-first-out processing                          |
| `Ds\PriorityQueue` | Priority-based processing                              |
| `Ds\Hashable`      | Custom equality/hash behavior for object keys          |

## Checks

- Verify installed major with `composer.json`, lock files, CI image, or
  `php --ri ds` before editing code.
- Use `Ds\Queue` for FIFO and `Ds\Stack` for LIFO in v1.
- Use `Ds\PriorityQueue`, not `Ds\Heap`, in v1.
- Use `Ds\Hashable`, not `Ds\Key`, in v1.
- Keep `Ds\*` types inside code that can require `ext-ds`; convert to arrays at
  external boundaries.

## Minimal Examples

Import v1 collection types explicitly:

```php
use Ds\Deque;
use Ds\Map;
use Ds\PriorityQueue;
use Ds\Queue;
use Ds\Set;
use Ds\Stack;
use Ds\Vector;
```
