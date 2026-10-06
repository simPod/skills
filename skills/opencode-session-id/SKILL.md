---
name: opencode-session-id
description:
  Retrieve the exact OpenCode V2 session ID for the current chat. Use when the
  user asks for an OpenCode session ID, wants to reference the current session,
  or wants to post the session ID to an issue or merge request.
license: MIT
compatibility: opencode
metadata:
  version: '0.2.0'
  author: simPod
---

# OpenCode V2 Session ID

Retrieve the session for this chat, not merely the most recently updated session
in the current project. Use the server API, not direct SQLite queries. V2 does
not have the V1 `session`, `message`, and `part` database contract.

## Instructions

1. Load the `opencode-session-db` skill and follow its V2 lookup and
   verification instructions. If it is unavailable, read the V2
   [API guide](https://opencode.ai/v2/docs/api) and use `opencode api` with the
   existing server and authentication context. Do not guess endpoint shapes.
2. If the user supplied an exact session ID, inspect that ID. Do not substitute
   another session. Prefer an exact current-session ID supplied by the harness
   when one is available; verify it with the API before reporting it.
3. Otherwise, resolve the current project or worktree directory. Choose two to
   four distinctive identifiers from this chat, such as an MR number, branch
   name, class name, or error string. Do not use generic words or the project
   name as proof of identity.
4. List candidate sessions for that exact directory. Follow session cursors when
   the first page has no unique match. Read every message page for each
   candidate and match all identifiers in user-authored text only. Different
   identifiers can occur in different user messages.
5. Return an ID only when exactly one candidate matches. Verify its ID, title,
   directory, update time, and archive state through the API. Do not print
   message contents.
6. If no candidate matches, retry with a less-specific distinctive identifier.
   If multiple candidates match, show their IDs and metadata, then ask the user
   to choose under a **Your Input Needed** heading. Never choose the newest
   candidate merely because it is newest.
7. If no distinctive identifier exists, use the single non-archived session only
   after checking all session pages. Otherwise, ask for an exact ID or
   distinctive phrase. A native archive timestamp is not the same as a
   file-backed archive created by the optional session-archive plugin.

Session identification is read-only. Do not rename, delete, import, or change
archive state to find an ID. Publishing the verified ID requires an explicit
request; use the requested issue-tracker or merge-request tool.
