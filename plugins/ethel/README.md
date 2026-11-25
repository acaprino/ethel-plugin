# Ethel for Claude Code

A Claude Code plugin that connects Claude Code to the MCP server of your Ethel service, so you can read and change
your tasks from it.

**Status:** written against the design of the MCP server; to be verified against a running server. The design is
`docs/specs/2026-11-23-mcp-server-design.md`. Until that check is done, the tool fields the skill lists, the shape
of `.mcp.json` and what the plugin does when the two values it asks for were never given are unverified.

## What you need

- An Ethel server that offers MCP: the setting `SYNC_MCP` is on. The stock server has it off by default; the
  enterprise server has it on by default. With it off, `/mcp` answers 404.
- A key, created in the Ethel app: Settings, «Connect an assistant». It is shown once.
- Claude Code.

## Install

Type these inside a Claude Code session, not in a shell:

```text
/plugin marketplace add acaprino/ethel-plugin
/plugin install ethel@ethel
```

The install asks for two values. A shell `claude plugin install` does not ask for them, so the server would have
no address and no key.

| Asked | Value |
|---|---|
| Address of your Ethel server | Without `/mcp` and without a trailing slash, for example `https://ethel.example.org` |
| Ethel key | The key from the app (it starts with `mcp_`). Claude Code keeps it in secure storage |

The plugin's files hold no address and no key, and it reads no environment variable. Never paste the key into a
chat; if you did, revoke it.

Then type `/mcp`: the server `ethel` is listed with 24 tools. `/ethel:ethel-connect` walks through these steps and
the common failures.

Update:

```text
/plugin marketplace update ethel
```

## What is in the plugin

| Part | What it does |
|---|---|
| `plugin.json` | Declares the two values the install asks for (`userConfig`: `server_url`, and `mcp_key` marked sensitive) |
| `.mcp.json` | Registers the MCP server `ethel` at the address you gave followed by `/mcp`, with the key as a `Bearer` token in the `Authorization` header |
| Skill `ethel` | Loaded when you ask about your tasks, plans, reminders, inbox, projects or Ethel. It tells Claude which of the 24 tools does which job and with which fields, to read before it writes, to ask before it deletes, and to keep the `undoId` of each change |
| `/ethel:ethel-connect` | Guides the setup: the server's switch, the key, the install that asks for the address and the key, the check with `/mcp`, how to revoke a key |
| `/ethel:ethel-today` | Reads today's tasks and the late ones and gives a short plan: what to do first and why, what can wait. It changes nothing |
| `/ethel:ethel-capture <text>` | Creates the tasks a free text contains, with the fields the text states (due date, priority, place, reminder, repeat), shows what it created and offers to undo |

## Before you use it

- **Claude can do what you can do, deleting included.** The server gives a key the whole of your task list. There
  is one level of access, not a read-only one.
- **Text in your tasks comes from outside.** A title, a note, a comment, an attachment or an imported list can
  contain text written to make an assistant act. The skill tells Claude to treat it as data and to tell you, but
  an instruction to a model is not a lock.
- **Keep the approval prompt on for the destructive tools:** `delete_task`, `delete_subtask`, `delete_comment`,
  `remove_attachment`. Do not add them to an allow list.
- **Changes can be undone for ten minutes.** Each change answers with an `undoId` that the `undo` tool accepts
  once, until ten minutes pass or the server restarts.
- **The content of the tasks Claude reads goes to the provider of the model.**
- **A file is at most 5 MiB**, as everywhere in Ethel, and travels as base64 inside the conversation.

## What it does not do

Deleting areas, folders, projects or sections, sticky notes, the Pomodoro timer, import and export and voice
capture have no tool in this version of the server: use the Ethel app.

## Revoke a key

In the app: Settings, «Connect an assistant», «Revoke». The next call with that key is refused; your other keys
and the app keep working. To use a new key, give it to the plugin: its settings under `/plugin`, or uninstall and
install it again from inside Claude Code.
