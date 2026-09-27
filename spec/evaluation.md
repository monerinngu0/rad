# Evaluation model

Status: Draft.

## Currently agreed

Definitions evaluate their right-hand expression once.

Thus:

```rad
X = [1, 10]
Y = X + X
```

contains one stochastic evaluation for `X`.

By contrast:

```rad
Y = [1, 10] + [1, 10]
```

contains two stochastic evaluations.

## `each`

`each E` marks `E` for element-wise re-evaluation by an enclosing
element-wise construction.

For example:

```rad
A = array(N, each [1, 10])
```

evaluates `[1, 10]` separately for every element.

If `X` is an already-bound value, then:

```rad
A = array(N, X)
A = array(N, each X)
```

are intended to be semantically equivalent and should lower to the same
canonical execution plan.

Therefore `each` is a re-evaluation boundary, not a new value type.

## Canonical execution plan

Accepted source should lower to a canonical execution plan describing:

- definition/evaluation order;
- stochastic evaluation points;
- structured constructors;
- transformations;
- output projections;
- random-consumption order.

Source programs that normalize to the same canonical plan are intended to have
the same seed behavior within one rad version.

## To decide

- Exact evaluation order of function arguments.
- Exact evaluation order inside compound expressions.
- General applicability of `each` beyond arrays.
- Which optimizations are permitted without changing the canonical plan.
