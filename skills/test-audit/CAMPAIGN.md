# Test-pruning campaign

Use campaign mode to audit one subsystem's full test surface. The value bar,
retention bar, candidate evidence, and validation in [SKILL.md](SKILL.md) apply
to each step. Keep a written ledger so that a large deletion can be reviewed.

## 1. Baseline

Pin a base revision and record every in-scope test file's pass/fail result and
the subsystem's test, support, and production line counts. Include baseline
failures as possible product bugs, not automatic deletion candidates.

Done when every in-scope test file has a recorded baseline result or a stated
reason it cannot run.

## 2. Inventory

Group test files and scenarios by production owner boundaries. Include shared
boundary tests owned by the subsystem, as well as relevant end-to-end or manual
proof. Split discovery across independent lanes when tools and collaborators are
available.

Done when each in-scope test and scenario belongs to exactly one lane.

## 3. Read-only ledger

Read each assigned test declaration, including parameterized cases, plus its
production owner, entry points, callers, overlapping coverage, relevant history,
and CI routing. Mark each declaration (or a parameter row if rows differ):

- `R`: retain; name the contract and the regression it catches.
- `F`: retain the contract but fix an assertion that can pass vacuously.
- `C`: consolidate; name the stronger owner that will absorb its unique proof.
- `D`: delete; name the remaining proof or explain why no contract exists.

Judge assertions rather than names. Apply the candidate evidence fields from the
main skill before proposing any deletion.

Done when every declaration has a mark and an evidence line.

## 4. Layer plan

Review the ledger by contract, not just by file. Find redundant layers and name
one keeper for each contract. Prefer a real external boundary with a controlled
fake dependency over a mock that supplies the asserted behavior. Identify
assertions to carry into keepers and test-only seams that can safely be removed.

Done when the plan names retired suites, keepers, transferred assertions, and
any production seams affected.

## 5. Cutover and preservation

Edit lane by lane. Give shared test support one edit owner. Update test
registration, CI routing, and inventories where the repository uses them. Run
the narrow owner and sibling checks after each lane. Independently compare the
removed assertions with the keepers; restore any lost distinct contracts. Where
practical, verify a restored contract catches a deliberate temporary mutation of
the production owner, then restore the source exactly.

Done when each lane's checks pass or failures are classified, and every
preservation gap has been restored or rejected with evidence.

## 6. Reconcile and hand off

If the base changes during a long campaign, compare new tests and contracts
against the keeper plan. Use the repository's normal integration workflow and
rerun the full subsystem suite when available. Handle product defects found on
the baseline as separate, evidenced fixes within the agreed scope.

Done when new contracts still have owners and the final report includes the main
skill's handoff items, baseline and final line counts, lanes and keepers,
preservation gaps, and any product defects with their control and passing proof.
