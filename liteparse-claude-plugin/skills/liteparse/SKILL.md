---
name: liteparse
description: Background knowledge for running LiteParse: choosing between installed lit and the npx fallback, verifying LibreOffice or ImageMagick dependencies, CLI flag reference, and failure handling across parsing workflows.
---

# LiteParse Reference

LiteParse is a local, no-cloud document parser. This skill is **reference only** — it provides context for the action skills `/liteparse:parse-document`, `/liteparse:batch-parse`, and `/liteparse:screenshot-document`. Do not execute steps from this skill directly; defer to whichever action skill is active.

## CLI availability

LiteParse ships as the npm package `@llamaindex/liteparse`. Two binaries are registered: `lit` and `liteparse`, pointing to the same script.

- **When `lit` is on PATH**: commands are `lit parse`, `lit batch-parse`, `lit screenshot`.
- **When only `npm` / `npx` is available**: the `lit` prefix is dropped — `npx -y @llamaindex/liteparse parse`, `npx -y @llamaindex/liteparse batch-parse`, `npx -y @llamaindex/liteparse screenshot`.
- **When neither is available**: Node.js 20.11+ and npm are required.

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
- `--no-precise-bbox`
- `--preserve-small-text`
- `--password <pw>` for encrypted PDFs
- `--config <file>` JSON config
- `--recursive` (batch-parse only)
- `--extension ".pdf"` (batch-parse only)
- `-q, --quiet`

**`screenshot`:**
- `-o, --output-dir <dir>` (default `./screenshots`) — output directory is passed via `-o`, not positionally
- `--target-pages "1,3,5"` or `"1-5"`
- `--dpi <n>`
- `--format png|jpg` (default `png`)
- `--password <pw>`
- `--config <file>`
- `-q, --quiet`

## Config file

A sample config ships with this plugin at [`examples/liteparse.config.json`](../../examples/liteparse.config.json). Any command accepts `--config <file>` to load consistent defaults (OCR language, DPI, max pages, output format, etc.).

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
      "curl -X POST https://api.example.com/notify -d '{\"dir\": \"{{outputDir}}\"}'"
    ]
  }
}
```

### How to execute hooks

1. After a successful parse/batch/screenshot/convert, check if a `liteparse.config.json` exists (passed via `--config` or in the project root).
2. Read the `hooks` object and find the matching hook array for the completed operation.
3. For each command, substitute `{{...}}` template variables with actual paths, then run via `bash -c`.
4. Report hook results after the main operation report.
5. If a hook fails, report the error but do **not** roll back the parse output.
6. Show the user which hook commands will run before executing them for the first time in a session.

## Known failure modes

| Symptom | Cause |
|---|---|
| `lit` not found and `npm` not found | Node.js 20.11+ and npm are not installed |
| LibreOffice conversion error on Office input | `libreoffice` is not on PATH |
| ImageMagick error on image input | Neither `magick` nor `convert` is on PATH |
| OCR server connection refused | The `--ocr-server-url` is unreachable; drop the flag to fall back to built-in Tesseract |
| Password-protected PDF fails | `--password <pw>` was not provided |
