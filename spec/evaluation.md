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

A structured expression follows the same rule. For example:

```rad
P = {
    x = [1, 10]
    y = [1, 10]
}

A = array(N, each P)
```

binds one already-evaluated structured value to `P`, so all elements of
`A` receive the same value.

To evaluate the stochastic fields independently for every element, the
structured expression itself must appear inside the `each` expression:

```rad
A = array(N, each {
    x = [1, 10]
    y = [1, 10]
})
```

## Pipeline composition

A pipeline composes a value-producing expression with registered operations.

Example:

```rad
A = array(N, each [1, 100])
    |> distinct
    |> sort
```

The pipeline is interpreted semantically before execution. Operations may
represent constraints, transformations, projections or other registered
behaviors.

A constraint stage does not necessarily correspond to a runtime mutation.
For example, `distinct` contributes a requirement to the generated result.
The planner may therefore choose an appropriate generation strategy rather than
generate invalid values and repair them afterwards.

A transformation such as `sort` does represent a transformation in the
canonical plan. Implementations may optimize generation internally only when
the observable behavior required by that canonical plan, including random
consumption and version-scoped reproducibility, is preserved.

## Canonical execution plan

Accepted source should lower to a canonical execution plan describing:

- definition/evaluation order;
- stochastic evaluation points;
- generators and their capabilities;
- semantic constraints/properties;
- transformations;
- output projections;
- random-consumption order.

The plan is also the point where property and capability information supplied
by the standard library or extensions is resolved.

Source programs that normalize to the same canonical plan are intended to have
the same seed behavior within one rad version.

The core does not encode the meaning of individual standard-library constraints
such as `distinct`. It only provides the general planning mechanism through
which registered operations contribute requirements and properties.

## To decide

- Exact evaluation order of function arguments.
- Exact evaluation order inside compound expressions.
- General applicability of `each` beyond arrays.
- Exact pipeline precedence and associativity.
- Which optimizations are permitted without changing the canonical plan.
