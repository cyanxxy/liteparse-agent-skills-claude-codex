---
name: liteparse
description: Background knowledge for running LiteParse: choosing between installed lit and the npx fallback, verifying LibreOffice or ImageMagick dependencies, CLI flag reference, and failure handling across parsing workflows.
---

# LiteParse Skill

Parse unstructured documents (PDF, DOCX, PPTX, XLSX, images, and more) locally with LiteParse: fast, lightweight, no cloud dependencies or LLM required.

This is the background/reference skill for LiteParse. It provides context for the action skills (parse-document, batch-parse, screenshot-document, extract-structured, extract-tables, compare-documents, convert-format, merge-parsed). Use it to look up CLI flags, dependency rules, and failure handling — the action skills defer here for those details.

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

LiteParse ships as the npm package `@llamaindex/liteparse`. Two binaries are registered: `lit` and `liteparse`, pointing to the same script.

If `lit` is not installed, install it globally:

```bash
npm i -g @llamaindex/liteparse
```

Verify installation:

```bash
lit --version
```

### CLI availability

- **When `lit` is on PATH**: commands are `lit parse`, `lit batch-parse`, `lit screenshot`.
- **When only `npm` / `npx` is available**: the `lit` prefix is dropped — the subcommand comes first:

  ```bash
  npx -y @llamaindex/liteparse parse file.pdf --format json
  npx -y @llamaindex/liteparse batch-parse ./in ./out --recursive
  npx -y @llamaindex/liteparse screenshot file.pdf -o ./out --target-pages "1-3"
  ```

- **When neither is available**: Node.js 20.11+ and npm are required.

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
| DOC, DOCX, DOCM, ODT, RTF, PPT, PPTX, PPTM, ODP, XLS, XLSX, XLSM, ODS, CSV, TSV | LibreOffice | `libreoffice` |
| JPG, JPEG, PNG, GIF, BMP, TIFF, WebP, SVG | ImageMagick | `magick` or `convert` |

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

Both positional directories are **required** by the upstream CLI — always pass both.

### Generate Page Screenshots

```bash
lit screenshot document.pdf -o ./screenshots
lit screenshot document.pdf --target-pages "1,3,5" -o ./screenshots
lit screenshot document.pdf --dpi 300 --format png -o ./screenshots
```

The output directory is passed via `-o`, not as a positional argument.

---

## Step 2 — Supported Flags (upstream CLI, v1.4.x)

### `parse` / `batch-parse`

- `--format json|text` (default `text`)
- `-o, --output <file>` (parse only; `batch-parse` takes a positional `<output-dir>`)
- `--no-ocr` to disable OCR
- `--ocr-server-url <url>` for external OCR
- `--ocr-language <lang>` (default `en`)
- `--num-workers <n>` OCR parallelism (defaults to CPU count − 1)
- `--max-pages <n>` (default 10000)
- `--target-pages "1-5,10"` (parse only)
- `--dpi <n>` (default 150)
- `--no-precise-bbox` (faster; drops precise bounding boxes)
- `--preserve-small-text` (keeps very small text that would otherwise be dropped)
- `--password <pw>` for encrypted PDFs
- `--config <file>` JSON config
- `--recursive` (batch-parse only)
- `--extension ".pdf"` (batch-parse only)
- `-q, --quiet`

### `screenshot`

- `-o, --output-dir <dir>` (default `./screenshots`) — output directory is passed via `-o`, not positionally
- `--target-pages "1,3,5"` or `"1-5"`
- `--dpi <n>`
- `--format png|jpg` (default `png`)
- `--password <pw>`
- `--config <file>`
- `-q, --quiet`

### Key Options Reference

#### OCR Options

| Option | Description |
|--------|-------------|
| (default) | Tesseract.js — zero setup, built-in |
| `--ocr-language fra` | Set OCR language (ISO code) |
| `--ocr-server-url <url>` | Use external HTTP OCR server |
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
| `--no-precise-bbox` | Disable precise bounding boxes (faster) |
| `--preserve-small-text` | Keep very small text that would otherwise be dropped |

