---
name: opencode-session-db
description: Finds, renames, and restores OpenCode sessions in the local SQLite database. Use when locating a prior OpenCode discussion, fetching a session ID, checking archive status, renaming a session, or unarchiving an OpenCode session.
license: MIT
compatibility: opencode
metadata:
  version: '0.4.0'
  author: simPod
---

# OpenCode Session Database

Use `opencode db` for all database access. Do not locate the database file or
call `sqlite3` directly. Use `--format tsv` for readable tabular results and
`--format json` when structured output is useful.

## Scope

Resolve the current project root. Search only sessions whose `directory` equals
that root unless the user asks to search all projects. Escape it for SQL:

```sh
PROJECT_ROOT=$(git rev-parse --show-toplevel)
PROJECT_ROOT_SQL=${PROJECT_ROOT//\'/\'\'}
```

## Current Chat Session

Retrieve the session for the current chat, not merely the most recently updated
session in the current project.

1. If the user supplied an exact session ID, inspect that ID and use it. Do not
   search for a substitute.
2. Otherwise, select two to four distinctive identifiers from the current chat.
   Prefer a merge-request number, branch name, class name, error string, or
   another exact phrase. Do not use generic terms such as `session`, `OpenCode`,
   `ID`, or the project name.
3. Search the 20 most recently updated project sessions. A candidate must
   contain every identifier in user-authored text, even when the identifiers
   occur in different messages.

```sh
opencode db --format tsv "
  WITH recent_sessions AS (
    SELECT id, time_archived, time_updated, title
    FROM session
    WHERE directory = '$PROJECT_ROOT_SQL'
    ORDER BY time_updated DESC
    LIMIT 20
  )
  SELECT recent_sessions.id, recent_sessions.title,
         datetime(recent_sessions.time_updated / 1000, 'unixepoch', 'localtime') AS updated_at,
         CASE WHEN recent_sessions.time_archived IS NULL THEN 'active' ELSE 'archived' END AS archive_status
  FROM recent_sessions
  WHERE EXISTS (
    SELECT 1
    FROM message
    JOIN part ON part.message_id = message.id
    WHERE message.session_id = recent_sessions.id
      AND json_extract(message.data, '$.role') = 'user'
      AND json_extract(part.data, '$.type') = 'text'
      AND lower(coalesce(json_extract(part.data, '$.text'), '')) LIKE '%<identifier-1>%'
  )
  AND EXISTS (
    SELECT 1
    FROM message
    JOIN part ON part.message_id = message.id
    WHERE message.session_id = recent_sessions.id
      AND json_extract(message.data, '$.role') = 'user'
      AND json_extract(part.data, '$.type') = 'text'
      AND lower(coalesce(json_extract(part.data, '$.text'), '')) LIKE '%<identifier-2>%'
  )
  ORDER BY recent_sessions.time_updated DESC;
"
```

4. Add one `EXISTS` block for each additional identifier. Replace identifiers
   with correctly SQL-escaped, lower-case values.
5. Return the ID only when exactly one candidate matches. Inspect that session's
   ID, title, directory, update time, and archive state before returning it.
6. If no candidate matches, retry with one less-specific identifier. If multiple
   candidates match, show their IDs, titles, update times, and archive states,
   then ask the user to choose. Never select a candidate only because it is
   newest.
7. If no distinctive identifier exists, return an ID only when the project has
   exactly one non-archived session. Otherwise, ask for an exact ID or a
   distinctive phrase.

Read only user-authored message text to identify the chat. Do not print message
contents. `time_updated` limits and orders candidates; it does not select one.

## Find a Previous Discussion

Search user-authored text first to avoid false matches from tool output, agent
reasoning, and generated files. Replace `<term>` with a lower-case search term.
Add `AND` clauses for multiple required terms.

```sh
opencode db --format tsv "
  SELECT s.id, s.title,
         datetime(s.time_updated / 1000, 'unixepoch', 'localtime') AS updated_at,
         CASE WHEN s.time_archived IS NULL THEN 'active' ELSE 'archived' END AS archive_status
  FROM session AS s
  JOIN message AS m ON m.session_id = s.id
  JOIN part AS p ON p.message_id = m.id
  WHERE s.directory = '$PROJECT_ROOT_SQL'
    AND json_extract(m.data, '$.role') = 'user'
    AND json_extract(p.data, '$.type') = 'text'
    AND lower(coalesce(json_extract(p.data, '$.text'), '')) LIKE '%<term>%'
  GROUP BY s.id, s.title, s.time_updated, s.time_archived
  ORDER BY s.time_updated DESC
  LIMIT 30;
"
```

If this has no result, broaden one term at a time or search all parts. Report
the ID, title, update time, and archive status, and label possible matches.

## Inspect Before Restore

```sh
opencode db --format tsv "
  SELECT id, title, directory,
         datetime(time_created / 1000, 'unixepoch', 'localtime') AS created_at,
         datetime(time_updated / 1000, 'unixepoch', 'localtime') AS updated_at,
         CASE WHEN time_archived IS NULL THEN 'active' ELSE 'archived' END AS archive_status
  FROM session
  WHERE id = '<session-id>';
"
```

## Restore an Archived Session

Only unarchive after the user explicitly requests it and gives or confirms the
exact ID. Update only `session.time_archived` and verify it immediately.

```sh
opencode db --format tsv "
  UPDATE session
  SET time_archived = NULL
  WHERE id = '<session-id>';

  SELECT id, title, time_archived IS NULL AS is_unarchived
  FROM session
  WHERE id = '<session-id>';
"
```

An `is_unarchived` value of `1` confirms success. Do not change session messages or parts.

## Rename a Session

Rename only after the user explicitly requests it and gives or confirms both the
exact session ID and new title. Inspect the session first. Escape both values
before the SQL update, update only `session.title`, and verify the new title
immediately.

```sh
SESSION_ID='<session-id>'
SESSION_TITLE='<new title>'
SESSION_ID_SQL=${SESSION_ID//\'/\'\'}
SESSION_TITLE_SQL=${SESSION_TITLE//\'/\'\'}

opencode db --format tsv "
  UPDATE session
  SET title = '$SESSION_TITLE_SQL'
  WHERE id = '$SESSION_ID_SQL';

  SELECT id, title
  FROM session
  WHERE id = '$SESSION_ID_SQL';
"
```

The result must contain the exact session ID and requested title. Do not change
session messages, parts, archive state, or timestamps.
