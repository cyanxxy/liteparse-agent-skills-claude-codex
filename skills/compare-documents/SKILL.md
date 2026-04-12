---
name: compare-documents
description: Parse two documents and produce a structured diff showing what changed between them. Use for comparing contract versions, resume revisions, spec updates, or any two documents.
compatibility: Requires Node 18+ and `@llamaindex/liteparse`. LibreOffice for Office files. ImageMagick for images.
license: Apache-2.0
metadata:
  author: Local Workspace
  version: "0.2.0"
---

# Compare Documents

Parse two documents with LiteParse and diff their text content.

## Steps

1. **Resolve two file paths**. Both required. If fewer than two, ask.
2. **Check file-type dependencies** for each file (LibreOffice for Office, ImageMagick for images).
3. **Choose the CLI**: `which lit` → `lit parse`, otherwise `npx -y @llamaindex/liteparse parse`.
4. **Parse both files** to text:
   ```bash
   <cli> parse <file-a> --format text -o /tmp/liteparse-compare-a.txt
   <cli> parse <file-b> --format text -o /tmp/liteparse-compare-b.txt
   ```
5. **Diff**: `diff -u /tmp/liteparse-compare-a.txt /tmp/liteparse-compare-b.txt`. If no differences, report identical.
6. **Summarize**: sections added/removed/modified, key content changes (numbers, names, dates), scale of change.
7. **Optional output**: if `-o <path>` given, write the unified diff there.
8. **Clean up** temp files.
9. **Report**: files compared, whether they differ, diff + summary or output path, parse errors verbatim.

## Examples

```bash
/compare-documents ./contracts/v1.pdf ./contracts/v2.pdf
/compare-documents ./resume-old.docx ./resume-new.docx -o changes.diff
/compare-documents ./spec-draft.pdf ./spec-final.pdf
```
