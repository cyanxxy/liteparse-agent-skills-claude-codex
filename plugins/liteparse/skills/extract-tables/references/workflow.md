# Extract Tables Details

LiteParse v2 JSON exposes positioned text items, not native table objects. Detect grids by clustering aligned `text_items` or `textItems` into columns and rows, requiring at least two columns and two rows.

Emit tables in page and top-left reading order. For merged cells, place the value in the top-left cell and leave spanned cells empty. For multi-row headers, combine parent and child labels.

Never fabricate a table when no stable grid structure is present.
