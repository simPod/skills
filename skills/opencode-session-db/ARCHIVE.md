# Session Archive Plugin

The optional `opencode-plugin-sessions` plugin archives a session family by
saving verified JSON transcripts, then deleting the live sessions. Restore
imports the original IDs, parents first. This is not the native archive flag and
not a complete runtime or project backup.

The family can span projects. Its complete archive stays in the root project's
folder, retaining each descendant's project ID and directory. Do not skip
foreign-project descendants: native deletion removes them too. That root archive
can contain other projects' private transcripts and permissions.

## Use the Desktop or TUI Commands

1. Confirm that the plugin is enabled on the connected server. For terminal use,
   also confirm that its TUI companion is loaded. Do not install it or change
   global configuration merely to answer a session lookup request.
2. For archiving, resolve the exact live session ID using this skill. Explain
   that archiving deletes the selected session and its descendants after
   verification.
3. Run or ask the user to run `/session-archive <exact-session-id>` only after
   the user clearly requests archiving that target. There is no confirmation
   dialog. `/session-archive` without an ID uses the currently open chat, which
   may not be the chat being discussed.
4. Only after an explicit restore request, use the command
   `/session-restore <exact-session-id>` to find an archive containing that root or
   descendant ID. The live API need not contain the deleted session. Lookup
   searches only archives available to the current project, including explicitly
   mapped source archives; it does not search all projects. One match restores
   immediately. Multiple matches open a picker limited to those archives. No
   match or a cancelled picker imports nothing. Restoring by a descendant ID
   restores the whole saved family, including its root, not only that descendant.
5. Use `/session-restore` without an ID to browse the current project's saved
   transcripts. `/session-restore <archive-uuid>` remains supported to restore
   a specific archive immediately. Selecting an archive restores it without
   another confirmation. The old `/session-unarchive` and `/session-archives`
   commands are no longer registered.

These are native commands, not prompt templates. Desktop commands are registered
on the server and can use `session.command`. Archive and restore run without
confirmation; only archive selection and completed-result panels use native
questions. Do not treat a lookup request as permission to archive or restore, or
claim a pending command succeeded. TUI commands run through the terminal keymap.
Neither path invokes a model. Verify the reported archive UUID or restored root
ID before reporting success; the desktop cannot report to a session it just
deleted.

Desktop commands require the hosting server's managed-service registration and
verify the exact plugin instance before session access. Standalone or embedded
hosts without a matching registration are refused. Desktop commands cannot be
queued. After archiving the open chat, open another session from the sidebar.
Restoration returns sessions to the list but does not navigate the desktop.

## Limits and Permissions

- OpenCode V2 has no atomic export-and-delete or session lock. Another client
  can add work after the final check. Other clients and automations must stop
  writing to the tree before archiving. Disclose this remaining risk unless the
  user has already accepted it; do not describe the archive as lossless.
- The plugin refuses running, queued, incomplete, forked, reverted, or visibly
  workspace-linked session families. Do not bypass those refusals.
- Restoration checks that all saved directories exist and are readable on the
  server, and resolve to their original project IDs or explicit mapped
  destinations before importing. Project metadata can be cached; imported
  project IDs and locations are also verified. Recreate removed worktrees,
  restore their original project identity, or configure an explicit restore
  mapping before retrying. Do not silently remap project IDs.
- Optional server plugin `restoreMappings` match an exact
  `from: { projectID, directory }` pair to a `to: { projectID, directory }`
  pair. Both directories must be absolute server paths. Configure them only
  after an explicit request to relocate matching sessions. Rules apply to later
  restores until removed, with no prefix matching or chaining. Desktop and TUI
  use the same server rules.
- Mapped root archives appear in the destination picker only for an exact
  authorized root pair. Archive files are not moved or rewritten. Session IDs,
  parent links, and transcripts remain unchanged; imported project IDs and
  directories intentionally change. Paths inside messages, metadata, and
  permission rules are not rewritten. Review saved permissions before resuming a
  relocated session.
- V2.0.24's public HTTP API strips workspace IDs and cannot restore them.
  Visible workspace IDs are refused, but HTTP cannot detect every
  workspace-linked live session. Do not claim workspace identity restoration.
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
