---
name: opencode-session-db
description: Finds OpenCode sessions in the local SQLite database and safely restores archived sessions. Use when locating a prior OpenCode discussion, fetching a session ID, checking archive status, or unarchiving an OpenCode session.
license: MIT
compatibility: opencode
metadata:
  version: '0.1.0'
  author: simPod
---

# OpenCode Session Database
```sh
DB="${XDG_DATA_HOME:-$HOME/.local/share}/opencode/opencode.db"
```

## Scope

Resolve the current project root. Search only sessions whose `directory` equals
that root unless the user asks to search all projects. Escape it for SQL:

```sh
PROJECT_ROOT=$(git rev-parse --show-toplevel)
PROJECT_ROOT_SQL=${PROJECT_ROOT//\'/\'\'}
```

## Latest Active Session

Return the most recently updated, non-archived session ID:

```sh
sqlite3 -noheader "$DB" "
  SELECT id
  FROM session
  WHERE directory = '$PROJECT_ROOT_SQL'
    AND time_archived IS NULL
  ORDER BY time_updated DESC
  LIMIT 1;
"
```

Return only the ID when the user asks only for the session ID.

## Find a Previous Discussion

Search user-authored text first to avoid false matches from tool output, agent
reasoning, and generated files. Replace `<term>` with a lower-case search term.
Add `AND` clauses for multiple required terms.

```sh
sqlite3 -header -column "$DB" "
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
sqlite3 -header -column "$DB" "
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
sqlite3 "$DB" "
  UPDATE session
  SET time_archived = NULL
  WHERE id = '<session-id>';

  SELECT id, title, time_archived IS NULL AS is_unarchived
  FROM session
  WHERE id = '<session-id>';
"
```

An `is_unarchived` value of `1` confirms success. Do not change session messages or parts.
