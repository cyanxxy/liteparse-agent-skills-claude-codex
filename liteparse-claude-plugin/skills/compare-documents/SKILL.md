---
name: compare-documents
description: Parse two documents and produce a structured diff showing what changed between them. Use for diffing contract versions, comparing resume revisions, reviewing spec updates, redline reviews, seeing what changed between two PDFs or Word files, or any "what's different between these two files" request.
argument-hint: "<file-a> <file-b>"
allowed-tools: Read Write Bash(which *) Bash(lit *) Bash(liteparse *) Bash(npx -y @llamaindex/liteparse *) Bash(diff *) Bash(mktemp *) Bash(libreoffice *) Bash(magick *) Bash(convert *)
---

# Compare Documents

Parse two documents with LiteParse and diff their text content.

## Steps

1. **Resolve `$0` and `$1`** as file paths relative to the project root. Both are required. If fewer than two arguments were passed, ask for both file paths.

2. **Check file-type dependencies** for each file:
   - Office files: verify `which libreoffice`.
   - Image files: verify `which magick || which convert`.
   - PDFs: no extra dependency.
   Stop and report if a required tool is missing.

3. **Choose the CLI**: run `which lit || which liteparse`. If either exists, use that binary as `<cli>`. Otherwise, use `npx -y @llamaindex/liteparse`.

4. **Create a per-run temp directory**:
   ```bash
   tmpdir=$(mktemp -d "${TMPDIR:-/tmp}/liteparse-compare.XXXXXX")
   trap 'rm -rf -- "$tmpdir"' EXIT
   ```

5. **Parse both files** to text. Run two parse commands:
   ```bash
   <cli> parse <file-a> --format text -o "$tmpdir/a.txt"
   <cli> parse <file-b> --format text -o "$tmpdir/b.txt"
   ```

6. **Diff the parsed text**:
   ```bash
   diff -u "$tmpdir/a.txt" "$tmpdir/b.txt"
   ```
   If `diff` reports no differences, tell the user the documents are textually identical.

7. **Produce a summary**. After showing the raw diff, provide a concise human-readable summary:
   - Sections added, removed, or modified
   - Key content changes (numbers, names, dates, clauses)
   - Approximate scale of change (minor edits vs. major rewrite)

8. **Optional output file**. If the user passed `-o <path>` in `$ARGUMENTS`, write the unified diff to that path and report it.

9. **Clean up** the temp directory.

10. **Report**:
   - The two files compared
   - Whether they differ or are identical
   - The diff and summary (or output path if written to file)
   - Any parse errors verbatim

## Examples

```
/liteparse:compare-documents ./contracts/v1.pdf ./contracts/v2.pdf
/liteparse:compare-documents ./resume-old.docx ./resume-new.docx -o changes.diff
/liteparse:compare-documents ./spec-draft.pdf ./spec-final.pdf
```

For CLI flag details and dependency rules, see the background `liteparse` skill.
