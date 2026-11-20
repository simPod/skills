# CI for a Renovate PR or MR

1. Find all required checks for the request's current head SHA. Check external
   CI as well as the Git host. Use project rules and live status to determine
   what blocks merge; a status on an older commit does not establish a merge
   requirement. For TeamCity, load `teamcity-cli`, match the runs to the MR
   revision, and watch the relevant runs to completion. Do not mistake a
   successful GitLab pipeline or a passing run for an older revision for a
   passing external check.
2. If required checks are queued or running, wait in the current task with a
   bounded provider watch. Continue when it returns, using background tool
   completion when available. While waiting, report progress if useful; pending
   CI alone is not a request for user input. If the CI provider cannot be
   observed or a reasonable watch limit is reached, report the exact state and
   whether a watcher is still active. Do not claim to monitor an inactive watch.
3. If a check fails, inspect the failed job logs and identify the cause.
   Distinguish an update-related failure from a transient failure or unrelated
   baseline failure. Leave dependency and security audit findings and jobs alone
   unless separately requested.
4. Fix the cause in a clean worktree for the same request's source branch.
   Preserve the Renovate update and unrelated work. Do not weaken checks. Push
   only to that request's source branch, then wait for the required checks on
   the pushed revision. Do not treat a stale run as proof. Stop and report a
   concrete blocker if you cannot make a safe in-scope fix, cannot push, or the
   same failure persists without a new remedy. Do not create another PR or MR.

For `merge`, the same request covers this CI work and the eventual approval and
merge. Before proceeding, recheck the live request and its diff so the current
head still contains the intended update and only expected changes. If unexpected
changes or another real blocker need a user decision, ask about that blocker,
not for a repeat of the merge command.

## Renovate release age after a rebase

`renovate/stability-days` is Renovate's status for its `minimumReleaseAge` rule.
A missing status on the current SHA is not a failure or proof of a merge
requirement, even if the old status had `allow_failure: false`.

- If Renovate reports the current release is too new, wait for it to meet the
  threshold. Investigate any other pending or failing release-age status.
- If the Git host or project policy requires the status, wait for a pass on the
  current SHA. Never post a replacement status or bypass a merge rule.
- If the status is missing, is not required, and a release-age rule applies,
  check the effective threshold, including inherited presets. Compare the exact
  versions and available source or distribution references from the current diff
  and lockfile with the authoritative registry (for example, Packagist). Verify
  the registry's publication time against the threshold for each changed
  release, including transitive changes in the lockfile. If the rule cannot be
  verified or a release is too new, report the concrete blocker.
- If the releases qualify and all actual required checks pass on the current
  head, continue the ordinary approval and merge workflow. Do not wait for
  Renovate to repost the status. Say the release age was verified directly, not
  that the bot check passed.

For example, if a rebase keeps the same Composer releases and the registry
confirms they all meet the configured three-day minimum, a missing bot status
alone does not block the merge.

A package manager can enforce its own release-age policy independently of
Renovate. For a pnpm `ERR_PNPM_MINIMUM_RELEASE_AGE_VIOLATION`, verify the exact
locked release and publication time against the registry and the effective pnpm
threshold. Wait until it qualifies before rerunning the failed jobs. If that is
outside the watch window, report the earliest retry time. Do not add exemptions,
lower the threshold, or treat elapsed time as passing CI.
