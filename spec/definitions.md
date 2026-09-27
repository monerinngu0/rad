# Definitions

Status: Draft.

## Currently agreed

A definition has the form:

```rad
name = expression
```

`=` is binding/definition. It does not itself mean random generation.

The right-hand expression is evaluated once and the resulting value is bound to
the name.

Examples:

```rad
N = 10
M = N + 1
X = [1, 100]
```

This distinction allows stochastic expressions to compose naturally.

## To decide

- Whether rebinding is forbidden.
- Scope rules.
- Forward references.
- Whether definitions are strictly evaluated in source order.
- Whether there are local scopes in the initial language.
