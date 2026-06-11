# LiteParse for Codex

Codex plugin from the [`liteparse-agent-skills-claude-codex`](https://github.com/cyanxxy/liteparse-agent-skills-claude-codex) repo. Packages LiteParse v2 as a set of reusable Codex skills for local document parsing, OCR, PNG screenshots, and structured extraction.

## Codex skills

Use the Codex skill picker or mention the skill by name with `$...`.

| Skill | Description |
|---------|-------------|
| `$liteparse:parse-document <file> [flags]` | Parse a single PDF, DOCX, XLSX, PPTX, image, or scan into text or JSON |
| `$liteparse:batch-parse <input-dir> [output-dir] [flags]` | Parse every supported file in a directory |
| `$liteparse:screenshot-document <file> [output-dir] [flags]` | Render document pages as PNG images |
| `$liteparse:merge-parsed <files...> [-o output]` | Combine parsed outputs from multiple files into one document |
| `$liteparse:compare-documents <file-a> <file-b> [-o diff]` | Parse two documents and produce a structured diff with summary |
| `$liteparse:extract-tables <file> [--csv\|--json] [-o output]` | Extract tables from documents into CSV or structured JSON |
| `$liteparse:extract-structured <file> [--fields ... \| --schema ...] [--json\|--jsonl\|--csv] [-o output]` | Extract user-defined fields into repeatable structured output |
| `$liteparse:convert-format <file> --to <format> [-o output]` | Convert between file formats via LibreOffice (no parsing) |

The background `liteparse` skill loads automatically as reference material and provides LiteParse v2 CLI reference, dependency rules, JSON output shape, and failure handling.

## Structured extraction

`$liteparse:extract-structured` is the agent-facing entry point for repeatable field extraction. Either:

- pass inline `--fields` to describe what to extract — the agent normalizes the loose request into a canonical schema before extracting, or
- pass `--schema <file>` to reuse a saved recipe as the stable automation contract for later runs.

JSON is the source of truth; `--jsonl` and `--csv` are flattened export views for downstream automation pipelines. Use `--save-schema <file>` to persist the normalized recipe as a reusable contract.

An example reusable schema lives at [`examples/invoice.extract.json`](examples/invoice.extract.json).

## Requirements

- Node.js 18+ and `npm` (for the `npx` fallback).
- Optional: Python 3.10+ for the Python LiteParse package, Python SDK use, or custom OCR server packages.
- Optional: `lit` installed globally with the current npm (`npm i -g @llamaindex/liteparse@latest`), Python (`pip install "liteparse>=2.0.7"`), or Rust (`cargo install liteparse`) package. Do not use the Homebrew LiteParse formula as the primary v2 path; it has lagged upstream.
- Optional: `@llamaindex/liteparse-wasm` for browser/edge PDF parsing experiments. It has no `lit` CLI and does not cover file-path input, Office conversion, built-in/HTTP OCR, or screenshots.
- Optional: `liteparse-server` or LiteParse Docker images for a host/container parser service when an agent runtime cannot load the native CLI.
- Optional: LibreOffice for Office documents (`docx`, `xlsx`, `pptx`, `odt`, ...).
- Optional: ImageMagick (`magick` or `convert`) for image inputs (`png`, `jpg`, `tiff`, `webp`, `svg`, ...).
- Optional env vars: `TESSDATA_PREFIX` (offline OCR language packs).

If `lit` is not on `PATH`, the skills fall back to `npx -y @llamaindex/liteparse@latest`.

### Linux VM note

On older Linux VMs, LiteParse native binaries can fail before startup with loader errors such as `GLIBC_2.38 not found`, `GLIBC_2.39 not found`, or `GLIBCXX_3.4.31 not found`. LiteParse 2.0.6+ lowered the Linux GNU binary baseline, so first use the current package (`npx -y @llamaindex/liteparse@latest ...`, `npm i -g @llamaindex/liteparse@latest`, or `pip install "liteparse>=2.0.7"`).

If a loader error still happens, stop and report the runtime incompatibility instead of retrying the same binary. Capture `uname -m`, `ldd --version`, and the exact error. Then try the Python wheel, build from source with `cargo install liteparse` inside the target VM, run LiteParse in a compatible container/server, or parse in a compatible host environment and bring the output files back.

For runtimes with host-side MCP support, a local MCP/server bridge can expose LiteParse from a compatible host process while the agent UI remains in the restricted runtime.

## Install

This plugin is discovered via `.agents/plugins/marketplace.json` at the repo root. From Codex, add the repo as a local plugin source and install `liteparse` from the `local-workspace` marketplace.

## Structure

```
plugins/liteparse/
├── .codex-plugin/plugin.json
├── LICENSE
├── README.md
├── examples/
│   └── invoice.extract.json
└── skills/
    ├── liteparse/SKILL.md              # reference knowledge (auto-load)
    ├── parse-document/SKILL.md         # $liteparse:parse-document
    ├── batch-parse/SKILL.md            # $liteparse:batch-parse
    ├── screenshot-document/SKILL.md    # $liteparse:screenshot-document
    ├── merge-parsed/SKILL.md           # $liteparse:merge-parsed
    ├── compare-documents/SKILL.md      # $liteparse:compare-documents
    ├── extract-tables/SKILL.md         # $liteparse:extract-tables
    ├── extract-structured/SKILL.md     # $liteparse:extract-structured
    └── convert-format/SKILL.md         # $liteparse:convert-format
```

## Upstream

- Repo: <https://github.com/run-llama/liteparse>
- Docs: <https://developers.llamaindex.ai/liteparse/>
- License: Apache-2.0 (matches upstream).
