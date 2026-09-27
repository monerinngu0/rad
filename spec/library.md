# Standard library boundary

Status: Draft.

## Currently agreed

The core parser should understand generic function calls and member access, not
special syntax for every competitive-programming structure.

Likely standard-library facilities include:

```rad
A = array(N, each [1, 100])
P = permutation(N)
T = tree(N)
G = graph(N, M)
```

Likely standard-library categories are:

- generators;
- constraints/properties;
- transformations;
- projections/views.

The term "plugin" is not currently preferred for built-in standard facilities.
External additions may instead be called extensions.

A tree or graph may internally use arrays of edges, but should retain a semantic
tree/graph type and invariants at the language/library level.

## To decide

- Whether `array` is core or standard library.
- Exact standard generator set.
- Constraint syntax.
- Transformation syntax.
- Property propagation model.
- Extension API/ABI.
- Whether library-defined structured types are nominal or structural.
