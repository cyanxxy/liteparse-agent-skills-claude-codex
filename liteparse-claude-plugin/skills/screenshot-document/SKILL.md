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
2. **Verify it is a PDF**. If the extension is not `.pdf`, tell the user `lit screenshot` only supports PDF input and stop. (For other formats, suggest running `/liteparse:parse-document` to convert first.)
3. **Resolve `$1`** (output directory). If omitted, default to `<pdf-basename>-screenshots` as a sibling of the input file. Create it with `mkdir -p`.
4. **Parse extra flags from `$ARGUMENTS`**: `--target-pages "1,3,5"` or `"1-5"`, `--dpi <n>`, `--format png|jpg`, `--password <pw>`, `--config <file>`, `-q`.
5. **Choose the CLI**: run `which lit`. If present, use `lit screenshot <file> -o <output-dir> <flags>`. Otherwise, fall back to `npx -y @llamaindex/liteparse screenshot <file> -o <output-dir> <flags>` (no `lit` prefix under npx). **Always pass the output directory via `-o`** — it is not a positional argument.
6. **Report**:
   - source file,
   - page selection (default: all pages),
   - output directory,
   - image format and DPI if specified,
   - number of screenshots written, or the exact CLI error on failure.

## Examples

```
/liteparse:screenshot-document ./docs/report.pdf
/liteparse:screenshot-document ./docs/report.pdf ./report-pages --target-pages "1-3" --dpi 300
/liteparse:screenshot-document ./docs/report.pdf --format jpg
/liteparse:screenshot-document ./encrypted.pdf ./out --password secret
```

For CLI flag details, see the background `liteparse` skill.
