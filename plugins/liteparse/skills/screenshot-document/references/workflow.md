# Screenshot Details

Use `screenshot <file.pdf> -o <output-dir>`. The output directory is passed with `-o`; it is not positional.

Common flags: `--target-pages "1,3,5"` or `"1-5"`, `--dpi <n>`, `--format png|jpg`, `--password <pw>`, `--config <file>`, and `-q`.

If the input is not a PDF, convert it to PDF first, then screenshot the result.
