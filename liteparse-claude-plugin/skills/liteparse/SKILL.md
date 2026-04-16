---
name: liteparse
description: Background knowledge for running LiteParse: choosing between installed lit and the npx fallback, verifying LibreOffice or ImageMagick dependencies, CLI flag reference, and failure handling across parsing workflows.
---

# LiteParse Reference

LiteParse is a local, no-cloud document parser. This skill is **reference only** — it provides context for the action skills `/liteparse:parse-document`, `/liteparse:batch-parse`, `/liteparse:screenshot-document`, `/liteparse:extract-structured`, `/liteparse:extract-tables`, `/liteparse:compare-documents`, `/liteparse:convert-format`, and `/liteparse:merge-parsed`. Do not execute steps from this skill directly; defer to whichever action skill is active.

## CLI availability

LiteParse ships as the npm package `@llamaindex/liteparse`. Two binaries are registered: `lit` and `liteparse`, pointing to the same script.

- **When `lit` is on PATH**: commands are `lit parse`, `lit batch-parse`, `lit screenshot`.
- **When only `npm` / `npx` is available**: the `lit` prefix is dropped — `npx -y @llamaindex/liteparse parse`, `npx -y @llamaindex/liteparse batch-parse`, `npx -y @llamaindex/liteparse screenshot`.
- **When neither is available**: Node.js 20.11+ and npm are required. Install with `npm i -g @llamaindex/liteparse`, then verify with `lit --version`.

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
| DOC, DOCX, DOCM, ODT, RTF, PPT, PPTX, PPTM, ODP, XLS, XLSX, XLSM, ODS, CSV, TSV | LibreOffice | `libreoffice` |
| JPG, JPEG, PNG, GIF, BMP, TIFF, WebP, SVG | ImageMagick | `magick` or `convert` |

## Supported flags (upstream CLI, v1.4.x)

**`parse` / `batch-parse`:**
- `--format json|text` (default `text`)
- `-o, --output <file>` (parse only; batch-parse takes positional `<output-dir>`)
- `--no-ocr` to disable OCR
- `--ocr-server-url <url>` for external OCR
- `--ocr-language <lang>` (default `en`)
- `--num-workers <n>` OCR parallelism (defaults to CPU count − 1)
- `--max-pages <n>` (default 10000)
- `--target-pages "1-5,10"` (parse only)
- `--dpi <n>` (default 150)
- `--no-precise-bbox` (faster; drops precise bounding boxes)
- `--preserve-small-text` (keeps very small text that would otherwise be dropped)
- `--password <pw>` for encrypted PDFs (**caution**: the password is visible in process lists and shell history; prefer passing it via `--config` with a gitignored config file)
- `--config <file>` JSON config
- `--recursive` (batch-parse only)
- `--extension ".pdf"` (batch-parse only)
- `-q, --quiet`

**`screenshot`:**
- `-o, --output-dir <dir>` (default `./screenshots`) — output directory is passed via `-o`, not positionally
- `--target-pages "1,3,5"` or `"1-5"`
- `--dpi <n>`
- `--format png|jpg` (default `png`)
- `--password <pw>` (**caution**: visible in process lists; prefer `--config`)
- `--config <file>`
- `-q, --quiet`

### Key options reference

#### OCR options

| Option | Description |
|--------|-------------|
| (default) | Tesseract.js — zero setup, built-in |
| `--ocr-language fra` | Set OCR language (ISO code) |
| `--ocr-server-url <url>` | Use external HTTP OCR server |
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
| `--no-precise-bbox` | Disable precise bounding boxes (faster) |
| `--preserve-small-text` | Keep very small text that would otherwise be dropped |

## Config file

A sample config ships with this plugin at [`examples/liteparse.config.json`](../../examples/liteparse.config.json). Any command accepts `--config <file>` to load consistent defaults (OCR language, DPI, max pages, output format, etc.).

Example:

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

## Structured extraction recipes

