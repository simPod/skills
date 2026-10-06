# Session Archive Plugin

The optional `opencode-plugin-sessions` plugin archives a session family by
saving verified JSON transcripts, then deleting the live sessions. Restore
imports the original IDs, parents first. This is not the native archive flag and
not a complete runtime or project backup.

## Use the Desktop or TUI Commands

1. Confirm that the plugin is enabled on the connected server. For terminal use,
   also confirm that its TUI companion is loaded. Do not install it or change
   global configuration merely to answer a session lookup request.
2. Resolve the exact session ID using this skill. Explain that archiving deletes
   the selected session and its descendants after verification.
3. Ask the user to run `/session-archive <exact-session-id>` and confirm
   exclusive use in the desktop question panel or TUI dialog. `/session-archive`
   without an ID uses the currently open chat, which may not be the chat being
   discussed.
4. Use `/session-archives` to browse the current project's saved transcripts, or
   `/session-unarchive <archive-uuid>` to restore a specific archive after its
   confirmation. An archive UUID is not a session ID.

These are native commands, not prompt templates. Desktop commands are registered
on the server and can use `session.command`, but require the user's answer in a
native question panel. Do not answer that confirmation for the user or claim a
pending command succeeded. TUI commands run through the terminal keymap. Neither
path invokes a model. Verify the reported archive UUID or restored root ID
before reporting success; the desktop cannot report to a session it just
deleted.

Desktop commands require the hosting server's managed-service registration and
verify the exact plugin instance before session access. Standalone or embedded
hosts without a matching registration are refused. Desktop commands cannot be
queued. After archiving the open chat, open another session from the sidebar.
Restoration returns sessions to the list but does not navigate the desktop.

## Limits and Permissions

- OpenCode V2 has no atomic export-and-delete or session lock. Another client
  can add work after the final check. Require exclusive use and disclose this
  remaining risk; do not describe the archive as lossless.
- The plugin refuses running, queued, incomplete, forked, reverted, or
  unsupported cross-project session families. Do not bypass those refusals.
- Existing session IDs prevent restoration. Keep the archive after partial
  imports or deletion errors. Do not delete existing sessions to retry.
- Files are private but unencrypted and can contain secrets. Configurable
  `storageDirectory` is on the server and defaults to
  `~/.opencode-session-archives`; project folders use stable keys and retain
  original working directories. Do not print transcript contents or publish
  archive files.

When the plugin is absent, explain that the public V2 API has export/import but
no native archive operation. Exporting a transcript does not itself hide a
session. Do not replace the guarded plugin with manual recursive deletion unless
the user explicitly requests a separate destructive recovery procedure.
