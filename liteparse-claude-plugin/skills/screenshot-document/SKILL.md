---
name: screenshot-document
description: Render document pages as PNG images with LiteParse v2. Use when exporting selected pages as screenshots, generating image previews for visual inspection, or producing page renders for agent reasoning.
argument-hint: "<file> [output-dir]"
allowed-tools: Read Write Bash(which *) Bash(lit *) Bash(liteparse *) Bash(npx -y @llamaindex/liteparse@latest *) Bash(mkdir *) Bash(libreoffice *) Bash(magick *) Bash(convert *)
---

# Screenshot Document

Generate PNG page screenshots from the document at `$0` into the directory `$1` (or a sensible default).

## Steps

1. **Resolve `$0`** relative to the project root. If missing, ask for the document path. If no arguments were passed, ask for a file.
2. **Check file-type dependencies**: Office files require `libreoffice`; image files require `magick` or `convert`; PDFs need no extra dependency.
3. **Resolve `$1`** (output directory). The upstream CLI default is `./screenshots` (relative to the current working directory). For clarity when running on multiple documents, prefer `<basename>-screenshots` if `$1` is omitted, and create it with `mkdir -p` if it does not exist.
4. **Parse extra flags from `$ARGUMENTS`**: `--target-pages "1,3,5"` or `"1-5"`, `--dpi <n>`, `--password <pw>`, `-q`. LiteParse v2 outputs PNG files only.
5. **Choose the CLI**: run `which lit || which liteparse`. If either exists, use that binary as `<cli>` and run `<cli> screenshot <file> -o <output-dir> <flags>`. Otherwise, fall back to `npx -y @llamaindex/liteparse@latest screenshot <file> -o <output-dir> <flags>` (no `lit` prefix under npx). **Always pass the output directory via `-o`** — it is not a positional argument.
6. **Report**:
   - source file,
   - page selection (default: all pages),
   - output directory,
   - PNG output and DPI if specified,
   - number of screenshots written,
   - or the exact CLI error on failure.

## Examples

```
/liteparse:screenshot-document ./docs/report.pdf
/liteparse:screenshot-document ./docs/report.pdf ./report-pages --target-pages "1-3" --dpi 300
/liteparse:screenshot-document ./docs/report.docx ./report-pages
/liteparse:screenshot-document ./encrypted.pdf ./out --password secret   # caution: password visible in ps/history
```

For CLI flag details, see the background `liteparse` skill.
