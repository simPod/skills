---
name: test-audit
description: >-
  Audit tests for low-value, duplicate, or implementation-coupled assertions.
  Use when writing or changing tests, reviewing test coverage, or pruning a
  subsystem's tests and test-only production seams.
license: MIT
compatibility: opencode
metadata:
  version: '0.1.0'
  author: simPod
---

# Test Audit

Use the same value bar when authoring tests and when auditing existing ones. For
a whole-subsystem audit, read [CAMPAIGN.md](CAMPAIGN.md) before starting. Choose
confidence over the number of tests removed.

## Authoring gate

Before adding or changing a test, answer:

1. Which observable behavior, invariant, or independent contract does it
   protect?
2. Which credible regression makes it fail for the intended reason?
3. Why does existing coverage not catch that regression? Name the primary test
   owner at the strongest practical boundary. Another layer needs its own
   distinct risk, such as transport or lifecycle behavior the owner cannot
   reach.
4. Does the test require a production export, flag, wrapper, or injection hook
   with no production caller? If so, test through the real boundary instead.

Prefer extending a table-driven case or shared fixture to repeating a scenario.
Check every new test against the [junk patterns](#junk-patterns). A match needs
an independently guarded contract from the [retention bar](#retention-bar).
Tests that break on behavior-preserving refactors should be rewritten at the
owning boundary. For a bug regression, show the test fails on pre-fix code for
the intended reason and passes with the fix, when the pre-fix state is available
and a control run is practical.

## Junk patterns

Use this checklist for new tests and audit candidates:

- Assertion-free coverage probes, self-comparisons, and identity copiers.
- Copied fixtures, inventories, manifests, or export lists that only restate
  source.
- Source/import/string greps that track implementation rather than a contract.
- Private predicate and call-shape tests already covered at a real boundary.
- Duplicate assertions of the same contract in several layers or providers.
- Tests kept only to preserve test-only exports, globals, or wrappers.
- Dead production paths whose only callers are tests.
- Expected values calculated by the helper or renderer under test.
- Mocks that implement the behavior being asserted, or one interchangeable mock
  for APIs with different behavior.
- Fixtures that supply the ordering, admission, or persisted state that the
  production owner should produce.
- Capability tests that only repeat declared flags rather than exercise
  delivery.
- Negative controls that pass for an unrelated reason or miss the intended
  guard.
- Names or fixtures promising behavior the test input and assertions never
  cover.

## Value and retention bar

A test earns its maintenance cost when it independently guards behavior, a
credible regression, or a meaningful contract. Keep independent guards of public
APIs, protocols, configuration, migrations, persistence, security, platform
behavior, defaults, byte-exact outputs, generated interfaces, packaging, release
behavior, and architectural boundaries when these are actual contracts of the
repository. Keep observable call ordering and source inspection when source
inspection is the cheapest independent guard of a stable external key, byte, or
path. Static or slow is not itself a reason to delete a test.

A test that fails on the baseline may expose a product defect. Reproduce the
failure before deciding whether to change the test, product, or neither.

## Focused audit workflow

1. Read repository instructions, test commands, CI routing, and the requested
   scope. Discover candidates read-only, across the relevant packages, apps,
   tooling, and shared boundaries. Select a small coherent group of high-
   confidence candidates. Discovery is complete when each candidate has an
   identified production owner and likely keeper test.
2. For each candidate, read the full test, production owner, entry points,
   callers, nearby and overlapping tests, relevant history, and applicable
   dependency contracts. Record the evidence below before editing. The group is
   ready when every proposed deletion has complete evidence.
3. Edit one owner-boundary group. Keep each distinct contract at its strongest
   practical boundary; carry over any unique assertions before removing a weaker
   duplicate. Remove a test-only seam only after confirming it has no production
   callers or independent contract. The edit is complete when all retained
   contracts have an identified keeper.
4. Run the narrow owner and sibling tests using the repository's test runner.
   For deleted source checks or plans, run the executable path or safe dry-run
   that owns the contract. Run the repository's required checks and inspect the
   diff for lost coverage. The audit is complete when the relevant tests pass
   (or failures are classified), and every deleted contract has a keeper or a
   documented reason it needs no proof.

### Candidate evidence

Record for each proposed deletion:

- Exact test name and location, and what failure it can actually detect.
- Production owner and non-test callers of the covered code or support seam.
- Stronger remaining proof, or why no proof is needed.
- Relevant history and why the test or seam exists.
- Test-support or production deletion unlocked, if any.
- Risk and focused validation command.

## Handoff

Report the low-value patterns found, contracts preserved and where they live,
production simplifications, false positives retained and why, tests and checks
actually run, unresolved failures, and follow-ups. Distinguish production or
tooling changes from test and test-support changes. Follow the repository's own
approval, commit, and review workflow.
