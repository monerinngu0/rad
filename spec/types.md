# Types

Status: Draft.

## Currently agreed

rad is statically typed. Type annotations are not required for ordinary code;
types should normally be inferred.

Stochasticity is not itself a type.

For example:

```rad
N = 10
M = [1, 10]
```

both bind values of integer type.

The core type model is expected to include at least:

- integer;
- sequence/array-like values;
- record/structured values.

Standard-library facilities may define semantic structured types such as tree,
graph and permutation. Their implementation representation must not erase their
semantic invariants.

## To decide

- Exact integer width.
- Whether `array` is a core type, a standard-library constructor, or both.
- Nominal vs structural typing for library-defined structures.
- Tuple type vs record type.
- Whether output shape is represented in the type system.
- Whether explicit type annotations exist.
