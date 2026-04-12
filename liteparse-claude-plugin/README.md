# LiteParse plugin for Claude Code, Cowork & Codex

Local document parsing via [LiteParse](https://github.com/run-llama/liteparse), packaged as a plugin for Claude Code, Claude Cowork, and OpenAI Codex. Repo name: **`liteparse-claude-codexplugin`**.

## What you get

Four skills:

- **`liteparse`** — background knowledge skill (not user-invocable). Tells Claude how to choose between `lit` and `npx`, which dependencies each file type needs, and which CLI flags exist.
- **`/liteparse:parse-document <file> [flags]`** — parse a single PDF, DOCX, XLSX, PPTX, image, or scan into text or JSON.
- **`/liteparse:batch-parse <input-dir> [output-dir] [flags]`** — walk a directory and parse every supported file.
- **`/liteparse:screenshot-document <file.pdf> [output-dir] [flags]`** — render PDF pages as PNG or JPG.

Plus a sample config at `examples/liteparse.config.json` (referenced from the skills via relative links).

## Requirements

- Node.js 18+ and `npm` (for the `npx` fallback).
- Optional: `lit` installed globally: `npm i -g @llamaindex/liteparse`.
- Optional: LibreOffice for Office documents (`docx`, `xlsx`, `pptx`, `odt`, …).
- Optional: ImageMagick (`magick` or `convert`) for image inputs (`png`, `jpg`, `tiff`, `webp`, `svg`, …).

If `lit` is not on `PATH`, the skills fall back to `npx -y @llamaindex/liteparse`.

## Install

### Claude Code

```bash
claude plugin marketplace add cyanxxy/liteparse-claude-codexplugin
claude plugin install liteparse@local-liteparse
```

### Claude Cowork

Cowork currently documents two install paths for custom plugins. Pick one:

1. **Add a marketplace from GitHub.** Push this repo to GitHub, then in Cowork's plugin marketplace use "Add marketplace" and point it at `cyanxxy/liteparse-claude-codexplugin`. `liteparse` appears in the listing for install.
2. **Upload a custom plugin file.** Package the `liteparse-claude-plugin/` directory and install it through Cowork's custom-plugin upload flow.

Adding a local filesystem marketplace the way Claude Code does above (`claude plugin marketplace add /path`) is **not** a documented Cowork install path — use one of the two flows above instead.

## Structure

```
liteparse-claude-plugin/
├── .claude-plugin/plugin.json
├── LICENSE
├── README.md
├── examples/liteparse.config.json
└── skills/
    ├── liteparse/SKILL.md              # reference knowledge
    ├── parse-document/SKILL.md         # /liteparse:parse-document
    ├── batch-parse/SKILL.md            # /liteparse:batch-parse
    └── screenshot-document/SKILL.md    # /liteparse:screenshot-document
```

## Upstream

- Repo: https://github.com/run-llama/liteparse
- Docs: https://developers.llamaindex.ai/liteparse/
- License: Apache-2.0 (matches this plugin).
