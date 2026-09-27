# Output model

Status: Draft.

## Currently proposed

Generation and serialization are separate.

Example:

```rad
N = [2, 20]
T = tree(N)
M = T.edge_count

print:
N, M
T.edge
```

Tentative interpretation:

- `print:` begins the output description;
- comma-separated expressions appear on the same output row;
- source newlines inside the block separate output rows;
- a sequence may expand on one row;
- a multi-row projection such as `T.edge` may expand to multiple rows.

A generated tree does not intrinsically imply edge-list output. Other
projections may later expose parent arrays or adjacency representations.

## To decide

- Exact grammar of `print:`.
- Whether indentation is significant.
- How scalar, sequence, record and multi-row values serialize.
- Whether separators can be customized.
- Whether multiple print blocks are allowed.
- Whether printing is itself part of the canonical execution plan.
