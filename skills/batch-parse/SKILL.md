---
name: batch-parse
description: Batch parse every supported document in a directory with LiteParse. Use when processing a folder of PDFs or Office files into text or JSON, filtering by file extension, or recursing into nested folders.
compatibility: Requires Node 18+ and `@llamaindex/liteparse`. LibreOffice for Office files. ImageMagick for images.
license: Apache-2.0
metadata:
  author: Local Workspace
  version: "0.2.0"
---

# Batch Parse

Run LiteParse over every supported file in an input directory.

## Steps

1. **Resolve input directory**. If missing, ask the user.
2. **Resolve output directory**. If omitted, use `<input-dir>-liteparse-output`. Create it with `mkdir -p`.
3. **Parse flags**: `--format json|text`, `--recursive`, `--extension ".pdf"`, `--no-ocr`, `--ocr-server-url`, `--ocr-language`, `--dpi`, `--max-pages`, `--password`, `--config`, `-q`.
4. **Choose the CLI**: `which lit` → `lit batch-parse <in> <out>`, otherwise `npx -y @llamaindex/liteparse batch-parse <in> <out>`.
5. **Post-parse hooks**: if a `liteparse.config.json` exists and defines `hooks.postBatchParse`, execute each hook after a successful batch, substituting `{{inputDir}}` and `{{outputDir}}`.
6. **Report**: input/output directories, filters, file counts (succeeded/failed), errors verbatim.

## Examples

```bash
lit batch-parse ./docs ./parsed-docs --format json --recursive
lit batch-parse ./contracts ./contracts-json --extension ".pdf" --no-ocr
lit batch-parse ./scans ./scans-out --ocr-server-url http://localhost:5000/ocr
```
