---
name: screenshot-document
description: Render PDF pages as PNG or JPG images with LiteParse. Use when exporting selected pages as screenshots, generating image previews for visual inspection, or producing page renders for agent reasoning.
argument-hint: "<file.pdf> [output-dir]"
allowed-tools: Read Write Bash(which *) Bash(lit *) Bash(npx *) Bash(mkdir *)
---

# Screenshot Document

Generate page screenshots from the PDF at `$0` into the directory `$1` (or a sensible default).

## Steps

1. **Resolve `$0`** relative to the project root. If missing, ask for the PDF path. If no arguments were passed, ask for a file.
2. **Verify it is a PDF**. If the extension is not `.pdf`, tell the user `lit screenshot` only supports PDF input and stop. For other formats, suggest running `/liteparse:convert-format` first (e.g. `docx → pdf`, `pptx → pdf`), then screenshot the resulting PDF.
3. **Resolve `$1`** (output directory). The upstream CLI default is `./screenshots` (relative to the current working directory). For clarity when running on multiple PDFs, prefer `<pdf-basename>-screenshots` if `$1` is omitted, and create it with `mkdir -p` if it does not exist.
4. **Parse extra flags from `$ARGUMENTS`**: `--target-pages "1,3,5"` or `"1-5"`, `--dpi <n>`, `--format png|jpg`, `--password <pw>`, `--config <file>`, `-q`.
5. **Choose the CLI**: run `which lit`. If present, use `lit screenshot <file> -o <output-dir> <flags>`. Otherwise, fall back to `npx -y @llamaindex/liteparse screenshot <file> -o <output-dir> <flags>` (no `lit` prefix under npx). **Always pass the output directory via `-o`** — it is not a positional argument.
6. **Post-screenshot hooks**: if a `liteparse.config.json` exists (passed via `--config` or found at the project root) and contains `hooks.postScreenshot`, execute each command after success. Substitute `{{file}}` with the input PDF path and `{{outputDir}}` with the screenshot directory before running via `bash -c`. Report hook results. If a hook fails, report the error but do not roll back screenshots.
7. **Report**:
   - source file,
   - page selection (default: all pages),
   - output directory,
   - image format and DPI if specified,
   - number of screenshots written,
   - hooks executed and their results,
   - or the exact CLI error on failure.

## Examples

```
/liteparse:screenshot-document ./docs/report.pdf
/liteparse:screenshot-document ./docs/report.pdf ./report-pages --target-pages "1-3" --dpi 300
/liteparse:screenshot-document ./docs/report.pdf --format jpg
/liteparse:screenshot-document ./encrypted.pdf ./out --password secret   # caution: password visible in ps/history; prefer --config
```

For CLI flag details, see the background `liteparse` skill.
