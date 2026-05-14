# Extract Tables Details

Use native `type: table` items first. If none exist, detect grids by clustering aligned text boxes into columns and rows, requiring at least two columns and two rows.

Emit tables in page and top-left reading order. For merged cells, place the value in the top-left cell and leave spanned cells empty. For multi-row headers, combine parent and child labels.

Never fabricate a table when neither native tables nor stable grid structure are present.
