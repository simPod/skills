# Type-Safe Assertions

## Common Recipes

Check the original matcher's semantics before conversion. These recipes apply to
immediate JavaScript values, not framework polling, locators, DOM objects, soft
assertions, mocks, or snapshots.

| Matcher style                               | Program style                       | Condition                                                                                             |
| ------------------------------------------- | ----------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `expect(value).toBeDefined()`               | `assert(value !== undefined)`       | Direct replacement                                                                                    |
| `expect(value).toBe(expected)`              | `assert.equal(value, expected)`     | Direct replacement in strict mode                                                                     |
| `expect(value).toStrictEqual(expected)`     | `assert.deepEqual(value, expected)` | Closest strict comparison; verify `WeakMap`, `Promise`, `Error`, and other library-specific semantics |
| `expect(value).toEqual(expected)`           | No exact replacement                | Use `deepEqual` only when strict prototypes, sparse arrays, and `undefined` properties are intended   |
| `expect(value).toBeInstanceOf(Error)`       | `assert(value instanceof Error)`    | Direct replacement                                                                                    |
| `expect(value).toMatch(pattern)`            | `assert.match(value, pattern)`      | `pattern` must be a `RegExp`; use `includes` for a string                                             |
| `expect(items).toContain(item)`             | `assert(items.includes(item))`      | Arrays and strings only; `includes` uses SameValueZero, including for `NaN`                           |
| `expect(items).toHaveLength(2)`             | `assert.equal(items.length, 2)`     | Values with a numeric `length`                                                                        |
| `expect(value).toHaveProperty("id")`        | `assert("id" in value)`             | Direct properties only, after narrowing to an object                                                  |
| `expect(fn).toThrow(Error)`                 | `assert.throws(fn, Error)`          | Synchronous errors only                                                                               |
| `await expect(fn()).rejects.toThrow(Error)` | `await assert.rejects(fn, Error)`   | Promise rejections only                                                                               |

Use a native condition when it narrows a value or TypeScript can check both
operands directly:

```ts
assert(result !== undefined);
assert(result.kind === 'success');
assert(result.items.includes(expectedItem));
```

Use an equality method when its failure diff gives useful diagnostics.

## Unknown Objects

Replace asymmetric matchers with sequential checks that narrow the value:

```ts
const value: unknown = await readPayload();

assert(typeof value === 'object' && value !== null);
assert('id' in value);
assert(typeof value.id === 'string');
assert('name' in value);
assert(typeof value.name === 'string');
```

This keeps every accessed property in TypeScript's control-flow analysis and
does not introduce `any` placeholders.

## Typed Expected Values

`assert.deepEqual(actual, expected)` narrows `actual` to the inferred type of
`expected`, but it does not prove before runtime that both operands had related
types. Check an object literal against the domain type first:

```ts
const expected = {
  id: 'user-1',
  status: 'active',
} satisfies User;

assert.deepEqual(actual, expected);
```

For repeated comparisons in TypeScript 5.4 or newer, infer from `actual` and
block inference from `expected` with TypeScript's `NoInfer` utility type:

```ts
import { strict as assert } from 'node:assert';

function assertEqual<T>(actual: T, expected: NoInfer<T>): void {
  assert.equal(actual, expected);
}

function assertDeepEqual<T>(actual: T, expected: NoInfer<T>): void {
  assert.deepEqual(actual, expected);
}

const count: number = getCount();

assertEqual(count, 42);
assertEqual(count, 'forty-two'); // TypeScript error
```

Use an explicitly typed expected value in older TypeScript versions. Use these
helpers only for values with a useful known static type. Narrow an `unknown`
value before comparing it.

## Errors

Use a validator when the error needs more than a class or message check:

```ts
await assert.rejects(loadUser(), (error: unknown) => {
  assert(error instanceof DomainError);
  assert.equal(error.code, 'USER_NOT_FOUND');
  assert.match(error.message, /user-1/);

  return true;
});
```

`assert.rejects` tests promise rejection. A synchronous throw from its callback
bypasses the error validator. When an API can either throw or reject and both
paths are part of the same contract, normalize both paths to a rejection:

```ts
await assert.rejects(
  Promise.resolve().then(() => loadUser()),
  DomainError,
);
```

Use `assert.throws` instead when a synchronous throw is the required behavior.

Use a regular expression, error class, validation object, or validator as the
second argument to `assert.throws` and `assert.rejects`. A string in that
position is an assertion failure message, not an expected error message.
