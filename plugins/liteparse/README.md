# LiteParse for Codex

Codex plugin from the [`liteparse-claude-codexplugin`](https://github.com/cyanxxy/liteparse-claude-codexplugin) repo. Packages LiteParse as a reusable skill.

## Included

- `.codex-plugin/plugin.json` with finalized local metadata
- `skills/liteparse/SKILL.md` for document parsing workflows
- `examples/liteparse.config.json` as a starter config
- `.agents/plugins/marketplace.json` entry for local discovery in this workspace

## Use Cases

- Parse PDFs into plain text or JSON
- OCR scanned documents locally
- Parse Office files and images through LiteParse conversion
- Generate PDF page screenshots

## Runtime Notes

- Prefer `lit` when it is already installed globally
- Otherwise use `npx -y @llamaindex/liteparse`
- Office formats require LibreOffice
- Image formats require ImageMagick

## Upstream

- Repo: <https://github.com/run-llama/liteparse>
- Docs: <https://developers.llamaindex.ai/liteparse/>
