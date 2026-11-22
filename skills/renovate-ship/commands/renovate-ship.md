---
description:
  List or track Renovate requests, show release notes, and handle updates
subagent: false
---

Load the `renovate-ship` skill and follow it in this OpenCode session for the
current GitHub or GitLab project. With no arguments, discover and track open
Renovate requests only. Accept `see all` (alias `see all mrs`) to list open
Renovate requests without tracking. Match these forms before `see <dependency>`.
Also accept `see <dependency>`, `merge <dependency>`, or `fix <dependency>` for
one update. For other arguments, show usage without taking action.

$ARGUMENTS
