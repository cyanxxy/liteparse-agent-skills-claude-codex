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
- Post-operation hook metadata for downstream automation review

Everything runs locally — no cloud dependency.

## Claude commands and Codex skills

Claude Code exposes these as `/liteparse:...` slash commands. Codex exposes the same workflows through the skill picker or `$liteparse:...` skill mentions.

| Workflow | Description |
|---------|-------------|
| `parse-document <file> [flags]` | Parse a single PDF, DOCX, XLSX, PPTX, image, or scan into text or JSON |
| `batch-parse <input-dir> [output-dir] [flags]` | Parse every supported file in a directory |
| `screenshot-document <file.pdf> [output-dir] [flags]` | Render PDF pages as PNG or JPG images |
| `merge-parsed <files...> [-o output]` | Combine parsed outputs from multiple files into one document |
| `compare-documents <file-a> <file-b> [-o diff]` | Parse two documents and produce a structured diff with summary |
| `extract-tables <file> [--csv\|--json] [-o output]` | Extract tables from documents into CSV or structured JSON |
| `extract-structured <file> [--fields ... \| --schema ...] [--json\|--jsonl\|--csv] [-o output]` | Extract user-defined fields into repeatable structured output |
| `convert-format <file> --to <format> [-o output]` | Convert between file formats via LibreOffice (no parsing) |

The background `liteparse` skill loads automatically and provides CLI reference, dependency rules, config file docs, and hook schema notes.

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
│   │   ├── liteparse.config.json
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
│   │   ├── liteparse.config.json
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

## Post-parse hooks

Define shell commands in `liteparse.config.json` to describe post-operation hooks:

```json
{
  "ocrLanguage": "en",
  "dpi": 150,
  "outputFormat": "json",
  "hooks": {
    "postParse": [
      "echo Parsed '{{file}}' to '{{output}}'"
    ],
    "postBatchParse": [
      "echo Batch complete for '{{inputDir}}' into '{{outputDir}}'"
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

The plugin treats these hooks as configuration data. It should not auto-discover or execute repo-defined hook commands during parse, batch, screenshot, or convert workflows. If a user explicitly wants to run one of these commands, review it as ordinary shell automation first; template variables are raw string substitutions and remain an injection/correctness risk when paths contain quotes or shell metacharacters. Do not embed passwords or secrets in hook commands; they can be visible in process lists and logs.

## Requirements

- Node.js 18+ and `npm` (for the `npx` fallback)
- Optional: Python 3.12+ for Python SDK or custom OCR server packages
- Optional: `lit` installed globally (`npm i -g @llamaindex/liteparse`, or `brew tap run-llama/liteparse && brew install llamaindex-liteparse`)
- Optional: LibreOffice (for Office formats and the convert-format workflow)
- Optional: ImageMagick (for image inputs)
- Optional env vars: `TESSDATA_PREFIX` (offline OCR language packs), `LITEPARSE_TMPDIR` (custom temp directory)

## Credits

Built on top of [LiteParse](https://github.com/run-llama/liteparse) by the [LlamaIndex](https://www.llamaindex.ai/) team. LiteParse provides the core local document parsing engine — this project wraps it as agent skills for Claude Code, Cowork, Codex, and the Vercel Skills ecosystem.

- [LiteParse repo](https://github.com/run-llama/liteparse)
- [LiteParse docs](https://developers.llamaindex.ai/liteparse/)
- [LlamaIndex official agent skills](https://github.com/run-llama/llamaparse-agent-skills)

## License

Apache-2.0 — same as [upstream LiteParse](https://github.com/run-llama/liteparse).
