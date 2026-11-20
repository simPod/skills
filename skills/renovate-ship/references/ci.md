# CI for a Renovate MR

1. Find all required checks for the MR's current head SHA. Check external CI as
   well as GitLab. For TeamCity, load `teamcity-cli`, match the runs to the MR
   revision, and watch the relevant runs to completion. Do not mistake a
   successful GitLab pipeline or a passing run for an older revision for a
   passing external check.
2. If checks are queued or running, wait in the current task with a bounded
   provider watch. Continue when it returns, using background tool completion
   when available. While waiting, report progress if useful; pending CI alone is
   not a request for user input. If the CI provider cannot be observed or a
   reasonable watch limit is reached, report the exact state and whether a
   watcher is still active. Do not claim to monitor an inactive watch.
3. If a check fails, inspect the failed job logs and identify the cause.
   Distinguish an update-related failure from a transient failure or unrelated
   baseline failure. Leave dependency and security audit findings and jobs alone
   unless separately requested.
4. Fix the cause in a clean worktree for the same MR's source branch. Preserve
   the Renovate update and unrelated work. Do not weaken checks. Push only to
   that MR's source branch, then wait for the required checks on the pushed
   revision. Do not treat a stale run as proof. Stop and report a concrete
   blocker if you cannot make a safe in-scope fix, cannot push, or the same
   failure persists without a new remedy. Do not create another MR.

For `merge`, the same request covers this CI work and the eventual approval and
merge. Before proceeding, recheck the live MR and its diff so the current head
still contains the intended update and only expected changes. If unexpected
changes or another real blocker need a user decision, ask about that blocker,
not for a repeat of the merge command.
