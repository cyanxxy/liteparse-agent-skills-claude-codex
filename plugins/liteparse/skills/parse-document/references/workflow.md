# Parse Document Details

Use `parse <file>` for one document. Common flags: `--format json|text`, `-o <file>`, `--target-pages "1-5,10"`, `--no-ocr`, `--ocr-server-url <url>`, `--ocr-language <lang>` (default `eng`), `--dpi <n>`, `--max-pages <n>`, `--preserve-small-text`, `--password <pw>`, `--num-workers <n>`, and `-q`. For local OCR data, prefer `TESSDATA_PREFIX`; pass `--tessdata-path` only when the active CLI help lists it.

If the command streams long text to stdout, show only a bounded preview and suggest `-o`. If JSON is requested, prefer a saved `.liteparse.json` file plus a short preview.
