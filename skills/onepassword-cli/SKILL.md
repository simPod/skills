---
name: onepassword-cli
description:
  Use 1Password CLI safely for op:// secret references, op run, Blackfire
  credentials, and environment injection. Use when commands need 1Password
  secrets or when op whoami authentication differs from op run.
---

# 1Password CLI

Use `op run` to inject `op://` secret references into the command that needs
them. Do not print, persist, or pass resolved secret values through shell
output.

## Authentication

Do not use `op whoami` as a required preflight for `op run`.

In this environment, `op whoami` can report that no account is signed in while
`op run` still resolves secret references through the 1Password desktop
integration. Test the actual required capability instead.

```sh
REQUIRED_SECRET='op://Vault/Item/field' \
op run -- sh -lc 'test -n "$REQUIRED_SECRET"'
```

The command succeeds only when the reference resolves. It does not reveal the
secret value.

## Secret Commands

- Define each secret as an `op://` reference immediately before `op run`.
- Use `op run -- <command>` so only the child process receives the secret.
- Pass needed values to Docker with `-e NAME`, not `-e NAME=value`.
- Do not use `op read` when the value could appear in output.
- Do not add secrets to source files, dotenv files, commits, logs, or reports.

Example for a Blackfire process:

```sh
BLACKFIRE_CLIENT_TOKEN='op://FLOP/Blackfire Creds/Blackfire/client token' \
op run -- docker run --rm -e BLACKFIRE_CLIENT_TOKEN image command
```
