# liteparse-claude-codexplugin

Plugin for [Claude Code](https://code.claude.com), [Claude Cowork](https://claude.com/cowork), and [OpenAI Codex](https://openai.com/codex/) that wraps [LiteParse](https://github.com/run-llama/liteparse) for local document parsing, OCR, and PDF page screenshots.

## What it does

- Parse PDFs, DOCX, XLSX, PPTX, images, and scanned files into text or JSON
- Run OCR locally via Tesseract or an external OCR server
- Batch parse entire directories of documents
- Generate PNG/JPG screenshots of PDF pages

Everything runs locally — no cloud dependency.

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
│   │   └── screenshot-document/SKILL.md    # /liteparse:screenshot-document
│   ├── examples/liteparse.config.json
│   ├── LICENSE
│   └── README.md
└── plugins/liteparse/                       # Codex plugin
    ├── .codex-plugin/plugin.json
    ├── skills/liteparse/SKILL.md
    ├── examples/liteparse.config.json
    ├── LICENSE
    └── README.md
```

## Install

### Claude Code

```bash
claude plugin marketplace add cyanxxy/liteparse-claude-codexplugin
claude plugin install liteparse@local-liteparse
```

### Claude Cowork

1. **From GitHub**: in Cowork's plugin marketplace, use "Add marketplace" and point it at `cyanxxy/liteparse-claude-codexplugin`.
2. **Custom upload**: package `liteparse-claude-plugin/` and upload via Cowork's custom-plugin flow.

### OpenAI Codex

The Codex plugin lives in `plugins/liteparse/` and is discovered via `.agents/plugins/marketplace.json` at the repo root.

## Requirements

- Node.js 18+ and `npm` (for the `npx` fallback)
- Optional: `lit` installed globally (`npm i -g @llamaindex/liteparse`)
- Optional: LibreOffice (for Office formats)
- Optional: ImageMagick (for image inputs)

## License

Apache-2.0 — same as [upstream LiteParse](https://github.com/run-llama/liteparse).
