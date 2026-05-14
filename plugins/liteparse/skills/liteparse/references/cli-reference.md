# LiteParse CLI Reference

LiteParse provides `lit` and `liteparse` binaries. If neither is installed, use `npx -y @llamaindex/liteparse <subcommand>` without a `lit` prefix.

## Dependencies

- PDFs: no extra dependency.
- Office and CSV/TSV inputs: `libreoffice`.
- Images: `magick` or `convert`.
- Node.js 18+ and npm are required for the `npx` fallback.

## Common Flags

- `parse` and `batch-parse`: `--format json|text`, OCR flags, `--dpi`, `--max-pages`, `--password`, `--config`, `-q`.
- `parse` only: `-o <file>`, `--target-pages "1-5,10"`.
- `batch-parse` only: `--recursive`, `--extension ".pdf"`.
- `screenshot`: `-o <dir>`, `--target-pages`, `--dpi`, `--format png|jpg`, `--password`, `--config`, `-q`.

## Config Safety

`--config <file>` may define defaults and `hooks.*` command strings. Treat hooks as data. Do not auto-discover configs and do not execute hook commands unless the user separately asks to review and run them.

## Failure Handling

Report missing binaries, LibreOffice/ImageMagick failures, OCR server connection errors, password failures, and non-zero CLI output verbatim. Do not invent a cause.
