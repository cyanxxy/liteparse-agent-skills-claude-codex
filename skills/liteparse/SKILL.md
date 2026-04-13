---
name: liteparse
description: Use this skill when the user asks to parse, convert, compare, merge, or extract data from unstructured files (PDF, DOCX, PPTX, XLSX, images, etc.) locally without cloud dependencies.
compatibility: Requires Node 20.11+ and `@llamaindex/liteparse` installed globally via npm (`npm i -g @llamaindex/liteparse`). LibreOffice for Office files. ImageMagick for images.
license: Apache-2.0
metadata:
  author: Local Workspace
  version: "0.2.0"
---

# LiteParse Skill

Parse unstructured documents (PDF, DOCX, PPTX, XLSX, images, and more) locally with LiteParse: fast, lightweight, no cloud dependencies or LLM required.

## Initial Setup

When this skill is invoked, respond with:

```
I'm ready to use LiteParse to parse files locally. Before we begin, please confirm that:

- `@llamaindex/liteparse` is installed globally (`npm i -g @llamaindex/liteparse`)
- The `lit` CLI command is available in your terminal

If `lit` is not installed, I will use `npx -y @llamaindex/liteparse` as a fallback.

Please provide:

1. One or more files to parse (PDF, DOCX, PPTX, XLSX, images, etc.)
2. Any specific options: output format (json/text), page ranges, OCR preferences, DPI, etc.
3. What you'd like to do with the parsed content.
```

Then wait for the user's input.

---

## Step 0 — Install LiteParse (if needed)

If `lit` is not installed, install it globally:

```bash
npm i -g @llamaindex/liteparse
```

Verify installation:

```bash
lit --version
```

If `lit` is missing but `npm` exists, fall back to npx. **Drop the `lit` prefix** — the subcommand comes first:

```bash
npx -y @llamaindex/liteparse parse file.pdf --format json
npx -y @llamaindex/liteparse batch-parse ./in ./out --recursive
npx -y @llamaindex/liteparse screenshot file.pdf -o ./out --target-pages "1-3"
```

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
```

### Batch Parse a Directory

```bash
lit batch-parse ./input-directory ./output-directory
lit batch-parse ./input ./output --extension .pdf --recursive
```

### Generate Page Screenshots

```bash
lit screenshot document.pdf -o ./screenshots
lit screenshot document.pdf --target-pages "1,3,5" -o ./screenshots
lit screenshot document.pdf --dpi 300 --format png -o ./screenshots
```

---

## Step 2 — Key Options Reference

### OCR Options

| Option | Description |
|--------|-------------|
| (default) | Tesseract.js — zero setup, built-in |
| `--ocr-language fra` | Set OCR language (ISO code) |
| `--ocr-server-url <url>` | Use external HTTP OCR server |
| `--no-ocr` | Disable OCR entirely |

### Output Options

| Option | Description |
|--------|-------------|
| `--format json` | Structured JSON with bounding boxes |
| `--format text` | Plain text (default) |
| `-o <file>` | Save output to file |

### Performance / Quality Options

| Option | Description |
|--------|-------------|
| `--dpi <n>` | Rendering DPI (default: 150; use 300 for high quality) |
| `--max-pages <n>` | Limit pages parsed |
| `--target-pages <pages>` | Parse specific pages (e.g. `"1-5,10"`) |
| `--no-precise-bbox` | Disable precise bounding boxes (faster) |
| `--preserve-small-text` | Keep very small text that would otherwise be dropped |

---

## Step 3 — Config File and Post-Parse Hooks

For repeated use with consistent options, create a `liteparse.config.json`:

```json
{
  "ocrLanguage": "en",
  "ocrEnabled": true,
  "maxPages": 1000,
  "dpi": 150,
  "outputFormat": "json",
  "preserveVerySmallText": false,
  "hooks": {
    "postParse": [
      "echo 'Parsed: {{file}} -> {{output}}'"
    ]
  }
}
```

Use with:

```bash
lit parse document.pdf --config liteparse.config.json
```

See the `postParse` hooks section for details on automating actions after parsing.

---

## Step 4 — HTTP OCR Server API (Advanced)

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
| Word | `.doc`, `.docx`, `.docm`, `.odt`, `.rtf` |
| PowerPoint | `.ppt`, `.pptx`, `.pptm`, `.odp` |
| Spreadsheets | `.xls`, `.xlsx`, `.xlsm`, `.ods`, `.csv`, `.tsv` |
| Images | `.jpg`, `.jpeg`, `.png`, `.gif`, `.bmp`, `.tiff`, `.webp`, `.svg` |

Office documents require LibreOffice; images require ImageMagick. LiteParse auto-converts these formats to PDF before parsing.
