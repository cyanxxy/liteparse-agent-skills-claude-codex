---
name: merge-parsed
description: Combine parsed output from multiple files into a single unified text or JSON document. Use after batch-parse or multiple parse-document runs when you need one consolidated output.
compatibility: Requires parsed output files from LiteParse (text or JSON).
license: Apache-2.0
metadata:
  author: Local Workspace
  version: "0.2.0"
---

# Merge Parsed

Combine the parsed output of multiple documents into a single file.

## Steps

1. **Resolve inputs**. Inputs can be:
   - Individual files: `file1.txt file2.txt file3.json`
   - A glob pattern: `./parsed/*.json`
   - A directory produced by `batch-parse`: `./output-dir/`
   If no arguments were passed, ask the user what files or directory to merge.

2. **Detect format**. Look at file extensions to determine whether the inputs are text (`.txt`) or JSON (`.json`, `.liteparse.json`). All inputs must share the same format. If mixed, report the conflict and stop.

3. **Merge**:

   **Text mode** — concatenate all files in **natural sort order** by filename (so `page-2` comes before `page-10`). Insert a separator between each:
   ```
   ===== filename.txt =====
   ```
   followed by the file contents, then a blank line.

   **JSON mode** — read each JSON file. Build a merged JSON array where each element is:
   ```json
   {
     "source": "original-filename.pdf",
     "pages": [ ... ]
   }
   ```
   If the input JSON is already an array of page objects, nest it under `pages`. If it is a single object with a `pages` key, use it directly.

4. **Write output**:
   - If the user passed `-o <path>`, write there.
   - Otherwise, write to `merged-output.txt` or `merged-output.json` (matching the input format) in the current working directory.

5. **Report**:
   - Number of files merged
   - Total size of merged output
   - Output path
   - Any files that were skipped (empty, unreadable) with reasons

## Examples

```bash
# Merge text outputs
cat ./parsed-docs/report1.txt ./parsed-docs/report2.txt   # then add separators and write merged-output.txt

# Merge a batch-parse output directory into one JSON
ls ./batch-output/   # then read each .json file and write ./combined.json

# Merge with glob
ls ./contracts/*.liteparse.json   # then read each and write all-contracts.json

# Merge a set of text files
ls ./notes/*.txt   # then concatenate with separators and write merged-notes.txt
```

For details on the parsed output formats this skill consumes, see the background `liteparse` skill.
