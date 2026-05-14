# Batch Parse Details

Use `batch-parse <input-dir> <output-dir>`; both directories are required by LiteParse. Common flags: `--format json|text`, `--recursive`, `--extension ".pdf"`, OCR flags, `--dpi`, `--max-pages`, `--password`, `--config`, and `-q`.

Do not silently reuse a non-empty output directory unless the user supplied it. Report the chosen directory and the CLI's success/failure counts.
