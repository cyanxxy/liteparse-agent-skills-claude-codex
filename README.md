# liteparse-agent-skills-claude-codex

Plugin for [Claude Code](https://code.claude.com), [Claude Cowork](https://claude.com/cowork), [OpenAI Codex](https://openai.com/codex/), and [41+ Vercel Skills agents](https://skills.sh) that wraps [LiteParse v2](https://github.com/run-llama/liteparse) for local document parsing, OCR, and page screenshots.

## What it does

- Parse PDFs, DOCX, XLSX, PPTX, images, and scanned files into text or JSON
- Run OCR locally via Tesseract or an external OCR server
- Batch parse entire directories of documents
- Generate PNG screenshots of document pages
- Merge parsed outputs from multiple files into one document
- Compare two documents and produce a structured diff
- Extract tables from documents into CSV or JSON
- Convert between file formats via LibreOffice (DOCX to PDF, etc.)
- Agent-side workflows for merging, comparing, table extraction, and structured field extraction

Everything runs locally — no cloud dependency.

## Claude commands and Codex skills

Claude Code exposes these as `/liteparse:...` slash commands. Codex exposes the same workflows through the skill picker or `$liteparse:...` skill mentions.

| Workflow | Description |
|---------|-------------|
| `parse-document <file> [flags]` | Parse a single PDF, DOCX, XLSX, PPTX, image, or scan into text or JSON |
| `batch-parse <input-dir> [output-dir] [flags]` | Parse every supported file in a directory |
| `screenshot-document <file> [output-dir] [flags]` | Render document pages as PNG images |
| `merge-parsed <files...> [-o output]` | Combine parsed outputs from multiple files into one document |
| `compare-documents <file-a> <file-b> [-o diff]` | Parse two documents and produce a structured diff with summary |
| `extract-tables <file> [--csv\|--json] [-o output]` | Extract tables from documents into CSV or structured JSON |
| `extract-structured <file> [--fields ... \| --schema ...] [--json\|--jsonl\|--csv] [-o output]` | Extract user-defined fields into repeatable structured output |
| `convert-format <file> --to <format> [-o output]` | Convert between file formats via LibreOffice (no parsing) |

The background `liteparse` skill loads automatically and provides LiteParse v2 CLI reference, dependency rules, JSON output shape, and failure handling.

## Repo layout

```
.
├── .claude-plugin/marketplace.json          # Claude Code / Cowork marketplace
├── .agents/plugins/marketplace.json         # Codex marketplace
├── liteparse-claude-plugin/                 # Claude Code & Cowork plugin
│   ├── .claude-plugin/plugin.json
│   ├── skills/
│   │   ├── liteparse/SKILL.md              # reference knowledge (auto-load)
│   │   ├── parse-document/SKILL.md         # /liteparse:parse-document
│   │   ├── batch-parse/SKILL.md            # /liteparse:batch-parse
│   │   ├── screenshot-document/SKILL.md    # /liteparse:screenshot-document
│   │   ├── merge-parsed/SKILL.md           # /liteparse:merge-parsed
│   │   ├── compare-documents/SKILL.md      # /liteparse:compare-documents
│   │   ├── extract-tables/SKILL.md         # /liteparse:extract-tables
│   │   ├── extract-structured/SKILL.md     # /liteparse:extract-structured
│   │   └── convert-format/SKILL.md         # /liteparse:convert-format
│   ├── examples/
│   │   └── invoice.extract.json
│   ├── LICENSE
│   └── README.md
├── plugins/liteparse/                       # Codex plugin
│   ├── .codex-plugin/plugin.json
│   ├── skills/
│   │   ├── liteparse/SKILL.md
│   │   ├── parse-document/SKILL.md
│   │   ├── batch-parse/SKILL.md
│   │   ├── screenshot-document/SKILL.md
│   │   ├── merge-parsed/SKILL.md
│   │   ├── compare-documents/SKILL.md
│   │   ├── extract-tables/SKILL.md
│   │   ├── extract-structured/SKILL.md
│   │   └── convert-format/SKILL.md
│   ├── examples/
│   │   └── invoice.extract.json
│   ├── LICENSE
│   └── README.md
└── skills/                                  # Vercel Skills (41+ agents)
    ├── metadata.json
    ├── liteparse/SKILL.md
    ├── parse-document/SKILL.md
    ├── batch-parse/SKILL.md
    ├── screenshot-document/SKILL.md
    ├── merge-parsed/SKILL.md
    ├── compare-documents/SKILL.md
    ├── extract-tables/SKILL.md
    ├── extract-structured/SKILL.md
    └── convert-format/SKILL.md
```

## Install

### Claude Code

```bash
claude plugin marketplace add cyanxxy/liteparse-agent-skills-claude-codex
claude plugin install liteparse@local-liteparse
```

### Claude Cowork

1. **GitHub sync**: push this repo to GitHub, then in Cowork's plugin marketplace use "Add marketplace" and enter `cyanxxy/liteparse-agent-skills-claude-codex`.
2. **Manual upload (ZIP)**: package `liteparse-claude-plugin/` as a `.zip` and upload via Cowork's custom-plugin flow.

### OpenAI Codex

The Codex plugin lives in `plugins/liteparse/` and is discovered via `.agents/plugins/marketplace.json` at the repo root.

### Vercel Skills (Cursor, Copilot, Windsurf, Gemini CLI, and 41+ agents)

```bash
npx skills add cyanxxy/liteparse-agent-skills-claude-codex
```

Or install a specific skill:

```bash
npx skills add cyanxxy/liteparse-agent-skills-claude-codex --skill parse-document
npx skills add cyanxxy/liteparse-agent-skills-claude-codex --skill extract-tables
npx skills add cyanxxy/liteparse-agent-skills-claude-codex --skill extract-structured
npx skills add cyanxxy/liteparse-agent-skills-claude-codex --skill compare-documents
```

## Requirements

- Node.js 18+ and `npm` (for the `npx` fallback)
- Optional: Python 3.10+ for the Python LiteParse package, Python SDK use, or custom OCR server packages
- Optional: `lit` installed globally with the current npm (`npm i -g @llamaindex/liteparse@latest`), Python (`pip install "liteparse>=2.0.7"`), or Rust (`cargo install liteparse`) package. Do not use the Homebrew LiteParse formula as the primary v2 path; it has lagged upstream.
- Optional: `@llamaindex/liteparse-wasm` for browser/edge PDF parsing experiments. It has no `lit` CLI and does not cover file-path input, Office conversion, built-in/HTTP OCR, or screenshots.
- Optional: `liteparse-server` or LiteParse Docker images for a host/container parser service when an agent runtime cannot load the native CLI.
- Optional: LibreOffice (for Office formats and the convert-format workflow)
- Optional: ImageMagick (for image inputs)
- Optional env vars: `TESSDATA_PREFIX` (offline OCR language packs)

### Claude Cowork Linux VM note

Claude Cowork shell commands run inside an isolated Linux VM. Older LiteParse native binaries could fail before startup with loader errors such as `GLIBC_2.38 not found`, `GLIBC_2.39 not found`, or `GLIBCXX_3.4.31 not found`. LiteParse 2.0.6+ lowered the Linux GNU binary baseline, so first use the current package (`npx -y @llamaindex/liteparse@latest ...`, `npm i -g @llamaindex/liteparse@latest`, or `pip install "liteparse>=2.0.7"`).

If a loader error still happens, stop and report the runtime incompatibility instead of retrying the same binary. Capture `uname -m`, `ldd --version`, and the exact error. Then try the Python wheel, build from source with `cargo install liteparse` inside the target VM, run LiteParse in a compatible container/server, or parse in a compatible host environment and bring the output files back into Cowork.

For a Cowork-native user experience, the better long-term path is a host-side local MCP/server bridge: Cowork can run local plugin MCP servers on the host, while shell commands still run in the VM. This skill-only package does not include that bridge yet, but it is the practical way to keep LiteParse available from Cowork without depending on the VM's glibc.

## Credits

Built on top of [LiteParse v2](https://github.com/run-llama/liteparse) by the [LlamaIndex](https://www.llamaindex.ai/) team. LiteParse provides the core local document parsing engine — this project wraps it as agent skills for Claude Code, Cowork, Codex, and the Vercel Skills ecosystem.

- [LiteParse repo](https://github.com/run-llama/liteparse)
- [LiteParse docs](https://developers.llamaindex.ai/liteparse/)
- [LlamaIndex official agent skills](https://github.com/run-llama/llamaparse-agent-skills)

## License

Apache-2.0 — same as [upstream LiteParse](https://github.com/run-llama/liteparse).
