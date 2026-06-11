---
name: batch-parse
description: Batch parse every supported document in a directory with LiteParse. Use when processing a folder of PDFs or Office files into text or JSON, filtering by file extension, or recursing into nested folders.
compatibility: Requires LiteParse v2 via `lit`/`liteparse` or Node 18+ with npm for the `npx` fallback. LibreOffice for Office files. ImageMagick for images.
license: Apache-2.0
metadata:
  author: Local Workspace
  version: "0.3.0"
---

# Batch Parse

Run LiteParse over every supported file in an input directory, writing results to an output directory.

## Steps

1. **Resolve the input directory** relative to the project root. If it does not exist, stop and report the missing path. If no arguments were passed, ask for an input directory.
2. **Resolve the output directory**. The upstream CLI **requires** both positional directories, so always pass both. If the user did not supply an output directory, either default to `<input-dir>-liteparse-output` (and create it with `mkdir -p`) or ask the user — if silently defaulting, mention the chosen path in the report.
   - **If the output directory already exists and is non-empty**: do not silently overwrite. Ask the user whether to (a) reuse it (LiteParse will overwrite per-file outputs), (b) pick a suffixed name like `<input-dir>-liteparse-output-2`, or (c) abort. Only skip the prompt when the user explicitly passed the path as an argument — in that case treat it as consent and mention in the report that pre-existing files may have been overwritten.
3. **Parse extra flags**: `--format json|text`, `--recursive`, `--extension ".pdf"`, `--no-ocr`, `--ocr-server-url <url>`, `--ocr-language <lang>` (default `eng`), `--dpi <n>`, `--max-pages <n>`, `--password <pw>`, `--num-workers <n>`, `-q`. If the user needs local OCR language data, prefer `TESSDATA_PREFIX`; pass `--tessdata-path <path>` only after `<cli> batch-parse --help` shows that flag.
4. **Choose the CLI**: run `which lit || which liteparse`. If either exists, use that binary as `<cli>` and run `<cli> batch-parse <input-dir> <output-dir> <flags>`. Otherwise, fall back to `npx -y @llamaindex/liteparse@latest batch-parse <input-dir> <output-dir> <flags>` (no `lit` prefix under npx). Both positional directories are **required** by the upstream CLI — always pass both.
5. **Report**:
   - input directory,
   - output directory,
   - filters applied (recursion, extension),
   - file counts (succeeded / failed) from the CLI's summary,
   - the exact blocking error if the command fails. Do not invent a cause.

## Examples

```bash
lit batch-parse ./docs ./parsed-docs --format json --recursive
lit batch-parse ./contracts ./contracts-json --extension ".pdf" --no-ocr
lit batch-parse ./scans ./scans-out --ocr-server-url http://localhost:8828/ocr
TESSDATA_PREFIX=./tessdata lit batch-parse ./scans ./scans-out --ocr-language fra
```

For CLI flag details and dependency rules, see the background `liteparse` skill.