---

## Step 3 — Config File and Post-Parse Hooks

For repeated use with consistent options, create a `liteparse.config.json`. Any command accepts `--config <file>` to load consistent defaults (OCR language, DPI, max pages, output format, etc.).

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

A sample config ships with this skill as `liteparse.config.json` in the repository examples.

### Post-parse hooks

The config file supports a `hooks` object that defines shell commands to run automatically after successful operations. Each hook type is an array of shell command strings.

#### Hook types

| Hook | Trigger | Template variables |
|------|---------|-------------------|
| `postParse` | After a single file parse completes | `{{file}}` (input path), `{{output}}` (output path) |
| `postBatchParse` | After a batch parse completes | `{{inputDir}}`, `{{outputDir}}` |
| `postScreenshot` | After screenshots are generated | `{{file}}` (input PDF), `{{outputDir}}` (screenshot dir) |
| `postConvert` | After a format conversion completes | `{{file}}` (input path), `{{output}}` (output path) |

#### Example config with hooks

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
    ]
  }
}
```

#### How to execute hooks

1. After a successful parse/batch/screenshot/convert, check if a `liteparse.config.json` exists (passed via `--config` or in the project root).
2. Read the `hooks` object and find the matching hook array for the completed operation.
3. For each command, substitute `{{...}}` template variables with actual paths, then run via `bash -c`.
4. Report hook results after the main operation report.
5. If a hook fails, report the error but do **not** roll back the parse output.
6. Show the user which hook commands will run before executing them for the first time in a session.

#### Hook safety

Template variables are substituted as raw strings into a shell command before `bash -c` runs it. Users must single-quote every `{{...}}` placeholder in their hook templates (e.g. `'{{file}}'`, `'{{outputDir}}'`) so that filenames containing spaces, quotes, or shell metacharacters do not break the command or inject arbitrary shell. When reviewing or suggesting hook config, flag any `{{...}}` that is not single-quoted as a shell-injection risk.

---

## Step 4 — Structured Extraction Recipes

The `extract-structured` skill is an agent-facing recipe workflow, not a new upstream `lit` subcommand. It starts from `lit parse <file> --format json` and then uses the parsed pages, text items, and bounding boxes to extract user-defined fields.

- When the user supplies `--fields`, normalize the loose request into a canonical schema before extracting. Default inferred fields to optional single-value `string` fields and give them stable `snake_case` names when the user does not provide one.
- When the user supplies `--schema <file>`, treat that recipe as the stable contract for repeatable runs and downstream automations.
- JSON is the source of truth; `--jsonl` and `--csv` are flattened export views for pipelines.
- A reusable example schema lives at `examples/invoice.extract.json`.

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
| Word | `.doc`, `.docx`, `.docm`, `.odt`, `.rtf` |
| PowerPoint | `.ppt`, `.pptx`, `.pptm`, `.odp` |
| Spreadsheets | `.xls`, `.xlsx`, `.xlsm`, `.ods`, `.csv`, `.tsv` |
| Images | `.jpg`, `.jpeg`, `.png`, `.gif`, `.bmp`, `.tiff`, `.webp`, `.svg` |

Office documents require LibreOffice; images require ImageMagick. LiteParse auto-converts these formats to PDF before parsing.

---

## Known Failure Modes

| Symptom | Cause |
|---|---|
| `lit` not found and `npm` not found | Node.js 20.11+ and npm are not installed |
| LibreOffice conversion error on Office input | `libreoffice` is not on PATH |
| ImageMagick error on image input | Neither `magick` nor `convert` is on PATH |
| OCR server connection refused | The `--ocr-server-url` is unreachable; drop the flag to fall back to built-in Tesseract |
| Password-protected PDF fails | `--password <pw>` was not provided |
