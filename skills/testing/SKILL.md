---
name: testing
description: Apply shared policy for choosing, reviewing, and running tests and other validation. Use when adding, changing, removing, or reviewing tests; diagnosing analyzer or CI results; or running validation. Combine with language- and framework-specific testing skills when relevant.
---

# Testing Policy

Use this policy together with applicable project instructions and specialized skills such as `test-audit`, `typescript-testing`, `frontend-testing`, and `tdd`. This skill defines shared quality policy; specialized skills define their own procedures.

## Choosing and reviewing tests

- Start with the smallest sufficient test set, normally one owner-level scenario for one behavior. Add a test or provider case only when it prevents a distinct material failure that existing owner coverage cannot detect.
- Add tests for critical behavior with a concrete failure and material user or operational impact, such as authorization, destructive actions, data integrity, essential workflows, or important protocol guarantees. Before adding or retaining a changed test, identify the incorrect outcome it prevents and why the assertions distinguish it from correct behavior.
- Test a decision where it is made. Explicit callbacks, prop forwarding, dependency injection, and manual wiring are normal composition, not defects or automatic reasons for tests. Do not add caller tests only to prove separately tested behavior is connected. Test a caller's distinct material decision, protocol translation, effect ordering, or lifecycle when owner tests cannot establish it.
- For critical authorization and destructive-action decisions, test the real action path, including restored URL state when relevant. Assert whether the protected action is available or executed; a helper's return value alone does not prove the guard works.
- Avoid standalone tests with large fixtures, router setups, or mocks for trivial, low-risk UI wiring. Prefer a small assertion in relevant coverage when useful, otherwise use static checks and focused review. Keep dedicated coverage for material consequences.
- Keep focused tests when object identity or caching is a correctness contract (for example, production relies on reference equality). Shared helper tests do not replace a concrete type's factory contract.
- Before publication, review all added or changed tests against production owners and existing coverage. Remove duplicate coverage and trivial forwarding assertions; use the smallest fixture that distinguishes the credible regression. Apply verified simplifications within authorized implementation work; for question-only reviews, report them without editing.
- Reduce duplication by deleting repeated cases and unused setup, not by merging unrelated behavior into one large test or adding production seams. Keep each test focused on one operation or coherent lifecycle. Separate distinct failure modes, edge cases, and scenarios needing different setup.
- Do not remove unique critical proof or weaken quality gates to reduce test count. Passing CI does not prove a test is useful or simple.
- Organize tests around public operations and observable results. Prefer operation-level scenarios and meaningful named provider cases over narrow tests named only for the latest changed field. For validation, cover meaningful accepted and rejected forms at the boundary.
- Remove unused fixture filters, dependencies, callback parameters, repeated assertions, and provider rows whose only distinction no longer affects behavior. Do not replace setup noise with generic test frameworks or production seams.
- For copy, preservation, and partial-update tests, use source values that differ from defaults, parent values, and fallbacks. Caller orchestration assertions must distinguish the caller's choice without repeating dependency rules.
- Avoid private helpers, byte offsets, and internal call details unless they are the contract under test. Keep separate UI lifecycles and ownership boundaries clear (for example, selection/filtering versus renderer presentation).

## Validation and CI

- A request to fix CI does not include dependency or security audit failures. Handle those only when the user separately requests audit work.
- Prefer existing upstream CI, especially GitLab pipelines. Do not run duplicate local validation suites just to repeat checks upstream CI will run. When Docker is unavailable or paused, prefer remote CI over local Docker-based validation.
- Run local checks when the user requests them, the repository requires a local-only step, or a narrowly scoped command is the fastest way to diagnose or unblock the task.
- Do not change production code to satisfy an analyzer result from a scoped command. Reproduce it with the exact full-source command used by CI; scoped runs may omit aliases, stubs, or configuration.
- For a confirmed analyzer false positive, preserve the source-of-truth declaration and use the smallest precise suppression. Follow the analyzer's project policy; do not add Mago baseline entries when the project forbids them. Draft an upstream issue only after full-source reproduction confirms a defect.
- Do not weaken typecheck, lint, test, security, or build gates without explicit user approval. Fix the issue or stop and ask before changing a gate.
