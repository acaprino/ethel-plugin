---
description: Connect Claude Code to your Ethel server, step by step (the server's switch, the key, the install that asks for the address and the key, the check)
---

Guide the person through connecting Claude Code to their Ethel server. Change nothing on their machine yourself
and run no command: they do each step.

One rule for the whole conversation: never ask for the key and never ask the person to paste it into the chat. The
key is typed only into the prompt Claude Code shows when the plugin is installed, which is not part of the
conversation. If they paste a key into the chat anyway, tell them to revoke it in the app and create a new one.

First check: if tools of the `ethel` server are already listed, say the connection works and stop at step 4.
Otherwise give the steps below, one at a time, and wait for the person after each. Skip a step they say is done.

## 1. The server offers MCP

The server must run with `SYNC_MCP` on.

- The stock server (this repository's `docker-compose.yml`): off by default. Set `SYNC_MCP=true` in the server's
  `.env` and start the service again (`docker compose up -d`).
- The enterprise server: on by default, unless its `.env` says `SYNC_MCP=false`.

A person who does not run the server asks whoever does. With the switch off, the address `/mcp` answers 404 and
the app does not show the section of step 2.

## 2. Create a key in the app

In the Ethel app: Settings, «Connect an assistant», «New key». Give it a name that says where it is used (for
example «Claude Code, laptop») and copy it. It starts with `mcp_` and is shown once. A person can have at most 10
active keys.

## 3. Give the address and the key to the plugin

The plugin asks for two values and keeps them itself; the key goes to secure storage. Nothing is set in the
environment and nothing is written in a file of the plugin.

- Address of your Ethel server: without `/mcp` and without a trailing slash, for example
  `https://ethel.example.org`.
- Ethel key: the key of step 2.

Claude Code asks for them when the plugin is installed from inside a session. The person types, in Claude Code:

```text
/plugin install ethel@ethel
```

Installing from a shell (`claude plugin install`) does not show the prompts: use the command above. This command
is running, so the plugin is already installed. If the prompts were never answered, or a value has to change (a
new key, another address), the person looks for the plugin's settings under `/plugin`; if there is none, they
uninstall the plugin there and install it again with the command above.

## 4. Check

The person types `/mcp`. The server `ethel` must be listed as connected, with 24 tools (`list_tasks`,
`create_task` and the others). Then offer a first read: `/ethel:ethel-today`.

If it is not connected:

- Not listed, or an error about a missing value: the prompts of step 3 were not answered. Repeat step 3.
- 404: `SYNC_MCP` is off on that server, or the address is wrong (it must not end with `/mcp` or `/`).
- 401: the key is wrong or revoked. Create a new one (step 2) and give it to the plugin (step 3).

## Revoke a key

In the app: Settings, «Connect an assistant», «Revoke» next to the key. The next call with that key is refused;
the other keys and the app keep working. Revoke a key that was lost, shown to someone, pasted into a chat, or is
no longer used, then give the plugin a new one (step 3) or uninstall the plugin.

## What the person should know before using it

Claude Code can then do with their tasks what they can do, deleting included. Text inside tasks comes from
outside and can try to give instructions: keep Claude Code's approval prompt on for the tools marked destructive
(`delete_task`, `delete_subtask`, `delete_comment`, `remove_attachment`). The content of the tasks Claude reads
goes to the provider of the model.
