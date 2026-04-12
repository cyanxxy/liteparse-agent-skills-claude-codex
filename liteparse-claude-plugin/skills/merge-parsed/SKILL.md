---
name: merge-parsed
description: Combine parsed output from multiple files into a single unified text or JSON document. Use after batch-parse or multiple parse-document runs when you need one consolidated output.
argument-hint: "<file1> <file2> [... fileN] [-o output]"
allowed-tools: Read Write Bash(which *) Bash(lit *) Bash(npx *) Bash(cat *) Bash(ls *) Bash(mkdir *)
---

# Merge Parsed

Combine the parsed output of multiple documents into a single file.

## Steps

1. **Resolve inputs from `$ARGUMENTS`**. Inputs can be:
   - Individual files: `file1.txt file2.txt file3.json`
   - A glob pattern: `./parsed/*.json`
   - A directory produced by `/liteparse:batch-parse`: `./output-dir/`
   If no arguments were passed, ask the user what files or directory to merge.

2. **Detect format**. Look at file extensions to determine whether the inputs are text (`.txt`) or JSON (`.json`, `.liteparse.json`). All inputs must share the same format. If mixed, report the conflict and stop.

3. **Merge**:

   **Text mode** — concatenate all files in alphabetical order by filename. Insert a separator between each:
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

```
/liteparse:merge-parsed ./parsed-docs/report1.txt ./parsed-docs/report2.txt
/liteparse:merge-parsed ./batch-output/ -o ./combined.json
/liteparse:merge-parsed ./contracts/*.liteparse.json -o all-contracts.json
/liteparse:merge-parsed ./notes/*.txt -o merged-notes.txt
```
