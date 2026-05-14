# Compare Documents Details

Create a temp directory, parse both files to text, then run `diff -u`:

```bash
lit parse file-a.pdf --format text -o "$tmpdir/a.txt"
lit parse file-b.pdf --format text -o "$tmpdir/b.txt"
diff -u "$tmpdir/a.txt" "$tmpdir/b.txt"
```

If no differences are found, say the parsed text is identical. Otherwise summarize meaningful additions, removals, changed numbers, dates, names, clauses, and overall scale.
