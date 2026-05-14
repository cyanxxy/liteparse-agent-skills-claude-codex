# Merge Parsed Details

Natural sort filenames by alternating text and numeric runs, so `page-2.txt` sorts before `page-10.txt`.

Text output uses separators:

```text
===== filename.txt =====
<contents>
```

JSON output should use an envelope with `version`, `mergedAt`, `documentCount`, and `documents`. Preserve each document source filename and wrap unusual single-object inputs as one page with a note.
