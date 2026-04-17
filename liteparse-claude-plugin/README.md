# LiteParse plugin for Claude Code, Cowork & Codex

Local document parsing via [LiteParse](https://github.com/run-llama/liteparse), packaged as a plugin for Claude Code, Claude Cowork, and OpenAI Codex. Repo name: **`liteparse-agent-skills-claude-codex`**.

## Slash commands

| Command | Description |
|---------|-------------|
| `/liteparse:parse-document <file> [flags]` | Parse a single PDF, DOCX, XLSX, PPTX, image, or scan into text or JSON |
| `/liteparse:batch-parse <input-dir> [output-dir] [flags]` | Parse every supported file in a directory |
| `/liteparse:screenshot-document <file.pdf> [output-dir] [flags]` | Render PDF pages as PNG or JPG images |
| `/liteparse:merge-parsed <files...> [-o output]` | Combine parsed outputs from multiple files into one document |
| `/liteparse:compare-documents <file-a> <file-b> [-o diff]` | Parse two documents and produce a structured diff with summary |
| `/liteparse:extract-tables <file> [--csv\|--json] [-o output]` | Extract tables from documents into CSV or structured JSON |
| `/liteparse:extract-structured <file> [--fields ... \| --schema ...] [--json\|--jsonl\|--csv] [-o output]` | Extract user-defined fields into repeatable structured output |
| `/liteparse:convert-format <file> --to <format> [-o output]` | Convert between file formats via LibreOffice (no parsing) |

The background `liteparse` skill loads automatically and provides CLI reference, dependency rules, config file docs, and hook execution instructions.

## Structured extraction

`/liteparse:extract-structured` is the agent-facing entry point for repeatable field extraction. Users can either:

- pass inline `--fields` to describe what to extract, then let the agent normalize those loose requests into a canonical schema before extraction, or
- pass `--schema <file>` to reuse a saved recipe as the stable automation contract for later Codex, Claude Code, or Cloud Code runs.

The command uses LiteParse JSON output as its source of truth, then writes structured JSON by default. `--jsonl` and `--csv` are flattened export views for downstream automation pipelines.
Use `--save-schema <file>` when you want to persist the normalized recipe as a reusable contract for later runs.

An example reusable schema lives at [`examples/invoice.extract.json`](examples/invoice.extract.json).

## Post-parse hooks

Define shell commands in `examples/liteparse.config.json` that run automatically after operations. Supported hooks:

- `postParse` — after single file parse (`{{file}}`, `{{output}}`)
- `postBatchParse` — after batch parse (`{{inputDir}}`, `{{outputDir}}`)
- `postScreenshot` — after screenshot generation (`{{file}}`, `{{outputDir}}`)
- `postConvert` — after format conversion (`{{file}}`, `{{output}}`)

See the background `liteparse` skill for full hook documentation.

## Requirements

- Node.js 18+ and `npm` (for the `npx` fallback).
- Optional: `lit` installed globally via npm (`npm i -g @llamaindex/liteparse`) or Homebrew (`brew tap run-llama/liteparse && brew install llamaindex-liteparse`).
- Optional: LibreOffice for Office documents (`docx`, `xlsx`, `pptx`, `odt`, ...).
- Optional: ImageMagick (`magick` or `convert`) for image inputs (`png`, `jpg`, `tiff`, `webp`, `svg`, ...).
- Optional env vars: `TESSDATA_PREFIX` (offline OCR language packs), `LITEPARSE_TMPDIR` (custom temp directory).

If `lit` is not on `PATH`, the skills fall back to `npx -y @llamaindex/liteparse`.

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
│   ├── liteparse.config.json
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

Built on [LiteParse](https://github.com/run-llama/liteparse) by the [LlamaIndex](https://www.llamaindex.ai/) team. All document parsing, OCR, and screenshot functionality comes from their work.

- [LiteParse repo](https://github.com/run-llama/liteparse)
- [LiteParse docs](https://developers.llamaindex.ai/liteparse/)
- [LlamaIndex official agent skills](https://github.com/run-llama/llamaparse-agent-skills)
- License: Apache-2.0 (matches upstream).
