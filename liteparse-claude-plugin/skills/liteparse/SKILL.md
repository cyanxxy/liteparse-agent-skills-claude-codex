---
name: liteparse
description: "Background knowledge for running LiteParse: choosing between installed binaries and the npx fallback, verifying LibreOffice or ImageMagick dependencies, CLI flag reference, and failure handling across parsing workflows."
---

# LiteParse Reference

LiteParse v2 is a local, no-cloud document parser. This skill is **reference only** — it provides context for the action skills `/liteparse:parse-document`, `/liteparse:batch-parse`, `/liteparse:screenshot-document`, `/liteparse:extract-structured`, `/liteparse:extract-tables`, `/liteparse:compare-documents`, `/liteparse:convert-format`, and `/liteparse:merge-parsed`. Do not execute steps from this skill directly; defer to whichever action skill is active.

## CLI availability

LiteParse v2 ships one `lit` CLI across package managers. The npm package also registers the `liteparse` alias.

- **When `lit` or `liteparse` is on PATH**: use the installed binary alias that already exists in the environment for `parse`, `batch-parse`, and `screenshot`.
- **When only `npm` / `npx` is available**: the `lit` prefix is dropped — `npx -y @llamaindex/liteparse@latest parse`, `npx -y @llamaindex/liteparse@latest batch-parse`, `npx -y @llamaindex/liteparse@latest screenshot`.
- **When neither is available**: Node.js 18+ and npm are required. Install with `npm i -g @llamaindex/liteparse@latest`, Python (`pip install "liteparse>=2.0.7"`), or Rust (`cargo install liteparse`), then verify with `lit --version` or `liteparse --version`. Treat `--version` as a smoke test; use the package manager when the exact patch version matters. Do not use the Homebrew LiteParse formula as the primary v2 path because it has lagged upstream.
- **When a Linux VM reports `GLIBC_x.y not found` or `GLIBCXX_x.y.z not found`**: the native LiteParse binary cannot load in that VM. LiteParse 2.0.6+ lowered the Linux GNU binary baseline, so first try the current npm package or `pip install "liteparse>=2.0.7"`. If the error remains, stop and report the runtime incompatibility; capture `uname -m`, `ldd --version`, and the exact error. Recommend the Python wheel, a source build via `cargo install liteparse` inside the VM, a compatible container/server, or parsing in a compatible host environment and importing the output files.
- **When running inside Claude Cowork**: shell commands run in the Linux VM, but local plugin MCP servers can run on the host. If the VM cannot load LiteParse, recommend a host-side MCP/server bridge rather than another CLI retry.
- **When considering WASM**: `@llamaindex/liteparse-wasm` has no `lit` CLI. It can parse PDF bytes in browser/edge-style environments, but it is not a drop-in replacement for file-path input, Office conversion, built-in/HTTP OCR, or screenshots.

### Installing extra dependencies

For Office document support (DOCX, PPTX, XLSX), LibreOffice is required:

```bash
# macOS
brew install --cask libreoffice

# Ubuntu/Debian
apt-get install libreoffice
```

For image parsing, ImageMagick is required:

```bash
# macOS
brew install imagemagick

# Ubuntu/Debian
apt-get install imagemagick
```

## File-type dependencies

| Input type | Extra dependency | Binary to check |
|---|---|---|
| PDF | None | — |
| DOC, DOCX, DOCM, ODT, RTF, PAGES, PPT, PPTX, PPTM, ODP, KEY, XLS, XLSX, XLSM, ODS, CSV, TSV, NUMBERS | LibreOffice | `libreoffice` |
| JPG, JPEG, PNG, GIF, BMP, TIFF, WebP, SVG | ImageMagick | `magick` or `convert` |

## Environment variables

| Variable | Purpose |
|---|---|
| `TESSDATA_PREFIX` | Path to a directory containing Tesseract `.traineddata` files when running OCR offline or with custom language packs. |

## Supported flags (upstream CLI, v2.x)

