# Extract Structured Details

Schema fields should use stable `snake_case` names, a type (`string`, `number`, `date`, or `boolean`), `required`, `multiple`, and optional matching guidance. See [`examples/invoice.extract.json`](../../../examples/invoice.extract.json).

Single-value fields should include `value`, `confidence`, and evidence. Mark unresolved fields as `missing`; mark competing matches as `ambiguous` with alternatives. Multi-value fields should emit `values` and source evidence.

JSON is the source of truth; JSONL and CSV are flattened exports.
