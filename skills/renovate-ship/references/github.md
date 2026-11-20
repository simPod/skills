# GitHub workflow

## Discovery and tracking

1. Use the repository and host from the current Git remote. For the hosted
   Renovate App, list open PRs with
   `gh pr list --repo <host/owner/repo> --state open --app renovate --json number,title,url,author`.
   Set an explicit `--limit`. If the result reaches it, or the project uses a
   different Renovate identity, enumerate all pages with
   `gh api --hostname <host> --paginate 'repos/<owner>/<repo>/pulls?state=open&per_page=100'`.
   Match the App or the configured bot against author metadata, not the PR title
   or branch. Include drafts and grouped updates.
2. When tracking is authorized, check the owning session's saved links and
   exclusions. For a missing link, use a direct
   `gh api --hostname <host> repos/<owner>/<repo>/pulls/<number>` if the
   installed tracker supports GitHub API adoption. Keep the canonical `html_url`
   in the result. Otherwise, read the live title and run
   `gh pr edit <full-pr-url> --title "<unchanged title>"` as a literal argument.
   Preserve all other fields. Verify the saved link. If neither route works,
   report the unconfirmed URL; do not offer GitLab-only `/mr-track` as a GitHub
   fallback or edit tracker storage.

## CI and merge

1. Read
   `gh pr view <url> --json headRefOid,baseRefName,state,isDraft,mergeable,mergeStateStatus,reviewDecision,statusCheckRollup`
   and `gh pr diff <url>`. Use `gh pr checks <url> --required` and live branch
   protection or rulesets to identify required checks, including external
   statuses. Use `gh run watch` for GitHub Actions and
   `gh run view --job <id> --log-failed` for failed jobs. Match runs to the
   current PR head. Recheck and review a replacement revision before using its
   CI.
2. For a transient failure, rerun the failed workflow jobs after the cause
   clears. If fresh CI needs a newer base, update the PR branch only when this
   preserves the intended diff. For a real conflict or CI fix, follow the shared
   CI workflow and regenerate any conflicted lockfile with its package manager.
   Preserve the dependency update and existing checks.
3. Before merging, verify the intended GitHub account, current head, diff,
   required approvals, passing checks, and conflict-free state. Do not post an
   unsolicited review or bypass required reviewers. If GitHub blocks merge,
   report the reason.
4. Select a merge method allowed by the repository and its conventions. Run
   `gh pr merge <url> --match-head-commit <head-sha> <merge-method-flag>` (for
   example, `--squash`). Do not use `--admin` or `--auto`. When a merge queue is
   required, submit an eligible PR through the normal queue without a
   merge-method flag, only after all required pre-queue conditions pass. Recheck
   live state and report a queued PR as pending until GitHub confirms `MERGED`.
