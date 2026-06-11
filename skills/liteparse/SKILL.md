---
name: liteparse
description: Use this skill when the user asks to parse, convert, compare, merge, or extract data from unstructured files (PDF, DOCX, PPTX, XLSX, images, etc.) locally without cloud dependencies.
compatibility: Requires LiteParse v2 via `lit`/`liteparse` or Node 18+ with npm for the `npx` fallback. LibreOffice for Office files. ImageMagick for images.
license: Apache-2.0
metadata:
  author: Local Workspace
  version: "0.3.0"
---

# LiteParse Skill

Parse unstructured documents (PDF, DOCX, PPTX, XLSX, images, and more) locally with LiteParse v2: fast, lightweight, no cloud dependencies or LLM required.

This is the background/reference skill for LiteParse. It provides context for the action skills (parse-document, batch-parse, screenshot-document, extract-structured, extract-tables, compare-documents, convert-format, merge-parsed). Use it to look up CLI flags, dependency rules, and failure handling — the action skills defer here for those details.

Do not turn this file into an interactive setup flow. Use it to look up CLI flags, dependency rules, JSON output shape, and failure handling while an action skill performs the work.

---

## Step 0 — Install LiteParse (if needed)

LiteParse v2 ships one `lit` CLI across npm, Python, and Rust package managers. The npm package also registers the `liteparse` alias.

Install via the user's preferred package manager:

```bash
npm i -g @llamaindex/liteparse@latest
pip install "liteparse>=2.0.7"
cargo install liteparse
```

For browser or edge PDF-byte parsing experiments only, install the WASM package:

```bash
npm i @llamaindex/liteparse-wasm
```

Do not use the Homebrew LiteParse formula as the primary v2 install path; it has lagged upstream.

Verify installation:

```bash
lit --version
liteparse --version
```

Treat `--version` as a smoke test. Some wrappers have reported stale patch versions; use the package manager (`npm view`, `pip show`, or `cargo info`) when the exact installed patch version matters.

### CLI availability

- **When `lit` or `liteparse` is on PATH**: use the installed binary alias that already exists in the environment for `parse`, `batch-parse`, and `screenshot`.
- **When only `npm` / `npx` is available**: the `lit` prefix is dropped — the subcommand comes first:

  ```bash
  npx -y @llamaindex/liteparse@latest parse file.pdf --format json
  npx -y @llamaindex/liteparse@latest batch-parse ./in ./out --recursive
  npx -y @llamaindex/liteparse@latest screenshot file.pdf -o ./out --target-pages "1-3"
  ```

- **When neither is available**: Node.js 18+ and npm are required for the `npx` fallback.
- **When a Linux VM reports `GLIBC_x.y not found` or `GLIBCXX_x.y.z not found`**: the native LiteParse binary cannot load in that VM. LiteParse 2.0.6+ lowered the Linux GNU binary baseline, so first try the current npm package or `pip install "liteparse>=2.0.7"`. If the error remains, stop and report the runtime incompatibility; capture `uname -m`, `ldd --version`, and the exact error. Recommend the Python wheel, a source build via `cargo install liteparse` inside the VM, a compatible container/server, or parsing in a compatible host environment and importing the output files.
- **When running inside Claude Cowork**: shell commands run in the Linux VM, but local plugin MCP servers can run on the host. If the VM cannot load LiteParse, recommend a host-side MCP/server bridge rather than another CLI retry.
- **When considering WASM**: `@llamaindex/liteparse-wasm` has no `lit` CLI. It can parse PDF bytes in browser/edge-style environments, but it is not a drop-in replacement for file-path input, Office conversion, built-in/HTTP OCR, or screenshots.

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

### File-type dependencies

| Input type | Extra dependency | Binary to check |
|---|---|---|
| PDF | None | — |
| DOC, DOCX, DOCM, ODT, RTF, PAGES, PPT, PPTX, PPTM, ODP, KEY, XLS, XLSX, XLSM, ODS, CSV, TSV, NUMBERS | LibreOffice | `libreoffice` |
| JPG, JPEG, PNG, GIF, BMP, TIFF, WebP, SVG | ImageMagick | `magick` or `convert` |

### Environment variables

| Variable | Purpose |
|---|---|
| `TESSDATA_PREFIX` | Path to a directory containing Tesseract `.traineddata` files when running OCR offline or with custom language packs. |

---

## Step 1 — CLI Commands

### Parse a Single File

```bash
lit parse document.pdf
lit parse document.pdf --format json -o output.json
lit parse document.pdf --target-pages "1-5,10,15-20"
lit parse document.pdf --no-ocr
lit parse document.pdf --ocr-server-url http://localhost:8828/ocr
lit parse document.pdf --dpi 300
curl -sL https://example.com/report.pdf | lit parse --no-ocr -
```

### Batch Parse a Directory

```bash
lit batch-parse ./input-directory ./output-directory
lit batch-parse ./input ./output --extension .pdf --recursive
```

Both positional directories are **required** by the upstream CLI — always pass both.

### Generate Page Screenshots

```bash
lit screenshot document.pdf -o ./screenshots
lit screenshot document.pdf --target-pages "1,3,5" -o ./screenshots
lit screenshot document.pdf --dpi 300 -o ./screenshots
lit screenshot report.docx -o ./screenshots
```

