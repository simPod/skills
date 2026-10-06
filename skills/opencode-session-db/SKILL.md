---
name: opencode-session-db
description:
  Finds and renames OpenCode V2 sessions through the server API. Use when
  locating a prior OpenCode discussion, fetching a session ID, checking archive
  status, renaming a session, or archiving and restoring session transcripts.
license: MIT
compatibility: opencode
metadata:
  version: '0.6.1'
  author: simPod
---

# OpenCode V2 Session Management

Use `opencode api` for session access. V2 has no `opencode db` command. Do not
read or modify the live SQLite database directly. Keep the skill ID
`opencode-session-db` unchanged so existing installations still find it.

Before acting, read the V2 [API reference](https://opencode.ai/v2/docs/api) and
[command examples](API.md). Use only V2 documentation, not V1 API contracts. The
API command uses service discovery and authentication and can start the
background service. Keep any explicit server and authentication context.

## Scope

Resolve the current project or worktree root. Outside Git, use the current
directory. Filter with the API's `directory` parameter and confirm that each
candidate's `location.directory` equals that root. Search all projects only when
the user asks. Do not substitute the main checkout for a worktree.

```sh
PROJECT_ROOT=$(git rev-parse --show-toplevel 2>/dev/null || pwd -P)
opencode api session.list --param "directory=$PROJECT_ROOT" --param limit=50
```

## Current Chat Session

Retrieve the session for the current chat, not merely the most recently updated
session in the current project.

1. If the user supplied an exact session ID, inspect that ID and use it. If the
   harness supplies the current session ID explicitly, inspect and use it. Never
   infer an ID from a subagent result or select a substitute for an exact ID.
2. Otherwise, select two to four distinctive identifiers from the current chat.
   Prefer a merge-request number, branch name, class name, error string, or
   another exact phrase. Do not use generic terms such as `session`, `OpenCode`,
   `ID`, or the project name.
3. Page through the project sessions. Sort them locally by `time.updated` and
   inspect the 20 most recently updated candidates; API list order is not an
   update-time guarantee. Retrieve messages with `type=user`. A candidate must
   contain every identifier in user-authored `text`, even across messages.
4. Follow message cursors until all identifiers match or all user messages have
   been checked. Compare case-insensitive literal text, not SQL patterns.
5. Return the ID only when exactly one candidate matches. Inspect that session's
   ID, title, `location.directory`, `time.updated`, and `time.archived` before
   returning it.
6. If no candidate matches, retry with one less-specific identifier. If multiple
   candidates match, show their IDs, titles, update times, and archive states,
   then ask the user to choose. Never select a candidate only because it is
   newest.
7. If no distinctive identifier exists, return an ID only when the complete
   project session list has exactly one non-archived session. Otherwise, ask for
   an exact ID or a distinctive phrase.

Read only user-authored message text to identify the chat. Do not print message
contents. `time.updated` limits and orders candidates; it does not select one.
An incomplete or failed API response is not evidence that no session matches.

## Find a Previous Discussion

Page through project sessions and their user messages. Use the same literal text
matching as above. Do not assume the session-list `search` parameter searches
message content. Multiple terms may occur in different user messages.

If nothing matches, broaden one term at a time. Read other message types only
when needed and label those results as possible matches. Report only the ID,
title, directory, update time, and archive state, not message contents.

## Archive and Restore

Inspect the exact session with `session.get`. A present `time.archived` value
means archived; an absent value means non-archived, not necessarily running.

V2 has no native archive-timestamp update API. Do not edit SQLite or invent a
request. If the user's `opencode-plugin-sessions` plugin is enabled, follow
[ARCHIVE.md](ARCHIVE.md) for its confirmed export/delete/import workflow.
Without it, explain the limit; do not delete sessions as an ad hoc workaround.

## Rename a Session

Rename only after an explicit request with the exact session ID and new title.
Inspect first, JSON-encode the title, and send only `title` with
`session.update` as shown in [API.md](API.md). Then read the session again and
verify the exact ID and title. A successful update returns no body; it is not
verification. Do not change messages, permissions, metadata, or archive state.
The server owns any update-time changes.
