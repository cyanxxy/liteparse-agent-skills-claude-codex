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

1. **Resolve inputs**. Accepts individual files, a glob pattern (`./parsed/*.json`), or a directory from batch-parse. If no arguments, ask the user.
2. **Detect format**. All inputs must be the same format (`.txt` or `.json`). If mixed, report and stop.
3. **Merge**:
   - **Text mode**: concatenate in alphabetical order with `===== filename.txt =====` separators.
   - **JSON mode**: build a merged array where each element is `{ "source": "filename.pdf", "pages": [...] }`.
4. **Write output**: to `-o <path>` if given, otherwise `merged-output.txt` or `merged-output.json` in the current directory.
5. **Report**: files merged, total size, output path, any skipped files with reasons.

## Examples

```bash
# Merge text outputs
/merge-parsed ./parsed-docs/report1.txt ./parsed-docs/report2.txt

# Merge a batch-parse output directory into one JSON
/merge-parsed ./batch-output/ -o ./combined.json

# Merge with glob
/merge-parsed ./contracts/*.liteparse.json -o all-contracts.json
```
