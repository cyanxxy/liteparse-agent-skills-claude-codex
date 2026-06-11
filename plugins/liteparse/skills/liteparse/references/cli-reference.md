# LiteParse CLI Reference

LiteParse v2 provides the `lit` CLI. The npm package also registers `liteparse`. If neither is installed, use `npx -y @llamaindex/liteparse@latest <subcommand>` without a `lit` prefix.

## Dependencies

- PDFs: no extra dependency.
- Office and CSV/TSV inputs: `libreoffice`.
- Images: `magick` or `convert`.
- Node.js 18+ and npm are required for the `npx` fallback.
- On older Linux VMs, including some Claude Cowork local VMs, native LiteParse binaries can fail to load with `GLIBC_x.y not found` or `GLIBCXX_x.y.z not found`. First try current LiteParse (`@latest` or `liteparse>=2.0.7`), then use the Python wheel, a source build in that VM, a compatible container/server, or a compatible host environment.
- In Claude Cowork, shell commands run in the Linux VM while local plugin MCP servers can run on the host. If the VM cannot load LiteParse, the practical Cowork path is a host-side MCP/server bridge.
- `@llamaindex/liteparse-wasm` has no `lit` CLI and is limited to PDF-byte parsing with custom JS OCR, not file-path input, Office conversion, built-in/HTTP OCR, or screenshots.

## Common Flags

- `parse`: `--format json|text`, `-o <file>`, `--target-pages "1-5,10"`, `--no-ocr`, `--ocr-language <lang>` (default `eng`), `--ocr-server-url <url>`, optional `--tessdata-path <path>` when shown in help, npm-only `--config <file>` when shown in help, `--dpi`, `--max-pages` (default `1000`), `--preserve-small-text`, `--password`, `--num-workers`, `-q`.
- `batch-parse`: `--format json|text`, OCR flags, optional `--tessdata-path` when shown in help, `--dpi`, `--max-pages` (default `1000`), `--recursive`, `--extension ".pdf"`, `--password`, `--num-workers`, `-q`.
- `screenshot`: `-o <dir>`, `--target-pages`, `--dpi`, `--password`, `-q`. Screenshots are PNG files named `page_<n>.png`.

## JSON Shape

`lit parse --format json` emits a top-level `pages` array. Rust/Python builds use `text_items`; the npm wrapper may expose `textItems`. Agent-side workflows should handle either field name.

## Failure Handling

Report missing binaries, LibreOffice/ImageMagick failures, OCR server connection errors, password failures, `GLIBC`/`GLIBCXX` loader failures, and non-zero CLI output verbatim. Do not invent a cause.
