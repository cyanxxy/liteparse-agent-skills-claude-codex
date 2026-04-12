# liteparse-agent-skills-claude-codex

Plugin for [Claude Code](https://code.claude.com), [Claude Cowork](https://claude.com/cowork), [OpenAI Codex](https://openai.com/codex/), and [41+ Vercel Skills agents](https://skills.sh) that wraps [LiteParse](https://github.com/run-llama/liteparse) for local document parsing, OCR, and PDF page screenshots.

## What it does

- Parse PDFs, DOCX, XLSX, PPTX, images, and scanned files into text or JSON
- Run OCR locally via Tesseract or an external OCR server
- Batch parse entire directories of documents
- Generate PNG/JPG screenshots of PDF pages
- Merge parsed outputs from multiple files into one document
- Compare two documents and produce a structured diff
- Extract tables from documents into CSV or JSON
- Convert between file formats via LibreOffice (DOCX to PDF, etc.)
- Post-parse hooks for automating actions after operations

Everything runs locally — no cloud dependency.

## Slash commands

| Command | Description |
|---------|-------------|
| `/liteparse:parse-document <file> [flags]` | Parse a single PDF, DOCX, XLSX, PPTX, image, or scan into text or JSON |
| `/liteparse:batch-parse <input-dir> [output-dir] [flags]` | Parse every supported file in a directory |
| `/liteparse:screenshot-document <file.pdf> [output-dir] [flags]` | Render PDF pages as PNG or JPG images |
| `/liteparse:merge-parsed <files...> [-o output]` | Combine parsed outputs from multiple files into one document |
| `/liteparse:compare-documents <file-a> <file-b> [-o diff]` | Parse two documents and produce a structured diff with summary |
| `/liteparse:extract-tables <file> [--csv\|--json] [-o output]` | Extract tables from documents into CSV or structured JSON |
| `/liteparse:convert-format <file> --to <format> [-o output]` | Convert between file formats via LibreOffice (no parsing) |

The background `liteparse` skill loads automatically and provides CLI reference, dependency rules, config file docs, and hook execution instructions.

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
│   │   └── convert-format/SKILL.md         # /liteparse:convert-format
│   ├── examples/liteparse.config.json
│   ├── LICENSE
│   └── README.md
├── plugins/liteparse/                       # Codex plugin
│   ├── .codex-plugin/plugin.json
│   ├── skills/liteparse/SKILL.md
│   ├── examples/liteparse.config.json
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
npx skills add cyanxxy/liteparse-agent-skills-claude-codex --skill compare-documents
```

## Post-parse hooks

Define shell commands in `liteparse.config.json` that run automatically after operations:

```json
{
  "ocrLanguage": "en",
  "dpi": 150,
  "outputFormat": "json",
  "hooks": {
    "postParse": [
      "echo 'Parsed: {{file}} -> {{output}}'",
      "git add '{{output}}'"
    ],
    "postBatchParse": [
      "curl -X POST https://api.example.com/notify -d '{\"dir\": \"{{outputDir}}\"}'"
    ],
    "postScreenshot": [
      "echo 'Screenshots saved to {{outputDir}}'"
    ],
    "postConvert": [
      "echo 'Converted: {{file}} -> {{output}}'"
    ]
  }
}
```

| Hook | Trigger | Variables |
|------|---------|-----------|
| `postParse` | After single file parse | `{{file}}`, `{{output}}` |
| `postBatchParse` | After batch parse | `{{inputDir}}`, `{{outputDir}}` |
| `postScreenshot` | After screenshot generation | `{{file}}`, `{{outputDir}}` |
| `postConvert` | After format conversion | `{{file}}`, `{{output}}` |

Hooks run via `bash -c` with template variables substituted. Failed hooks are reported but don't roll back the operation.

## Requirements

- Node.js 18+ and `npm` (for the `npx` fallback)
- Optional: `lit` installed globally (`npm i -g @llamaindex/liteparse`)
- Optional: LibreOffice (for Office formats and `/liteparse:convert-format`)
- Optional: ImageMagick (for image inputs)

## Credits

Built on top of [LiteParse](https://github.com/run-llama/liteparse) by the [LlamaIndex](https://www.llamaindex.ai/) team. LiteParse provides the core local document parsing engine — this project wraps it as agent skills for Claude Code, Cowork, Codex, and the Vercel Skills ecosystem.

- [LiteParse repo](https://github.com/run-llama/liteparse)
- [LiteParse docs](https://developers.llamaindex.ai/liteparse/)
- [LlamaIndex official agent skills](https://github.com/run-llama/llamaparse-agent-skills)

## License

Apache-2.0 — same as [upstream LiteParse](https://github.com/run-llama/liteparse).
