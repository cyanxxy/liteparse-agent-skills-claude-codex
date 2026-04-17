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

Run LiteParse over every supported file in an input directory, writing results to an output directory.

## Steps

1. **Resolve the input directory** relative to the project root. If it does not exist, stop and report the missing path. If no arguments were passed, ask for an input directory.
2. **Resolve the output directory**. The upstream CLI **requires** both positional directories, so always pass both. If the user did not supply an output directory, either default to `<input-dir>-liteparse-output` (and create it with `mkdir -p`) or ask the user — if silently defaulting, mention the chosen path in the report.
   - **If the output directory already exists and is non-empty**: do not silently overwrite. Ask the user whether to (a) reuse it (LiteParse will overwrite per-file outputs), (b) pick a suffixed name like `<input-dir>-liteparse-output-2`, or (c) abort. Only skip the prompt when the user explicitly passed the path as an argument — in that case treat it as consent and mention in the report that pre-existing files may have been overwritten.
3. **Parse extra flags**: `--format json|text`, `--recursive`, `--extension ".pdf"`, `--no-ocr`, `--ocr-server-url <url>`, `--ocr-language <lang>`, `--dpi <n>`, `--max-pages <n>`, `--password <pw>`, `--config <file>`, `-q`.
4. **Choose the CLI**: run `which lit`. If it exists, use `lit batch-parse <input-dir> <output-dir> <flags>`. Otherwise, fall back to `npx -y @llamaindex/liteparse batch-parse <input-dir> <output-dir> <flags>` (no `lit` prefix under npx). Both positional directories are **required** by the upstream CLI — always pass both.
5. **Post-batch-parse hooks**: if a `liteparse.config.json` exists (passed via `--config` or found at the project root) and contains `hooks.postBatchParse`, execute each command in the array after a successful batch. Substitute `{{inputDir}}` with the input directory and `{{outputDir}}` with the output directory before running via `bash -c`. Report hook results. If a hook fails, report the error but do not roll back the batch output. Show the user what hooks will run before first execution.
6. **Report**:
   - input directory,
   - output directory,
   - filters applied (recursion, extension),
   - file counts (succeeded / failed) from the CLI's summary,
   - hooks executed and their results,
   - the exact blocking error if the command fails. Do not invent a cause.

## Examples

```bash
lit batch-parse ./docs ./parsed-docs --format json --recursive
lit batch-parse ./contracts ./contracts-json --extension ".pdf" --no-ocr
lit batch-parse ./scans ./scans-out --ocr-server-url http://localhost:8828/ocr
```

For CLI flag details and dependency rules, see the background `liteparse` skill.
