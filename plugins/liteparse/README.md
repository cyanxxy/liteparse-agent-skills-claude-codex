# LiteParse for Codex

Codex plugin from the [`liteparse-agent-skills-claude-codex`](https://github.com/cyanxxy/liteparse-agent-skills-claude-codex) repo. Packages LiteParse as a set of reusable Codex skills for local document parsing, OCR, PDF screenshots, and structured extraction.

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

`/liteparse:extract-structured` is the agent-facing entry point for repeatable field extraction. Either:

- pass inline `--fields` to describe what to extract — the agent normalizes the loose request into a canonical schema before extracting, or
- pass `--schema <file>` to reuse a saved recipe as the stable automation contract for later runs.

JSON is the source of truth; `--jsonl` and `--csv` are flattened export views for downstream automation pipelines. Use `--save-schema <file>` to persist the normalized recipe as a reusable contract.

An example reusable schema lives at [`examples/invoice.extract.json`](examples/invoice.extract.json).

## Post-parse hooks

Define shell commands in `examples/liteparse.config.json` that run automatically after operations. Supported hooks:

- `postParse` — after single file parse (`{{file}}`, `{{output}}`)
- `postBatchParse` — after batch parse (`{{inputDir}}`, `{{outputDir}}`)
- `postScreenshot` — after screenshot generation (`{{file}}`, `{{outputDir}}`)
- `postConvert` — after format conversion (`{{file}}`, `{{output}}`)

Hooks execute via `bash -c` with `{{...}}` substituted as raw strings. **Always single-quote template variables** in your hook commands (e.g. `'{{file}}'`) so filenames containing spaces or shell metacharacters cannot inject arbitrary commands. See the background `liteparse` skill for full hook documentation.

## Requirements

- Node.js 18+ and `npm` (for the `npx` fallback).
- Optional: `lit` installed globally via npm (`npm i -g @llamaindex/liteparse`) or Homebrew (`brew tap run-llama/liteparse && brew install llamaindex-liteparse`).
- Optional: LibreOffice for Office documents (`docx`, `xlsx`, `pptx`, `odt`, ...).
- Optional: ImageMagick (`magick` or `convert`) for image inputs (`png`, `jpg`, `tiff`, `webp`, `svg`, ...).
- Optional env vars: `TESSDATA_PREFIX` (offline OCR language packs), `LITEPARSE_TMPDIR` (custom temp directory).

If `lit` is not on `PATH`, the skills fall back to `npx -y @llamaindex/liteparse`.

## Install

This plugin is discovered via `.agents/plugins/marketplace.json` at the repo root. From Codex, add the repo as a local plugin source and install `liteparse` from the `local-workspace` marketplace.

## Structure

```
plugins/liteparse/
├── .codex-plugin/plugin.json
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

## Upstream

- Repo: <https://github.com/run-llama/liteparse>
- Docs: <https://developers.llamaindex.ai/liteparse/>
- License: Apache-2.0 (matches upstream).
