---
name: batch-parse
description: Batch parse every supported document in a directory with LiteParse. Use when processing a folder of PDFs or Office files into text or JSON, filtering by file extension, or recursing into nested folders.
argument-hint: "<input-dir> [output-dir]"
allowed-tools: Read Write Bash(which *) Bash(lit *) Bash(npx *) Bash(ls *) Bash(mkdir *)
---

# Batch Parse

Run LiteParse over every supported file in `$0`, writing results to `$1` (or a sensible default if the user omitted it).

## Steps

1. **Resolve `$0`** (input directory) relative to the project root. If it does not exist, stop and report the missing path. If no arguments were passed, ask for an input directory.
2. **Resolve `$1`** (output directory). If the user omitted it, use `<input-dir>-liteparse-output` as a sibling of the input. Create it with `mkdir -p` before running.
3. **Parse extra flags from `$ARGUMENTS`**: `--format json|text`, `--recursive`, `--extension ".pdf"`, `--no-ocr`, `--ocr-server-url <url>`, `--ocr-language <lang>`, `--dpi <n>`, `--max-pages <n>`, `--password <pw>`, `--config <file>`, `-q`.
4. **Choose the CLI**: run `which lit`. If it exists, use `lit batch-parse <input-dir> <output-dir> <flags>`. Otherwise, fall back to `npx -y @llamaindex/liteparse batch-parse <input-dir> <output-dir> <flags>` (no `lit` prefix under npx). Both positional directories are **required** by the upstream CLI — always pass both.
5. **Report**:
   - input directory,
   - output directory,
   - filters applied (recursion, extension),
   - file counts (succeeded / failed) from the CLI's summary,
   - the exact blocking error if the command fails. Do not invent a cause.

## Examples

```
/liteparse:batch-parse ./docs
/liteparse:batch-parse ./docs ./parsed-docs --format json --recursive
/liteparse:batch-parse ./contracts ./contracts-json --extension ".pdf" --no-ocr
/liteparse:batch-parse ./scans ./scans-out --ocr-server-url http://localhost:5000/ocr
```

For CLI flag details and dependency rules, see the background `liteparse` skill.
