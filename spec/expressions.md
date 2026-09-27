# Expressions

Status: Draft.

## Currently agreed

A right-hand side is an expression. Expressions may be deterministic or
stochastic.

Expected core forms include:

```text
integer literal
name
(expr)
+expr
-expr
expr + expr
expr - expr
expr * expr
expr / expr
expr % expr
expr ** expr
[lower, upper]
f(arg1, arg2, ...)
expr.member
each expr
```

## Closed random interval

```rad
[lower, upper]
```

is a stochastic integer expression. Evaluation draws one integer uniformly from
the inclusive finite interval.

Therefore:

```rad
N = [1, 10] * 10
```

is valid in the intended language model.

## Function calls

Calls use generic syntax:

```rad
f(a, b, ...)
```

The parser should not need dedicated syntax for standard-library names such as
`array`, `tree` or `graph`.

## Member access

Structured values may expose members:

```rad
G.edge
G.edge_count
```

## To decide

- Exact precedence and associativity.
- Exact arithmetic semantics.
- Exact meaning and grammar category of `each`.
- Whether indexing such as `A[i]` exists.
- Whether literals exist for arrays, records or tuples.
