<div align="center">

# MCP LLM Browser

### One tiny browser that reads the web as clean Markdown — for humans **and** AI agents.

It fetches real pages over trusted HTTPS, turns the HTML into **clean Markdown**, and lets you use it
**two ways**: as an **interactive terminal browser**, or as a native **MCP** server an AI agent drives.

**No Chromium. No Node. No Playwright. No libraries at all** — a single **289 KB** static binary.

![size](https://img.shields.io/badge/binary-289%20KB-brightgreen)
![deps](https://img.shields.io/badge/dependencies-0-blue)
![ram](https://img.shields.io/badge/peak%20RAM-264%20KiB-orange)
![https](https://img.shields.io/badge/HTTPS-verified-red)
![mcp](https://img.shields.io/badge/MCP-native-purple)
![platform](https://img.shields.io/badge/Linux-x86--64-lightgrey)

</div>

---

## What it is

A **web browser whose native format is Markdown.** Point it at a page and it gives you the *content* —
headings, text, tables, links, forms — as clean Markdown, with none of the ad markup, tracking
scripts, or layout noise. The exact same engine serves two front-ends:

- 🧑 **Terminal** — an interactive browser you drive from the shell. Navigate by numbered links, go
  back, reload, search, use tabs and favorites, fill and submit forms. It renders the page nicely in
  your terminal, and `m` shows you the raw clean Markdown anytime.
- 🤖 **MCP** — the same browser as a Model Context Protocol server (`web --mcp`). An AI agent in
  Claude Code, Claude Desktop, VS Code, etc. drives it as a tool and gets each page back as Markdown.

> **One engine. Two front-ends. Zero dependencies.**

---

## Why it beats driving a headless Chrome

Today, giving an AI agent "a browser" usually means running a **full Chromium** through
Playwright/Puppeteer/CDP, then feeding the model a **giant DOM dump** (100k+ tokens) or a
**screenshot** it has to visually parse. Heavy, slow, expensive per step, and brittle.

This gives the model **exactly what it needs** — clean, structured, token-light Markdown and
**stable numbered links** — from a single static binary registered as one MCP tool.

| | **MCP LLM Browser** | Headless Chrome + Playwright |
|---|---|---|
| **Install size** | **289 KB**, one static file | hundreds of MB (Chromium + Node + driver) |
| **Dependencies** | **0** — `ldd` says *"not a dynamic executable"* | Node, a browser, driver libs |
| **Peak RAM** | **264 KiB** to fetch + render a 560 KB page | 100s of MB per session |
| **Cold start → page** | **~0.7–1.0 s** incl. DNS + TLS handshake + render | seconds to launch the browser alone |
| **What the model receives** | clean **Markdown** | full DOM (often 100k+ tokens) or a screenshot |
| **Acting on the page** | **numbered** links & fields → `follow link 3` | CSS/XPath selectors or vision "click x,y" |
| **HTTPS trust** | **verified** — chain, hostname, validity checked | delegated to Chromium |
| **Human + agent** | **the same browser** does both | separate tooling |

### The token story, on a real page

Fetching `en.wikipedia.org/wiki/Assembly_language`:

| The model would ingest… | Size | ≈ tokens* |
|---|---:|---:|
| Raw HTML (a naive DOM dump) | 511 KB | ~128,000 |
| **This browser's Markdown** | **156 KB** | **~39,000** |

**~70% fewer bytes — 3.3× smaller** — while keeping the text, links, and forms. Cheaper per step,
and nothing to visually parse. <sub>*rough ÷4 bytes→tokens estimate.</sub>

### Tiny *and* fast because there's nothing underneath it

- **289 KB** of statically-linked machine code — no runtime to boot, no libraries to load.
- **264 KiB** peak resident memory even on a half-megabyte page — it stays well under 2 MiB.
- **Zero dependencies**: DNS, HTTP, trusted HTTPS, and HTML→Markdown are all built in — nothing to
  install, nothing to update, no CVE surface from a dependency tree.

---

## Download

Grab the browser from [**Releases**](../../releases/latest):

| File | What it is |
|------|------------|
| `web` | The browser — interactive terminal UI **and** the MCP server |
| `QUICKSTART.txt` | One-page command cheat-sheet |

```sh
chmod +x web
./web --version
```

That's the whole install. No unpacking, no runtime, no dependencies.

> ⚠️ **Linux x86-64 binary.** On Windows run it under **WSL2**; on macOS in a Linux VM / Docker.

---

## Use it — as a human (terminal)

```sh
./web https://en.wikipedia.org/wiki/Assembly_language
```

At the `web>` prompt:

```
  N              follow link number N
  b   r          back / reload
  s <query>      web search
  m              show the current page as raw, clean Markdown
  form           list forms & fields
  set N <value>  fill field N
  submit [N]     submit form N   (GET or POST, cookies carried)
  f  F [N]       add / list / open a favorite
  t  T  w        new tab / list tabs / close tab
  H  h  q        history / help / quit
```

The page is rendered nicely in your terminal, every link numbered — browsing is just typing the
number. Press **`m`** anytime to see (or copy) the underlying clean Markdown of the page you're on.

---

## Use it — as an AI agent (MCP)

The MCP server **is** `./web --mcp` (JSON-RPC 2.0 over stdio). Point any MCP host at the **absolute
path** of the `web` binary — no wrapper, no Node, no runtime. Every navigation returns the page as
clean **Markdown** (plus its links and forms).

**Claude Code — one command:**

```sh
claude mcp add asm-browser -- /absolute/path/to/web --mcp
claude mcp add -s user asm-browser -- /absolute/path/to/web --mcp   # available in every project
claude mcp list                                                     # verify
```

Then just ask: *"navigate to example.com and read it"*, *"log into this site and open my profile"*.

**Claude Desktop** — `claude_desktop_config.json`:

```json
{ "mcpServers": { "asm-browser": { "command": "/absolute/path/to/web", "args": ["--mcp"] } } }
```

**VS Code** (Copilot agent mode) — `.vscode/mcp.json`:

```json
{ "servers": { "asm-browser": { "type": "stdio", "command": "/absolute/path/to/web", "args": ["--mcp"] } } }
```

The agent gets the **same abilities a human has** — navigate, follow links, search, read, fill &
submit forms (with cookies kept, so it can log in), favorites, tabs, history. The current page is
also exposed as an MCP **resource**.

---

## What it can do

- 🌐 **Fetch real websites** over HTTP **and trusted HTTPS** (redirects, chunked transfer, timeouts).
- 🔒 **Verify the connection** — checks the certificate's chain of trust, hostname, and validity; untrusted → refuses to load.
- 📄 **Clean Markdown** — headings, lists, tables, blockquotes, definition lists, links; no ads, no scripts, no layout noise.
- 🔗 **Numbered links & fields** — every one has a stable index; acting is `follow link 3`, no selectors.
- 🖼️ **Images as links, not pixels** — an `<img>` becomes a followable link (token-light by design).
- 📝 **Forms, POST & cookies** — fill fields, submit GET/POST, keep the session cookie → **log in and stay logged in**.
- 🔎 **Web search**, **favorites**, **tabs**, **history**, **back/reload** — a real browsing session.

---

## Honest limits

- **No JavaScript.** It reads server-rendered HTML; client-only SPAs won't show their content, and JS-driven forms aren't supported.
- **Images are links, not pixels** — by design, to stay token-light and text-first.
- **Linux x86-64 binary** — use WSL2 (Windows) or a Linux VM/Docker (macOS).

---

<div align="center">
<sub>© 2026 — All rights reserved. Binary provided for use; source is not published.</sub>
</div>
