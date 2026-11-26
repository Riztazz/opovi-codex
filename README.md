# opovi-codex

The Opowi plugin for Codex. A person sends AI issues from their Opowi world, then invokes a fetch skill in Codex,
and Codex serves them with the world's own skills. Nothing waits and nothing wakes.

Status: built from opovi-agent-kit 0.2.0. Not yet tried against a live Opowi server.

## Install

Codex plugins have no settings of their own, so the token reaches Codex from an environment variable.

1. Make an agent token in Opowi, under Settings, You, Agent connection (it acts for 90 days).
2. Set it as `OPOVI_TOKEN` in your user environment, then start a new terminal so Codex sees it:

   - **Windows (PowerShell, persistent for your user):**
     ```powershell
     [Environment]::SetEnvironmentVariable("OPOVI_TOKEN", "<your-token>", "User")
     ```
   - **macOS (zsh):**
     ```zsh
     echo 'export OPOVI_TOKEN="<your-token>"' >> ~/.zshrc && source ~/.zshrc
     ```
   - **Linux (bash):**
     ```bash
     echo 'export OPOVI_TOKEN="<your-token>"' >> ~/.bashrc && source ~/.bashrc
     ```

3. Add the server from `config.toml.example` to `~/.codex/config.toml`, replacing `<your Opowi install>` with your
   Opowi address (such as `opowi.example.com`, with no trailing slash before `/api`):

   ```toml
   [mcp_servers.opovi]
   url = "https://<your Opowi install>/api/plugins/ai-handoff/mcp"
   bearer_token_env_var = "OPOVI_TOKEN"
   ```

   `bearer_token_env_var` sends `Authorization: Bearer <token>` from `OPOVI_TOKEN` on every MCP call. (If your setup
   uses `env_http_headers` instead, map `"Authorization"` to an environment variable holding `"Bearer <token>"`.)

4. Install the plugin from https://github.com/Riztazz/opovi-codex (or copy `skills/fetch-list`, `skills/fetch-single`, and `skills/fetch-all`
   into `~/.agents/skills/`).

Then, in Opowi, open the world's Agent connection panel and save the world's harness for Codex into a folder of its
own. Open Codex in that folder and invoke a skill.

To update: pull the latest `opovi-codex` repository and start Codex again. An Opowi server that needs a newer plugin
says so on the first claim (`ai-handoff.kit-outdated`).

## Commands

| Skill | What Codex does |
|---|---|
| `$fetch-list` | Shows this world's open issues, numbered oldest first. Serves none. |
| `$fetch-single <id or number>` | Serves one issue, by id or by its number in the list. With none, the oldest open one. |
| `$fetch-all` | Serves every open issue, oldest first, one proposal each, then ends the turn. |

Arguments to `$fetch-single` are written directly after `$fetch-single` in your message (for example `$fetch-single 2`).
Each proposal waits in Opowi until you accept it. A folder with no `opovi.json` gets the steps to save one, and a
folder whose harness is older than the world's gets the panel's address, with nothing claimed.

## Layout

```
.codex-plugin/plugin.json         name "opovi", version (= KIT_VERSION, set by the kit), skills "./skills/"
config.toml.example               the [mcp_servers.opovi] entry for ~/.codex/config.toml
skills/fetch-list/SKILL.md        written by opovi-agent-kit's sync.py
skills/fetch-single/SKILL.md      reads the id or number written after $fetch-single
skills/fetch-all/SKILL.md
KIT_VERSION                       the kit version these skills came from, sent to claim as kitVersion
```

Reading the world folder (`opovi.json`, `roles/<role id>.md`, `.agents/skills/<name>/SKILL.md`) uses Codex's workspace
file tools. The skills instruct Codex never to run shell commands or edit files while serving issues.

## What it may do

- It calls four Opowi tools, all on the Opowi server: `list_issues`, `claim`, `read_project`, `submit_proposal`.
- It holds no hook and no script.
- It reads only the issues of the token's own person, and changes nothing until that person accepts a proposal.

## Built from

The skills, `KIT_VERSION` and `.codex-plugin/plugin.json`'s `version` come from `opovi-agent-kit`. Change them there
and run its `sync.py`, never here by hand. `py -3 sync.py --check` in the kit fails when this repository differs.

## References

- Codex MCP: https://learn.chatgpt.com/docs/extend/mcp?surface=cli
- Codex authentication: https://learn.chatgpt.com/docs/auth
