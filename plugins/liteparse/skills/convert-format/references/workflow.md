# Convert Format Details

Supported common targets: `pdf`, `docx`, `xlsx`, `pptx`, `html`, `txt`, and `csv`, depending on LibreOffice support for the input.

For concurrent conversions, avoid the shared LibreOffice profile by serializing with `flock` or setting a unique `-env:UserInstallation=file:///tmp/lo-profile-...`.

Conversion is best-effort for complex layouts, macros, embedded objects, and PDF-to-DOCX output.
