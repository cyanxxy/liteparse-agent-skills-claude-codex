# LiteParse plugin for Claude Code, Cowork & Codex

Local document parsing via [LiteParse v2](https://github.com/run-llama/liteparse), packaged as a plugin for Claude Code, Claude Cowork, and OpenAI Codex. Repo name: **`liteparse-agent-skills-claude-codex`**.

## Claude slash commands

This package is the Claude plugin surface. For Codex, use the sibling `plugins/liteparse/` package and `$liteparse:...` skill names.

| Command | Description |
|---------|-------------|
| `/liteparse:parse-document <file> [flags]` | Parse a single PDF, DOCX, XLSX, PPTX, image, or scan into text or JSON |
| `/liteparse:batch-parse <input-dir> [output-dir] [flags]` | Parse every supported file in a directory |
| `/liteparse:screenshot-document <file> [output-dir] [flags]` | Render document pages as PNG images |
| `/liteparse:merge-parsed <files...> [-o output]` | Combine parsed outputs from multiple files into one document |
| `/liteparse:compare-documents <file-a> <file-b> [-o diff]` | Parse two documents and produce a structured diff with summary |
| `/liteparse:extract-tables <file> [--csv\|--json] [-o output]` | Extract tables from documents into CSV or structured JSON |
| `/liteparse:extract-structured <file> [--fields ... \| --schema ...] [--json\|--jsonl\|--csv] [-o output]` | Extract user-defined fields into repeatable structured output |
| `/liteparse:convert-format <file> --to <format> [-o output]` | Convert between file formats via LibreOffice (no parsing) |

The background `liteparse` skill loads automatically and provides LiteParse v2 CLI reference, dependency rules, JSON output shape, and failure handling.

## Structured extraction

`/liteparse:extract-structured` is the agent-facing entry point for repeatable field extraction. Users can either:

- pass inline `--fields` to describe what to extract, then let the agent normalize those loose requests into a canonical schema before extraction, or
- pass `--schema <file>` to reuse a saved recipe as the stable automation contract for later Codex, Claude Code, or Claude Cowork runs.

The command uses LiteParse JSON output as its source of truth, then writes structured JSON by default. `--jsonl` and `--csv` are flattened export views for downstream automation pipelines.
Use `--save-schema <file>` when you want to persist the normalized recipe as a reusable contract for later runs.

An example reusable schema lives at [`examples/invoice.extract.json`](examples/invoice.extract.json).

## Requirements

- Node.js 18+ and `npm` (for the `npx` fallback).
- Optional: Python 3.10+ for the Python LiteParse package, Python SDK use, or custom OCR server packages.
- Optional: `lit` installed globally with the current npm (`npm i -g @llamaindex/liteparse@latest`), Python (`pip install "liteparse>=2.0.7"`), or Rust (`cargo install liteparse`) package. Do not use the Homebrew LiteParse formula as the primary v2 path; it has lagged upstream.
- Optional: `@llamaindex/liteparse-wasm` for browser/edge PDF parsing experiments. It has no `lit` CLI and does not cover file-path input, Office conversion, built-in/HTTP OCR, or screenshots.
- Optional: `liteparse-server` or LiteParse Docker images for a host/container parser service when Cowork's VM cannot load the native CLI.
- Optional: LibreOffice for Office documents (`docx`, `xlsx`, `pptx`, `odt`, ...).
- Optional: ImageMagick (`magick` or `convert`) for image inputs (`png`, `jpg`, `tiff`, `webp`, `svg`, ...).
- Optional env vars: `TESSDATA_PREFIX` (offline OCR language packs).

If `lit` is not on `PATH`, the skills fall back to `npx -y @llamaindex/liteparse@latest`.

### Claude Cowork Linux VM note

Claude Cowork shell commands run inside an isolated Linux VM. Older LiteParse native binaries could fail before startup with loader errors such as `GLIBC_2.38 not found`, `GLIBC_2.39 not found`, or `GLIBCXX_3.4.31 not found`. LiteParse 2.0.6+ lowered the Linux GNU binary baseline, so first use the current package (`npx -y @llamaindex/liteparse@latest ...`, `npm i -g @llamaindex/liteparse@latest`, or `pip install "liteparse>=2.0.7"`).

If a loader error still happens, stop and report the runtime incompatibility instead of retrying the same binary. Capture `uname -m`, `ldd --version`, and the exact error. Then try the Python wheel, build from source with `cargo install liteparse` inside the target VM, run LiteParse in a compatible container/server, or parse in a compatible host environment and bring the output files back into Cowork.

For a Cowork-native user experience, the better long-term path is a host-side local MCP/server bridge: Cowork can run local plugin MCP servers on the host, while shell commands still run in the VM. This skill-only package does not include that bridge yet, but it is the practical way to keep LiteParse available from Cowork without depending on the VM's glibc.

## Install

### Claude Code

```bash
claude plugin marketplace add cyanxxy/liteparse-agent-skills-claude-codex
claude plugin install liteparse@local-liteparse
```

### Claude Cowork

1. **GitHub sync**: push this repo to GitHub, then in Cowork's plugin marketplace use "Add marketplace" and enter `cyanxxy/liteparse-agent-skills-claude-codex`.
2. **Manual upload (ZIP)**: package `liteparse-claude-plugin/` as a `.zip` and upload via Cowork's custom-plugin flow.

## Structure

```
liteparse-claude-plugin/
├── .claude-plugin/plugin.json
├── LICENSE
├── README.md
├── examples/
│   └── invoice.extract.json
└── skills/
    ├── liteparse/SKILL.md              # reference knowledge (auto-load)
    ├── parse-document/SKILL.md         # /liteparse:parse-document
    ├── batch-parse/SKILL.md            # /liteparse:batch-parse
    ├── screenshot-document/SKILL.md    # /liteparse:screenshot-document
    ├── merge-parsed/SKILL.md           # /liteparse:merge-parsed
    ├── compare-documents/SKILL.md      # /liteparse:compare-documents
    ├── extract-tables/SKILL.md         # /liteparse:extract-tables
    ├── extract-structured/SKILL.md     # /liteparse:extract-structured
    └── convert-format/SKILL.md         # /liteparse:convert-format
```

## Credits

Built on [LiteParse v2](https://github.com/run-llama/liteparse) by the [LlamaIndex](https://www.llamaindex.ai/) team. All document parsing, OCR, and screenshot functionality comes from their work.

- [LiteParse repo](https://github.com/run-llama/liteparse)
- [LiteParse docs](https://developers.llamaindex.ai/liteparse/)
- [LlamaIndex official agent skills](https://github.com/run-llama/llamaparse-agent-skills)
- License: Apache-2.0 (matches upstream).
