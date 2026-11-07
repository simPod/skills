# V2 Session API Examples

Use `opencode api` with the same server context as the current chat. These
examples use operation IDs from the V2
[API contract](https://opencode.ai/docs/openapi.json). Check the current
[API reference](https://opencode.ai/docs/api) and the connected server's
`/openapi.json` before acting. The connected server's schema determines which
operations and fields are available.

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
values. Send only `title` for a rename; archive and restore use a separate
boolean field, not an archive timestamp.

## Native Archive and Restore

An ordinary archive request changes native session state without deleting the
session or transcript. A restore clears that state; it does not import a copy.
Resolve the exact session and require an explicit request before either update.

Check support on the connected server:

```sh
opencode api get /openapi.json | jq \
  '.paths["/api/session/{sessionID}"].patch.requestBody.content["application/json"].schema.properties.archived'
```

The schema must accept `archived` as a boolean. Older servers may lack this
field; do not infer support or absence from the general V2 label. If
unsupported, report the limitation. Do not invent another endpoint, write
SQLite, or fall back to export-and-delete for an ordinary archive request.

Archive:

```sh
SESSION_ID='<exact-session-id>'
opencode api session.get --param "sessionID=$SESSION_ID"
opencode api session.update --param "sessionID=$SESSION_ID" \
  --data '{"archived":true}'
opencode api session.get --param "sessionID=$SESSION_ID"
```

Restore, only after an explicit restore request:

```sh
SESSION_ID='<exact-session-id>'
opencode api session.get --param "sessionID=$SESSION_ID"
opencode api session.update --param "sessionID=$SESSION_ID" \
  --data '{"archived":false}'
opencode api session.get --param "sessionID=$SESSION_ID"
```

Both updates return HTTP 204. Verify the final `data.id` and
`location.directory` against the inspected session. After archive,
`data.time.archived` must be present; after restore it must be absent. Repeating
the current boolean state is a no-op. Send only `archived`, not `time`,
`time.archived`, `time_archived`, or unrelated mutable fields. Do not assume the
update applies to descendants.

For a session missing from the live API, inspect the applicable legacy archive
workflow in [ARCHIVE.md](ARCHIVE.md). A native archive flag and a saved
transcript are different; neither a successful HTTP update nor a missing live
session proves that a file-backed archive was created.
