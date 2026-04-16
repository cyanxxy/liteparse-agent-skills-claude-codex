---
name: compare-documents
description: Parse two documents and produce a structured diff showing what changed between them. Use for comparing contract versions, resume revisions, spec updates, or any two documents.
---

# Compare Documents

Parse two documents with LiteParse and diff their text content.

## Steps

1. **Resolve the two file paths** relative to the project root. Both are required. If fewer than two paths were provided, ask for both file paths.

2. **Check file-type dependencies** for each file:
   - Office files: verify `which libreoffice`.
   - Image files: verify `which magick || which convert`.
   - PDFs: no extra dependency.
   Stop and report if a required tool is missing.

3. **Choose the CLI**: run `which lit`. If it exists, use `lit parse`. Otherwise, fall back to `npx -y @llamaindex/liteparse parse`.

4. **Parse both files** to text. Create unique temp files to avoid collisions with concurrent runs:
   ```bash
   TMPA="$(mktemp /tmp/liteparse-compare-a-XXXXXX.txt)"
   TMPB="$(mktemp /tmp/liteparse-compare-b-XXXXXX.txt)"
   <cli> parse <file-a> --format text -o "$TMPA"
   <cli> parse <file-b> --format text -o "$TMPB"
   ```

5. **Diff the parsed text**:
   ```bash
   diff -u "$TMPA" "$TMPB"
   ```
   If `diff` reports no differences, tell the user the documents are textually identical.

6. **Produce a summary**. After showing the raw diff, provide a concise human-readable summary:
   - Sections added, removed, or modified
   - Key content changes (numbers, names, dates, clauses)
   - Approximate scale of change (minor edits vs. major rewrite)

7. **Optional output file**. If the user passed `-o <path>` as an additional flag, write the unified diff to that path and report it.

8. **Clean up** the temp files:
   ```bash
   rm -f "$TMPA" "$TMPB"
   ```

9. **Report**:
   - The two files compared
   - Whether they differ or are identical
   - The diff and summary (or output path if written to file)
   - Any parse errors verbatim

## Examples

```bash
# Compare two contract versions
TMPA="$(mktemp /tmp/liteparse-compare-a-XXXXXX.txt)" && TMPB="$(mktemp /tmp/liteparse-compare-b-XXXXXX.txt)"
lit parse ./contracts/v1.pdf --format text -o "$TMPA" && lit parse ./contracts/v2.pdf --format text -o "$TMPB" && diff -u "$TMPA" "$TMPB"; rm -f "$TMPA" "$TMPB"

# Compare resumes and save diff to file
TMPA="$(mktemp /tmp/liteparse-compare-a-XXXXXX.txt)" && TMPB="$(mktemp /tmp/liteparse-compare-b-XXXXXX.txt)"
lit parse ./resume-old.docx --format text -o "$TMPA" && lit parse ./resume-new.docx --format text -o "$TMPB" && diff -u "$TMPA" "$TMPB" > changes.diff; rm -f "$TMPA" "$TMPB"
```

For CLI flag details and dependency rules, see the background `liteparse` skill.
