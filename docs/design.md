# rad design draft

Status: Draft 0.1 — non-normative design notes for the new rad implementation.

This document records the current direction for rad. It is intentionally separate
from the old beta implementation (`rad_old`). The goal is to settle the language
model first, then turn accepted parts into a normative specification.

## 1. Goal

rad is a small statically typed language for reproducible random test-case
generation.

The main design goals are:

- describe *what values and structures should be generated* rather than how to
  hand-write a generator;
- keep the core language small;
- separate value generation from output formatting;
- make stochastic evaluation explicit and reproducible;
- reject invalid programs before generation whenever possible;
- allow standard generators and extensions without adding special syntax for
  every competitive-programming structure;
- prefer composition over making the language Turing-complete.

rad is primarily aimed at competitive-programming test generation, but the core
language should remain general enough for other structured random-data generation.

## 2. Core language and library boundary

The language is divided conceptually into three layers:

1. **Core language**
   - definitions and name binding;
   - literals and expressions;
   - integer arithmetic;
   - stochastic interval expressions;
   - static types and type checking;
   - function calls;
   - member access;
   - evaluation order;
   - random-number model;
   - output blocks.

2. **Standard library**
   - common generators such as arrays, permutations, trees and graphs;
   - transformations such as shuffle;
   - constraints and properties;
   - standard projections/views such as graph edges.

3. **Extensions**
   - user- or third-party-defined generators, transformations, constraints and
     structured types.

A standard-library function such as `tree` shall not require special parser
syntax merely because it constructs a tree.

## 3. Definitions

A definition has the form:

```rad
name = expression
```

`=` performs binding/definition. It does not itself mean random generation.

Examples:

```rad
N = 10
M = N + 1
X = [1, 100]
```

The right-hand expression is evaluated once when the definition is evaluated,
and the resulting value is bound to the name.

## 4. Random interval expression

A closed integer interval is a stochastic expression:

```rad
[lower, upper]
```

Evaluating it produces one integer uniformly from the inclusive range
`lower..upper`.

Examples:

```rad
N = [1, 10]
X = [1, 10] * 10
Y = [1, 3] + [10, 20]
```

The stochastic property belongs to the expression, not to `=`.

Therefore:

```rad
X = [1, 10]
Y = X + X
```

draws once for `X`, while:

```rad
Y = [1, 10] + [1, 10]
```

draws twice.

Intervals must be finite. An invalid or empty interval is a semantic or
generation error according to whether the compiler can prove the failure
statically.

## 5. Expressions

The core expression system is expected to include at least:

```text
integer literals
name references
(a)
+a, -a
a + b
a - b
a * b
a / b
a % b
a ** b
[a, b]
f(a, b, ...)
a.member
```

Arithmetic is integer arithmetic. Exact overflow, division, remainder and power
semantics will be specified separately.

## 6. Static typing

rad is statically typed.

Type annotations should normally be unnecessary; the compiler infers types from
expressions.

Examples:

```rad
N = [1, 10]              # int
A = array(N, each [1, 10])
G = tree(N)
```

Stochasticity is **not** a type. For example both `10` and `[1, 10]`
have type `int`; they differ in evaluation effect.

The core type model is expected to include at least:

- `int`
- sequence/array types;
- record/structured types.

Standard-library types may include semantic types such as trees, graphs and
permutations. Their implementation may internally use arrays or records, but
their semantic type and invariants must not be erased merely because the
implementation uses an array.

## 7. Function calls and standard generators

Function-call syntax is generic:

```rad
f(arg1, arg2, ...)
```

The parser does not need dedicated syntax for `array`, `tree`, `graph`,
etc.

Examples under consideration:

```rad
A = array(N, each [1, 100])
P = permutation(N)
T = tree(N)
G = graph(N, M)
```

Whether a facility belongs to the standard library or an extension is a library
design issue rather than a grammar issue.

## 8. `each`

`each E` marks `E` for element-wise evaluation by an enclosing element-wise
construction.

For an array constructor:

```rad
A = array(N, each E)
```

the expression `E` is evaluated once for each element.

Example:

```rad
A = array(N, each [1, 10])
```

conceptually performs:

```text
A[0] = evaluate [1, 10]
A[1] = evaluate [1, 10]
...
A[N-1] = evaluate [1, 10]
```

and therefore consumes one random draw per element.

By contrast:

```rad
X = [1, 10]
A = array(N, X)
```

evaluates `X` once at its definition and fills the array with that already
bound value.

