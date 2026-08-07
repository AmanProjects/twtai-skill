# docs-lsp

A **language server for Markdown documentation**, packaged as a Claude Code plugin.

It treats a docs set the way a compiler treats source code: a **heading is a definition**, a **link is a reference**. Everything else falls out of that graph.

```
## Authentication            ← a DEFINITION  (anchor: #authentication)
[auth](./api.md#authentication)  ← a REFERENCE
```

Zero dependencies. Just Node.

---

## Install

```
/plugin marketplace add AmanProjects/twtai-skill
/plugin install docs-lsp@twtai
/reload-plugins
```

Confirm it loaded — `/reload-plugins` should report at least `1 plugin LSP server`. If it doesn't, check the `/plugin` **Errors** tab.

Unlike most LSP plugins, there is **no binary to install separately**. The server is plain Node and ships inside the plugin.

---

## What Claude gets

Once loaded, Claude can navigate your docs instead of grepping them:

| Capability | What it does |
|---|---|
| **Go to definition** | From `[text](./api.md#authentication)` to that heading, in that file, at that line |
| **Find references** | From a heading to **every page that links to it** — backlinks, for free |
| **Document symbols** | The heading outline of a page, nested by level |
| **Workspace symbols** | Search every heading across the whole docs set by name |
| **Completion** | After `](` completes file paths; after `#` completes that file's real anchors |
| **Hover** | Preview a link's target heading and its first two lines |
| **Diagnostics** | Broken links, dead anchors (with "did you mean?"), duplicate headings |

Diagnostics are pushed into Claude's context automatically after each edit — so when Claude breaks a cross-reference while editing, it finds out immediately, with no lint step and no CI round-trip.

The navigation capabilities usually *reduce* context usage: one "find references" replaces a grep plus several speculative file reads.

---

## The `/docs-lsp:check-links` command

The same engine, in batch mode, for when you want a report rather than live editing:

```
/docs-lsp:check-links
/docs-lsp:check-links ./docs
```

You can also run it directly — it exits non-zero on errors, so it works as a CI gate:

```bash
node ~/.claude/plugins/.../docs-lsp/scan.js ./docs
```

---

## What it catches

**Broken links** — a relative link to a file that isn't there.

**Dead anchors** — `[setup](./guide.md#instalation)` when the heading is `## Installation`. Suggests the nearest match.

**Duplicate headings** — the one nobody catches by reading. Two `## Authentication` sections mean the second one's anchor is silently `#authentication-1`, so every link written to `#authentication` lands on the first.

### What it deliberately ignores

Documentation quotes link syntax constantly. All three of these are prose *about* a link, not a link, and none is reported:

- a link inside a fenced code block
- a link inside a single-backtick span
- a single-backtick span nested inside a double-backtick span

That last one is subtle: CommonMark closes a code span with a backtick run of *exactly* the opening length, so a naive regex masks the delimiters and leaves the link exposed in the middle. The server uses a scanner that measures run length instead.

---

## Heads-up: this plugin claims `.md`

`extensionToLanguage` registers `.md` and `.markdown`. When two enabled LSP servers declare the same extension, **the first registered wins and the other never starts** — `/plugin` shows a warning naming the active one.

If you already run `marksman`, `vale-ls`, or another Markdown language server, disable one of them.

---

## Limits

- **Anchor slugs approximate GitHub's rules.** Good for ASCII headings; emoji and leading punctuation can differ from your publishing platform. `slugify()` in `server.js` is the one place to change.
- **Relative links only.** External URLs are skipped, not fetched.
- **Full-document sync.** The client resends the whole file on each change. Fine at documentation scale; not tuned for 10k-line files.

---

## Files

| File | Purpose |
|---|---|
| `server.js` | The whole language server — transport, index, and all seven capabilities |
| `scan.js` | Batch mode: lints a workspace, exits 1 on errors |
| `.lsp.json` | Tells Claude Code how to launch the server |
| `commands/check-links.md` | The `/docs-lsp:check-links` command |

`server.js` is ~460 lines and readable end to end — the LSP wire protocol is JSON-RPC framed with `Content-Length` headers, and the whole thing fits in one file. It is meant to be read, not just installed.
