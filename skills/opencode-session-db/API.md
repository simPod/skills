# V2 Session API Examples

Use `opencode api` with the same server context as the current chat. These
examples use operation IDs from the V2
[API contract](https://opencode.ai/v2/openapi.json). Check the current
[API reference](https://opencode.ai/v2/docs/api) before acting.

## List and Inspect

```sh
PROJECT_ROOT=$(git rev-parse --show-toplevel 2>/dev/null || pwd -P)
opencode api session.list --param "directory=$PROJECT_ROOT" --param limit=50

SESSION_ID='<exact-session-id>'
opencode api session.get --param "sessionID=$SESSION_ID"
```

List responses contain a `data` array and `cursor.previous` / `cursor.next`.
Read `location.directory`, `time.created`, `time.updated`, and `time.archived`
from each session. Timestamps are milliseconds since the Unix epoch. A get
response contains one session under `data`.

Follow `cursor.next` until it is absent or null, or an empty page ends the list:

```sh
CURSOR='<cursor.next-from-the-previous-response>'
opencode api session.list --param "directory=$PROJECT_ROOT" \
  --param limit=50 --param "cursor=$CURSOR"
```

Keep the directory and all other filters unchanged across pages. Omit the
directory filter only for an explicit all-projects search. A page limit is not a
total count. Do not use default list order to infer the most recently updated
session; sort by `time.updated` after collecting the complete list.

## Read User Messages

```sh
opencode api session.message.list --param "sessionID=$SESSION_ID" \
  --param type=user --param limit=50 --param order=desc
```

Messages are objects in `data`, with `type: "user"` and a `text` string. They
are not V1 `info` / `parts` objects. Match identifiers locally against `text`;
print only session metadata, not user messages or attachments.

```sh
CURSOR='<cursor.next-from-the-previous-response>'
opencode api session.message.list --param "sessionID=$SESSION_ID" \
  --param type=user --param limit=50 --param "cursor=$CURSOR"
```

Keep `type=user` on every page. Do not combine a message cursor with `order`.
Stop at the end of the list, not merely after the first page. If a response is
truncated or cannot be parsed, reduce the page limit and retry; never treat an
incomplete response as a complete search.

## Rename and Verify

Only run this after an explicit rename request. This example requires `jq` to
encode quotes, backslashes, and newlines safely. If it is unavailable, use
another JSON encoder; do not interpolate a title into hand-built JSON or a shell
assignment. The quoted heredoc prevents shell expansion. Choose a delimiter that
does not occur as a complete line in the requested title.

```sh
SESSION_ID='<exact-session-id>'
opencode api session.get --param "sessionID=$SESSION_ID"
PAYLOAD=$(jq -Rs '{title: rtrimstr("\n")}' <<'OPENCODE_SESSION_TITLE'
<requested-title>
OPENCODE_SESSION_TITLE
)
opencode api session.update --param "sessionID=$SESSION_ID" --data "$PAYLOAD"
opencode api session.get --param "sessionID=$SESSION_ID"
```

Replace the title line with the exact requested text. `rtrimstr("\n")` removes
only the newline added by the heredoc, not intentional newlines in the title.

`session.update` is `PATCH /api/session/{sessionID}` and returns HTTP 204 on
success. Confirm that the final `data.id` and `data.title` equal the requested
values. Do not send an archive timestamp: the V2 update contract has no such
field and no documented restore operation.

## Archive Storage Is Not an Archive API

In [V2.0.24](https://github.com/anomalyco/opencode/tree/v2.0.24), the V2 table
is `session_v2`, and `time_archived` stores a nullable Unix timestamp in
milliseconds. The legacy `session` table is not the V2 session table.

Directly setting this column has only partial effects: the web/desktop app hides
archived sessions in several views, but the API and TUI picker still list them.
SQL writes also bypass session events, so open clients can remain stale. This is
not server-enforced archival.

Do not add or run a direct-write workaround without explicit user approval. Any
separately approved offline procedure needs all database-using OpenCode
processes stopped, a consistent backup, the exact database path and session ID,
and clients restarted afterward. Do not call `opencode api` during offline work:
it can start the service. Prefer a supported archive API when available.