For a deterministic expression/reference:

```rad
A = array(N, X)
A = array(N, each X)
```

are semantically equivalent when `X` denotes an already evaluated value.
They should lower to the same canonical execution plan, so the same rad version
and seed produce the same output.

Thus `each` specifies a **re-evaluation boundary**, not a new value type.

The exact generality of `each` outside array-like element-wise constructions
is still open.

## 9. Structured values

Generation and serialization are separate concepts.

For example:

```rad
N = [2, 20]
T = tree(N)
```

creates a tree value. It does not imply how that tree is printed.

A structured value may expose members:

```rad
T.edge
T.edge_count
```

For example, `T.edge` may have a sequence-of-edge-records type. A tree may be
implemented internally using an edge array, but the language-level tree value
retains its tree semantics and invariants.

This separation allows the same generated structure to have multiple output
representations in the future, for example edge lists, parent arrays or
adjacency representations.

## 10. Output

Output is explicitly separated from definitions.

Current proposed syntax:

```rad
print:
N, M
A
G.edge
```

Tentative rules:

- a physical line in the `print:` block corresponds to an output row;
- comma-separated expressions are emitted on the same row;
- a sequence may expand into one row;
- a multi-row projection such as `G.edge` may expand into multiple rows.

The exact output-shape type system and formatting rules remain open and must be
specified before the syntax is considered stable.

## 11. Randomness and reproducibility

rad should define its random behavior independently of operating system and C++
standard-library implementation.

A conforming implementation should therefore not make observable rad output
depend on implementation-specific behavior of facilities such as
`std::uniform_int_distribution` or `std::shuffle`.

The rad specification/runtime will define:

- the pseudorandom engine or equivalent bit stream;
- bounded-integer sampling;
- shuffle/permutation behavior where reproducibility requires it;
- the order in which stochastic expressions consume randomness.

Reproducibility is version-scoped.

The intended rule is approximately:

> The same rad version, the same canonical execution plan, and the same seed
> produce the same output bytes.

Different rad versions are not required to produce the same output for the same
seed.

Source spelling alone is not part of the reproducibility identity. Programs
which normalize to the same canonical execution plan should behave identically.

For example, harmless syntactic differences, comments, or a redundant
`each X` where `X` is an already evaluated deterministic value should not
change output if they lower to the same plan.

Programs with the same mathematical output distribution but different execution
plans are not necessarily equivalent for seed reproducibility.

## 12. Canonical execution plan

Compilation should lower accepted source into a canonical execution plan.

The plan determines:

- binding/evaluation order;
- stochastic evaluation points;
- structured constructors;
- transformations;
- output projections;
- RNG-consumption order.

Canonicalization is the basis for defining semantic equivalence relevant to
seed reproducibility.

Two source programs need not be textually identical to share a plan.

## 13. Non-goals for the initial language

The initial core should not add general-purpose language features merely to make
rad Turing-complete.

In particular, the first specification does not need:

- `while`;
- unrestricted loops;
- recursion;
- user-defined general-purpose functions;
- arbitrary mutable state.

The intended goal is not "every computation can be expressed", but rather
"structured random inputs can be composed from well-defined generators,
transformations and projections".

An extension mechanism can serve as an escape hatch for specialized generators.

## 14. Open design questions

The following are intentionally unresolved:

- exact grammar and operator precedence;
- exact integer width;
- whether arrays are a core type, a standard-library constructor, or both;
- nominal vs structural typing for tree/graph/permutation values;
- standard-library generator API;
- constraint syntax and semantics;
- transformation syntax and semantics;
- extension ABI/API;
- output-shape semantics for nested and multi-row values;
- exact spelling and semantics of comments;
- exact PRNG and bounded-sampling algorithm;
- exact versioning and compatibility policy;
- error taxonomy and diagnostics.

## 15. Working example

A possible future program:

```rad
N = [2, 20]
T = tree(N)
M = T.edge_count
A = array(N, each [1, 100])

print:
N, M
A
T.edge
```

This example is illustrative. `tree`, `array`, member names and output
semantics are not yet normative.

## 16. Design process

The old implementation, now maintained as `rad_old`, is treated as beta
implementation experience rather than the normative definition of the new
language.

The new rad repository should proceed specification-first:

1. record design principles and unresolved questions;
2. write proposals for substantial language/library changes;
3. settle wording and semantics;
4. implement;
5. add conformance tests against the accepted specification.

Once semantics stabilize, normative specification text should be separated from
tutorials, rationale and implementation notes.