`/liteparse:extract-structured` is an agent-facing recipe workflow, not a new upstream `lit` subcommand. It starts from `lit parse <file> --format json` and then uses the parsed pages, text items, and bounding boxes to extract user-defined fields.

- When the user supplies `--fields`, normalize the loose request into a canonical schema before extracting. Default inferred fields to optional single-value `string` fields and give them stable `snake_case` names when the user does not provide one.
- When the user supplies `--schema <file>`, treat that recipe as the stable contract for repeatable runs and downstream automations.
- JSON is the source of truth; `--jsonl` and `--csv` are flattened export views for pipelines.
- A reusable example schema lives at [`examples/invoice.extract.json`](../../examples/invoice.extract.json).

## Post-parse hooks

The config file supports a `hooks` object that defines shell commands to run automatically after successful operations. Each hook type is an array of shell command strings.

### Hook types

| Hook | Trigger | Template variables |
|------|---------|-------------------|
| `postParse` | After a single file parse completes | `{{file}}` (input path), `{{output}}` (output path) |
| `postBatchParse` | After a batch parse completes | `{{inputDir}}`, `{{outputDir}}` |
| `postScreenshot` | After screenshots are generated | `{{file}}` (input PDF), `{{outputDir}}` (screenshot dir) |
| `postConvert` | After a format conversion completes | `{{file}}` (input path), `{{output}}` (output path) |

### Example config with hooks

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
      "curl -s -X POST https://api.example.com/notify --data-urlencode 'dir={{outputDir}}'"
    ]
  }
}
```

### How to execute hooks

1. After a successful parse/batch/screenshot/convert, check if a `liteparse.config.json` exists (passed via `--config` or in the project root).
2. Read the `hooks` object and find the matching hook array for the completed operation.
3. For each command, **escape single quotes** in each value (replace `'` with `'\''`), substitute `{{...}}` template variables with the escaped values, then run via `bash -c`.
4. Report hook results after the main operation report.
5. If a hook fails, report the error but do **not** roll back the parse output.
6. Show the user which hook commands will run before executing them for the first time in a session.

### Hook safety

Template variables are substituted as raw strings into a shell command before `bash -c` runs it.

1. **Single-quote every `{{...}}`** in hook templates (e.g. `'{{file}}'`, `'{{outputDir}}'`) so that spaces and most shell metacharacters are safe. Flag any `{{...}}` that is not single-quoted as a shell-injection risk.
2. **Escape single quotes in values before substitution.** A filename containing `'` will break out of single quotes. Before replacing a `{{...}}` token, replace every `'` in the value with `'\''` (close quote, escaped literal quote, reopen quote). For example, a path `it's here` becomes `it'\''s here`, which is safe inside `'...'`.
3. **Passwords and secrets** should never appear in hook commands. Use environment variables or a separate config file instead — hook commands are visible in the process list (`ps`) and may be logged.

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
| Word | `.doc`, `.docx`, `.docm`, `.odt`, `.rtf` |
| PowerPoint | `.ppt`, `.pptx`, `.pptm`, `.odp` |
| Spreadsheets | `.xls`, `.xlsx`, `.xlsm`, `.ods`, `.csv`, `.tsv` |
| Images | `.jpg`, `.jpeg`, `.png`, `.gif`, `.bmp`, `.tiff`, `.webp`, `.svg` |

Office documents require LibreOffice; images require ImageMagick. LiteParse auto-converts these formats to PDF before parsing.

## Known failure modes

| Symptom | Cause |
|---|---|
| `lit` not found and `npm` not found | Node.js 20.11+ and npm are not installed |
| LibreOffice conversion error on Office input | `libreoffice` is not on PATH |
| ImageMagick error on image input | Neither `magick` nor `convert` is on PATH |
| OCR server connection refused | The `--ocr-server-url` is unreachable; drop the flag to fall back to built-in Tesseract |
| Password-protected PDF fails | `--password <pw>` was not provided (prefer `--config` to avoid leaking passwords in process lists and shell history) |
