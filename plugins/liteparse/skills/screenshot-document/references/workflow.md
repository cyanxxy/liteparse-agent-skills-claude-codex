# Screenshot Details

Use `screenshot <file> -o <output-dir>`. The output directory is passed with `-o`; it is not positional.

Common flags: `--target-pages "1,3,5"` or `"1-5"`, `--dpi <n>`, `--password <pw>`, and `-q`. LiteParse v2 writes PNG files named `page_<n>.png`.

For Office or image inputs, check the same LibreOffice/ImageMagick dependencies used by parsing.