The output directory is passed via `-o`, not as a positional argument.

---

## Step 2 — Supported Flags (upstream CLI, v2.x)

### `parse`

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

### `batch-parse`

- `--format json|text` (default `text`)
- `--no-ocr` to disable OCR
- `--ocr-server-url <url>` for external OCR
- `--ocr-language <lang>` (Tesseract code, default `eng`)
- `--tessdata-path <path>` to override `TESSDATA_PREFIX` when the active CLI exposes it
- `--num-workers <n>` OCR parallelism (defaults to CPU count − 1)
- `--max-pages <n>` per file (default 1000)
- `--dpi <n>` (default 150)
- `--password <pw>` for encrypted documents
- `--recursive` (batch-parse only)
- `--extension ".pdf"` (batch-parse only)
- `-q, --quiet`

### `screenshot`

- `-o, --output-dir <dir>` (default `./screenshots`) — output directory is passed via `-o`, not positionally
- `--target-pages "1,3,5"` or `"1-5"`
- `--dpi <n>`
- `--password <pw>` (**caution**: visible in process lists)
- `-q, --quiet`

LiteParse v2 writes PNG screenshots named `page_<n>.png`.

### Key Options Reference

#### OCR Options

| Option | Description |
|--------|-------------|
| (default) | Tesseract — zero setup, bundled with LiteParse |
| `--ocr-language fra` | Set OCR language (Tesseract code) |
| `--ocr-server-url <url>` | Use external HTTP OCR server |
| `--tessdata-path <path>` | Use a local tessdata directory when supported by the active CLI; for npm fallback portability, set `TESSDATA_PREFIX` instead |
| `--no-ocr` | Disable OCR entirely |

#### Output Options

| Option | Description |
|--------|-------------|
| `--format json` | Structured JSON with bounding boxes |
| `--format text` | Plain text (default) |
| `-o <file>` | Save output to file |

#### Performance / Quality Options

| Option | Description |
|--------|-------------|
| `--dpi <n>` | Rendering DPI (default: 150; use 300 for high quality) |
| `--max-pages <n>` | Limit pages parsed |
| `--target-pages <pages>` | Parse specific pages (e.g. `"1-5,10"`) |
| `--preserve-small-text` | Keep very small text that would otherwise be dropped |

---

## Step 3 — JSON Output Shape

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

---

## Step 4 — Structured Extraction Recipes

The `extract-structured` skill is an agent-facing recipe workflow, not a new upstream `lit` subcommand. It starts from `lit parse <file> --format json` and then uses the parsed pages, text items, and bounding boxes to extract user-defined fields.

- When the user supplies `--fields`, normalize the loose request into a canonical schema before extracting. Default inferred fields to optional single-value `string` fields and give them stable `snake_case` names when the user does not provide one.
- When the user supplies `--schema <file>`, treat that recipe as the stable contract for repeatable runs and downstream automations.
- JSON is the source of truth; `--jsonl` and `--csv` are flattened export views for pipelines.
- A reusable example schema lives at [`extract-structured/invoice.extract.json`](../extract-structured/invoice.extract.json).

---

## Step 5 — HTTP OCR Server API (Advanced)

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

---

## Supported Input Formats

| Category | Formats |
|----------|---------|
| PDF | `.pdf` |
| Word | `.doc`, `.docx`, `.docm`, `.odt`, `.rtf`, `.pages` |
| PowerPoint | `.ppt`, `.pptx`, `.pptm`, `.odp`, `.key` |
| Spreadsheets | `.xls`, `.xlsx`, `.xlsm`, `.ods`, `.csv`, `.tsv`, `.numbers` |
| Images | `.jpg`, `.jpeg`, `.png`, `.gif`, `.bmp`, `.tiff`, `.webp`, `.svg` |

Office documents require LibreOffice; images require ImageMagick. LiteParse auto-converts these formats to PDF before parsing.

---

## Common Workflows (skill chaining)

The action skills compose naturally for common real-world tasks. When a user's request matches one of these patterns, chain the skills in order rather than reinventing the pipeline.

| Goal | Chain |
|---|---|
| Consolidate a folder of PDFs into one searchable JSON file | `batch-parse` (→ JSON) → `merge-parsed` (→ single JSON) |
| Extract fields from every invoice in a directory | `batch-parse` (→ JSON) → `extract-structured` run per file (or loop with a shared `--schema`) |
| Summarize a scanned report | `parse-document --no-ocr=false` (→ text) → hand the text to the summarizer |
| Compare two contracts for review | `compare-documents <a> <b>` (produces text diff + summary directly) |
| Make a PowerPoint deck searchable | `convert-format deck.pptx --to pdf` → `parse-document deck.pdf --format json` |
| Generate page previews of an Office doc | `convert-format report.docx --to pdf` → `screenshot-document report.pdf` |
| Pull a table out of a scanned PDF | `parse-document scan.pdf --format json` (verifies OCR works) → `extract-tables scan.pdf` |
| Extract invoice line items (repeating rows) | `extract-structured invoice.pdf --schema invoice.extract.json` with a `multiple: true` line-items field |

When proposing a chain, always show the user the intermediate artifacts (temp file paths, counts, etc.) so they can audit each stage.

---

## Known Failure Modes

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
