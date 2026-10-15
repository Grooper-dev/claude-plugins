# Grooper plugins for Claude

Official [Claude plugin](https://code.claude.com/docs/en/plugins) from
Business Imaging Systems, Inc. (BIS). **Grooper is your harness for any kind
of data** — documents, images, JSON, EDI, email, databases — with whatever
processes and loops you build around it. Every repository can host as many
**MCP servers** as you need — curated toolsets or fully custom. Point Claude
at any one of them and just talk.

## Install

```text
/plugin marketplace add Grooper-dev/claude-plugins
/plugin install grooper@grooper-dev
```

Claude Code prompts you once for the address of the **Grooper MCP server**
you want to connect to. A repository's built-in server is:

```
https://your-server/GWS/{repository}/mcp
```

`{repository}` is the repository's name or its GUID — e.g.
`https://your-server/GWS/AcmeDocs/mcp`.

If your repository defines its own **MCP Server** nodes — Grooper's way of
controlling which tools an MCP client may see — each one has its own
address:

```
https://your-server/GWS/{repository}/mcp/servers/{serverName}
```

To see which servers a repository has, open
`https://your-server/GWS/{repository}/mcp/servers` in a browser.

One plugin install connects Claude to one Grooper server at a time; change
the address any time from `/plugin` → Configure.

`/mcp` should then show `plugin:grooper:mcp` connected. You get whatever
tools that server exposes — the built-in server has over 100, for reading
configuration, building it, running services and jobs, and searching
processed content. The plugin teaches Claude to hand work to three roles,
so each job runs with only the permissions it needs:

| Ask for… | Role Claude uses | Permissions |
| --- | --- | --- |
| "Map every Content Model in this project" | **grooper-explorer** | reads only — every mutating tool is blocked in its config |
| "Restart the parse service and clear the stuck jobs" | **grooper-operator** | runtime actions: services, jobs, batches, schedules, review queue, logs |
| "Add a DataModel to this document type and fix the script" | **grooper-builder** | config changes: nodes, scripts, overrides, extraction tests |

Claude picks the role from your wording, or you can name it: *"use the
grooper-explorer agent to inventory the projects."* The explorer researches
in its own context window and reports back with node paths, types, and
RowIds — your main conversation stays clean.

Claude Code also prompts for permission before every operator/builder
action; your GWS server is the final authority on what the connection may
do.

## Where it runs

Plugin components load differently on each Claude surface
([platform matrix](https://claude.com/docs/plugins/platform-support)):

| Component | Claude chat | Claude desktop (Cowork) | Claude Code |
| --- | --- | --- | --- |
| Navigation skill | ✅ | ✅ | ✅ |
| The three agents | — | ✅ | ✅ |
| MCP connection | via connector | via connector | ✅ (prompts for your URL) |

Claude Code prompts for your endpoint at install. **Desktop and chat don't
prompt** — there, install a copy of this folder with the URL hard-pinned in
`plugins/grooper/.mcp.json`: replace `${user_config.grooper_mcp_url}` with
your real endpoint URL, zip the folder, and use **Customize → Plugins → Add
→ Upload plugin**. Two minutes, full skill + agent + connection.

## Security

- **Tool fences are real, not prompts.** The explorer agent's frontmatter
  denies every mutating tool by name (`Delete_Node`, `Update_Node`,
  `Start_Service`, …); the builder can't touch runtime operations and the
  operator can't edit configuration. Claude Code's permission prompts gate
  each call on top of that.
- To hard-block writes repo-wide regardless of agent, add to your project's
  `.claude/settings.json`:

  ```json
  { "permissions": { "deny": [
    "mcp__plugin_grooper_mcp__Delete_Node",
    "mcp__plugin_grooper_mcp__Update_Node",
    "mcp__plugin_grooper_mcp__Add_Child_Nodes",
    "mcp__plugin_grooper_mcp__Clear_Child_Nodes",
    "mcp__plugin_grooper_mcp__Copy_Or_Move_Node"
  ] } }
  ```

- **Claude Code runs on your machine**, so intranet/VPN-hosted GWS servers
  work. (Claude on the web cannot reach servers that are not
  internet-exposed.)
- **Internal certificates work.** Current Claude Code (native installer,
  v2.1.101+) trusts your Windows certificate store, so an internal CA
  distributed by Active Directory is trusted automatically. For npm-based
  installs or machines missing the root, point one environment variable at
  your internal root CA — directly or via `.claude/settings.json`:

  ```json
  { "env": { "NODE_EXTRA_CA_CERTS": "C:\\certs\\your-internal-root-ca.pem" } }
  ```

  Plain `http://` on a trusted network also works; Claude Code places no
  HTTPS requirement on MCP servers it reaches locally.
- v1 supports anonymous or bearer-token endpoints; GWS OAuth configuration
  docs are coming. Credentials configured here stay in Claude Code's local
  credential store.

## Updating

```text
claude plugin update grooper@grooper-dev
```

Marketplace auto-update is off by default for third-party marketplaces;
enable it per-marketplace from `/plugin` → Marketplaces.

## Development

```powershell
claude plugin validate ./plugins/grooper             # before pushing
claude --plugin-dir ./plugins/grooper                # local test loop
```

`--strict` is green on Claude Code ≥ 2.1.281; older versions warn about the
directory listing fields (`icon`, `termsOfServiceUrl`, …) that they ignore
at load time.

Changes to `agents/`, `.mcp.json`, or hooks need `/reload-plugins` in a
running session; `SKILL.md` edits are hot-reloaded.

## License

Licensed under the Grooper End User License Agreement —
https://grooper.com/EULA/. Copyright © Business Imaging Systems, Inc.
See [LICENSE](LICENSE).

Docs: [help.grooper.com](https://help.grooper.com) ·
[wiki.grooper.com](https://wiki.grooper.com) ·
[learn.grooper.com](https://learn.grooper.com) ·
[support](https://grooper.com/support/) — support@bisok.com ·
1.800.662.0777 · info@grooper.com
