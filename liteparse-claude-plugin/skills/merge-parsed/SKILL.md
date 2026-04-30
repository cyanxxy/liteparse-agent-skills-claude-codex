---
name: merge-parsed
description: Combine parsed output from multiple files into a single unified text or JSON document. Use after batch-parse or multiple parse-document runs, when consolidating a folder of parsed outputs, concatenating multiple LiteParse JSON files, merging split documents back together, or building a single corpus from many parsed sources.
argument-hint: "<file1> <file2> [... fileN] [-o output]"
allowed-tools: Read Write Bash(cat *) Bash(ls *) Bash(mkdir *)
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

   **Natural sort rule** (applies to both modes): split each filename into alternating alpha and numeric runs, then compare run-by-run — numeric runs as integers, alpha runs as case-insensitive strings. Example ordering: `page-1.txt` < `page-2.txt` < `page-10.txt` < `page-a.txt`. Do not sort lexicographically.

   **Text mode** — concatenate all files in natural sort order by filename. Insert a separator between each:
   ```
   ===== filename.txt =====
   ```
   followed by the file contents, then a blank line.

   **JSON mode** — read each JSON file in natural sort order. Wrap everything in a versioned envelope so downstream consumers can detect the shape:
   ```json
   {
     "version": "1.0",
     "mergedAt": "2026-04-17T12:34:56Z",
     "documentCount": 3,
     "documents": [
       {
         "source": "original-filename.pdf",
         "pages": [ ... ]
       }
     ]
   }
   ```
   For each input file:
   - If the input JSON is already an array of page objects, nest it under `pages`.
   - If it is a single object with a `pages` key, copy `pages` directly.
   - If it is a single object with neither an array nor a `pages` key (unusual), set `pages: [<that object>]` and add `"note": "input lacked pages key; wrapped as single page"` on the document entry.
   - Set `source` to the original filename (not full path). If the input has a `source` field already, preserve it and add the file path as `"file"`.

4. **Write output**:
   - If the user passed `-o <path>`, write there.
   - Otherwise, write to `merged-output.txt` or `merged-output.json` (matching the input format) in the current working directory.

5. **Report**:
   - Number of files merged
   - Total size of merged output
   - Output path
   - Any files that were skipped (empty, unreadable) with reasons

## Examples

This skill is a **file read/transform/write** workflow. Below: three representative invocations and the expected output shape.

### 1. Merge text outputs from two reports → `merged-output.txt`

```
/liteparse:merge-parsed ./parsed-docs/report1.txt ./parsed-docs/report2.txt
```

Output file content:
```
===== report1.txt =====
<contents of report1.txt>

===== report2.txt =====
<contents of report2.txt>

```

### 2. Merge a `batch-parse` JSON output directory → `./combined.json`

```
/liteparse:merge-parsed ./batch-output/ -o ./combined.json
```

Output file content:
```json
{
  "version": "1.0",
  "mergedAt": "2026-04-17T12:34:56Z",
  "documentCount": 12,
  "documents": [
    { "source": "contract-a.pdf", "pages": [ ... ] },
    { "source": "contract-b.pdf", "pages": [ ... ] },
    ...
  ]
}
```

### 3. Merge a glob of saved recipes → `all-contracts.json`

```
/liteparse:merge-parsed ./contracts/*.liteparse.json -o all-contracts.json
```

Same envelope as example 2. If any input fails to read or parse, skip it, list it under the final report's "skipped" section, and continue.