**`parse`:**
- `--format json|text` (default `text`)
- `-o, --output <file>`
- `--no-ocr` to disable OCR
- `--ocr-server-url <url>` for external OCR
- `--ocr-language <lang>` (Tesseract code, default `eng`)
- `--tessdata-path <path>` to override `TESSDATA_PREFIX` when the active CLI exposes it
- `--num-workers <n>` OCR parallelism (defaults to CPU count − 1)
- `--max-pages <n>` (default 1000)
- `--target-pages "1-5,10"`
- `--dpi <n>` (default 150)
- `--preserve-small-text` (keeps very small text that would otherwise be dropped)
- `--config <file>` when the active CLI exposes it (the npm CLI exposes this flag; official cross-package docs may not)
- `--password <pw>` for encrypted documents (**caution**: the password is visible in process lists and shell history)
- `-q, --quiet`

**`batch-parse`:**
- `--format json|text` (default `text`)
- `--no-ocr` to disable OCR
- `--ocr-server-url <url>` for external OCR
- `--ocr-language <lang>` (Tesseract code, default `eng`)
- `--tessdata-path <path>` to override `TESSDATA_PREFIX` when the active CLI exposes it
- `--num-workers <n>` OCR parallelism (defaults to CPU count - 1)
- `--max-pages <n>` per file (default 1000)
- `--dpi <n>` (default 150)
- `--password <pw>` for encrypted documents
- `--recursive` (batch-parse only)
- `--extension ".pdf"` (batch-parse only)
- `-q, --quiet`

**`screenshot`:**
- `-o, --output-dir <dir>` (default `./screenshots`) — output directory is passed via `-o`, not positionally
- `--target-pages "1,3,5"` or `"1-5"`
- `--dpi <n>`
- `--password <pw>` (**caution**: visible in process lists)
- `-q, --quiet`

LiteParse v2 writes PNG screenshots named `page_<n>.png`.

### Key options reference

#### OCR options

| Option | Description |
|--------|-------------|
| (default) | Tesseract — zero setup, bundled with LiteParse |
| `--ocr-language fra` | Set OCR language (Tesseract code) |
| `--ocr-server-url <url>` | Use external HTTP OCR server |
| `--tessdata-path <path>` | Use a local tessdata directory when supported by the active CLI; for npm fallback portability, set `TESSDATA_PREFIX` instead |
| `--no-ocr` | Disable OCR entirely |

#### Output options

| Option | Description |
|--------|-------------|
| `--format json` | Structured JSON with bounding boxes |
| `--format text` | Plain text (default) |
| `-o <file>` | Save output to file |

#### Performance / quality options

| Option | Description |
|--------|-------------|
| `--dpi <n>` | Rendering DPI (default: 150; use 300 for high quality) |
| `--max-pages <n>` | Limit pages parsed |
| `--target-pages <pages>` | Parse specific pages (e.g. `"1-5,10"`) |
| `--preserve-small-text` | Keep very small text that would otherwise be dropped |

## JSON output shape

`lit parse --format json` writes an object with a `pages` array. Each page includes page number, dimensions, full page text, and positioned text items. Rust/Python builds use `text_items`; the npm wrapper may expose `textItems`. Agent-side workflows should accept either key.

```json
{
  "pages": [
    {
      "page": 1,
      "width": 612,
      "height": 792,
      "text": "Page text...",
      "text_items": [
        {
          "text": "Hello",
          "x": 72,
          "y": 96,
          "width": 40,
          "height": 12,
          "confidence": 1
        }
      ]
    }
  ]
}
```

## Structured extraction recipes

`/liteparse:extract-structured` is an agent-facing recipe workflow, not a new upstream `lit` subcommand. It starts from `lit parse <file> --format json` and then uses the parsed pages, text items, and bounding boxes to extract user-defined fields.

