# Standard library boundary

Status: Draft.

## Currently agreed

The core parser should understand generic function calls, member access and
pipeline composition. It should not contain dedicated syntax or semantic
special-cases for individual competitive-programming structures such as
`array`, `tree`, `graph`, `distinct`, `sort` or `shuffle`.

Typical standard-library facilities may include:

```rad
A = array(N, each [1, 100])
P = permutation(N)
T = tree(N)
G = graph(N, M)
```

The built-in library is conceptually divided into four kinds of operations:

- generators;
- constraints/properties;
- transformations;
- projections/views.

The term "plugin" is not used for built-in standard facilities. External
additions may instead be called extensions.

A tree or graph may internally use arrays of edges, but should retain a semantic
tree/graph type and its invariants at the language/library level.

## Array

The `array` constructor belongs to the standard library rather than the core
language.

The type system may still contain an array type such as `array<T>`; the
standard-library constructor is one standard way to create such a value.

For example:

```rad
A = array(N, each [1, 100])
```

constructs an array value whose element type is inferred from the element
expression.

The array library type supplies its own default output behavior. Printing an
array emits its elements using the output behavior of the element type. For an
`array<int>`, serialization of each integer remains a core responsibility.

## Pipeline operations

Pipeline syntax composes an expression with standard-library or extension
operations:

```rad
A = array(N, each [1, 100])
    |> distinct
    |> sort
```

A pipeline operation is not assumed to mean "execute this mutation now". Its
semantic category is supplied by the operation definition.

For example:

- `distinct` is a constraint/property operation;
- `sort` is a transformation;
- `shuffle` is a transformation.

The compiler lowers the pipeline into the canonical execution plan according to
those semantic categories.

## Constraint operations

A constraint operation describes a property that the generated result must
satisfy. It does not itself draw random numbers, mutate values, retry failed
generation, or directly implement the underlying generator.

For example:

```rad
A = array(N, each [1, 100])
    |> distinct
```

means that the resulting array must contain pairwise-distinct elements. It does
not mean "generate an ordinary array and then delete duplicate elements".

The core language does not know the name or meaning of `distinct`.
`distinct` is defined by the standard library and uses the general
constraint/property interface supplied by the core.

## Constraint and generator cooperation

Constraints interact with generators through capabilities and semantic metadata
rather than directly depending on generator implementation details.

Conceptually, a constraint may state requirements such as:

```text
property: distinct
target capability: sequence-like
generation requirement: sample without replacement when necessary
```

A generator advertises capabilities and properties that it can establish or
honor.

For example, the array generator may satisfy `distinct` when its element
generator exposes a finite discrete domain containing enough possible values.

The compiler/planner combines these declarations into a valid execution plan.

If the requested property cannot be implemented or proved valid for a
particular generator, compilation should fail rather than silently falling back
to unbounded rejection sampling.

## Property tracking

The compiler tracks semantic properties separately from ordinary static types.

Operations may declare relationships such as:

```text
permutation:
    establishes distinct

sort:
    establishes sorted
    preserves distinct

shuffle:
    preserves distinct
    does not preserve sorted

tree:
    establishes connected
    establishes acyclic
    establishes simple
```

The exact property vocabulary is defined by the relevant standard-library
facility or extension; the core provides the mechanism for tracking and
propagating properties.

This permits constraints such as `distinct`, `connected` and `simple` to
remain outside the core language while still participating in static analysis
and plan construction.

## Type-specific operations

Operations meaningful only for a particular semantic type should normally be
exposed as members or member functions rather than as core syntax.

Examples:

```rad
T.edge()
T.parent(1)

G.edge()
G.adjacency()
```

Generic transformations that naturally apply to many values may remain ordinary
pipeline/library operations, for example `sort` and `shuffle`.

## Extensions

Extensions should be able to define new:

- generators;
- semantic types;
- constraints/properties;
- transformations;
- projections/views;
- output behavior.

An extension must use the same registration and planning mechanisms as the
standard library. Core semantics must not contain extension-specific
special-cases.

## To decide

- Exact standard generator set.
- Exact pipeline grammar.
- Constraint-operation registration interface.
- Generator capability model.
- Property identity and propagation representation.
- Extension API/ABI.
- Whether library-defined structured types are nominal or structural.
