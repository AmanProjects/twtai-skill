# TWTAI Skills — a sample Claude Code marketplace

A complete, working **reference marketplace** for the Tech Writer's Tribe (TWT) AI program. It exists to show participants *how the pieces fit together* — a marketplace that contains a plugin, and a plugin that bundles a command, a skill, an agent, a hook, and (remote) MCP servers — plus a standalone script and a sample doc to test on.

Clone it, read it, install it, then build your own the same way.

## The hierarchy at a glance

```
twtai-skill/                          ← THIS REPO = a marketplace
├── .claude-plugin/
│   └── marketplace.json              ← the catalog (lists plugins)
├── plugins/
│   └── doc-skills/                    ← a PLUGIN (the installable bundle)
│       ├── .claude-plugin/plugin.json ← plugin manifest
│       ├── commands/                  ← COMMANDS  (/doc-skills:check-style)
│       │   ├── check-style.md
│       │   └── release-notes.md
│       ├── skills/                    ← SKILL     (auto-invoked SKILL.md)
│       │   └── doc-coverage/SKILL.md
│       ├── agents/                    ← AGENT     (delegated subagent)
│       │   └── doc-auditor.md
│       ├── hooks/                     ← HOOK      (event handler)
│       │   └── hooks.json
│       └── .mcp.json                  ← MCP SERVERS (remote, no setup)
│   └── docs-lsp/                      ← a second PLUGIN
│       ├── .claude-plugin/plugin.json ← plugin manifest
│       ├── .lsp.json                  ← LSP SERVER (code intelligence)
│       ├── server.js                  ← the language server itself
│       ├── scan.js                    ← batch mode, CI-friendly
│       └── commands/check-links.md    ← COMMAND (/docs-lsp:check-links)
├── examples/
│   └── remote-mcp/                    ← connect to an online MCP server (no plugin)
│       ├── .mcp.json
│       └── README.md
├── scripts/
│   └── check-links.mjs                ← a simple standalone SCRIPT
└── sample-docs/
    └── orders-api.md                  ← a flawed doc to test everything on
```

> **Marketplace ≠ plugin.** The marketplace is the *catalog*; the plugin is the *thing you install*. One marketplace can list many plugins.

## Use it (as an individual)

```
/plugin marketplace add AmanProjects/twtai-skill   ← add the catalog
/plugin install doc-skills@twtai                    ← install a plugin
/plugin install docs-lsp@twtai                      ← ...and the other one
```

`twtai` is the marketplace `name` from `marketplace.json` (not the repo name). After installing, these are available:

| Component | How you use it |
|-----------|----------------|
| Command | `/doc-skills:check-style sample-docs/orders-api.md` |
| Command | `/doc-skills:release-notes <changelog or git log>` |
| Skill | `doc-coverage` — auto-invoked when you ask "what's missing in this doc?" |
| Agent | `doc-auditor` — delegated for a full doc-set audit |
| MCP (remote) | Ask: "Using DeepWiki, summarize the architecture of repo X" |

From `docs-lsp`:

| Component | How you use it |
|-----------|----------------|
| LSP server | Automatic. Claude gets go-to-definition on links, backlinks on headings, and live broken-link diagnostics while it edits Markdown. |
| Command | `/docs-lsp:check-links ./docs` — the same engine in batch mode |

> `docs-lsp` is the only plugin here that ships an **LSP server**, and the only one with no external dependency at all — the language server is plain Node, bundled in the plugin. See [`plugins/docs-lsp/README.md`](plugins/docs-lsp/README.md).

## The MCP servers are remote and safe

This repo deliberately uses **online, hosted** MCP servers so there is nothing to install or run — and both are **read-only with no login or API key**, which makes them safe for a classroom:

| Server | URL | What it does |
|--------|-----|--------------|
| **DeepWiki** (Cognition/Devin) | `https://mcp.deepwiki.com/mcp` | Ask questions about any public GitHub repo's docs |
| **Claude Code Docs** (Anthropic) | `https://code.claude.com/docs/mcp` | Search the official Claude Code documentation |

Installing the `doc-skills` plugin connects them automatically. To connect **without** the plugin, see [`examples/remote-mcp/`](examples/remote-mcp/) — it shows both the one-line `claude mcp add` command and the shareable project `.mcp.json`.

## Try the standalone script (no install needed)

```bash
node scripts/check-links.mjs README.md
```

Dependency-free Markdown link checker (Node 18+). Reports broken http(s) links and missing relative files. (Try it on `README.md` — which has real links — rather than the sample doc, whose only URL sits inside a code block.)

## Build your own marketplace (the 5-minute loop)

1. Create a GitHub repo.
2. Add `.claude-plugin/marketplace.json` at the root.
3. Add a plugin folder with `.claude-plugin/plugin.json` and your skills in `commands/`.
4. Commit and push.
5. Share two lines: `/plugin marketplace add you/your-repo` and `/plugin install your-plugin@your-marketplace`.

Covered in **Module 4** of the TWT program. This repo is the worked example.