- When the user supplies `--fields`, normalize the loose request into a canonical schema before extracting. Default inferred fields to optional single-value `string` fields and give them stable `snake_case` names when the user does not provide one.
- When the user supplies `--schema <file>`, treat that recipe as the stable contract for repeatable runs and downstream automations.
- JSON is the source of truth; `--jsonl` and `--csv` are flattened export views for pipelines.
- A reusable example schema lives at [`examples/invoice.extract.json`](../../examples/invoice.extract.json).

## HTTP OCR server API (advanced)

If the user wants to plug in a custom OCR backend, the server must implement:

- **Endpoint**: `POST /ocr`
- **Accepts**: `file` (multipart) and `language` (string) parameters
- **Returns**:

```json
{
  "results": [
    { "text": "Hello", "bbox": [x1, y1, x2, y2], "confidence": 0.98 }
  ]
}
```

## Supported input formats

| Category | Formats |
|----------|---------|
| PDF | `.pdf` |
| Word | `.doc`, `.docx`, `.docm`, `.odt`, `.rtf`, `.pages` |
| PowerPoint | `.ppt`, `.pptx`, `.pptm`, `.odp`, `.key` |
| Spreadsheets | `.xls`, `.xlsx`, `.xlsm`, `.ods`, `.csv`, `.tsv`, `.numbers` |
| Images | `.jpg`, `.jpeg`, `.png`, `.gif`, `.bmp`, `.tiff`, `.webp`, `.svg` |

Office documents require LibreOffice; images require ImageMagick. LiteParse auto-converts these formats to PDF before parsing.

## Common workflows (skill chaining)

The action skills compose naturally for common real-world tasks. When a user's request matches one of these patterns, chain the skills in order rather than reinventing the pipeline.

| Goal | Chain |
|---|---|
| Consolidate a folder of PDFs into one searchable JSON file | `/liteparse:batch-parse` (→ JSON) → `/liteparse:merge-parsed` (→ single JSON) |
| Extract fields from every invoice in a directory | `/liteparse:batch-parse` (→ JSON) → `/liteparse:extract-structured` run per file with a shared `--schema` |
| Summarize a scanned report | `/liteparse:parse-document` (→ text) → hand the text to the summarizer |
| Compare two contracts for review | `/liteparse:compare-documents <a> <b>` |
| Make a PowerPoint deck searchable | `/liteparse:convert-format deck.pptx --to pdf` → `/liteparse:parse-document deck.pdf --format json` |
| Generate page previews of an Office doc | `/liteparse:convert-format report.docx --to pdf` → `/liteparse:screenshot-document report.pdf` |
| Pull a table out of a scanned PDF | `/liteparse:parse-document scan.pdf --format json` → `/liteparse:extract-tables scan.pdf` |
| Extract invoice line items (repeating rows) | `/liteparse:extract-structured invoice.pdf --schema invoice.extract.json` with a `multiple: true` line-items field |

When proposing a chain, always show the user the intermediate artifacts (temp file paths, counts, etc.) so they can audit each stage.

## Known failure modes

| Symptom | Cause |
|---|---|
| `lit`/`liteparse` not found and `npm` not found | Node.js 18+ and npm are not installed |
| Loader error mentions `GLIBC_x.y not found` or `GLIBCXX_x.y.z not found` | The Linux VM is older than the native LiteParse binary expects. First try current LiteParse (`@latest` or `liteparse>=2.0.7`), then use the Python wheel, a source build in that VM, a compatible container/server, or a compatible host environment. |
| Claude Cowork VM cannot load the native CLI | Keep the workflow in Cowork by using a host-side MCP/server bridge, or parse in a compatible host/container and import the outputs. |
| WASM package requested as a fallback | It has no CLI and only covers PDF-byte parsing with custom JS OCR; it does not support the full command-skill surface. |
| LibreOffice conversion error on Office input | `libreoffice` is not on PATH |
| ImageMagick error on image input | Neither `magick` nor `convert` is on PATH |
| OCR server connection refused | The `--ocr-server-url` is unreachable; drop the flag to fall back to built-in Tesseract |
| Password-protected document fails | `--password <pw>` was not provided. Avoid putting passwords into shell history when possible. |
