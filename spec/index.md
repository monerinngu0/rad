# rad specification draft

Status: Draft, non-normative until explicitly promoted.

This directory contains the working specification for the new rad language.
The previous beta implementation is maintained separately as `rad_old`.

## Documents

1. [Lexical conventions](lexical.md)
2. [Types](types.md)
3. [Expressions](expressions.md)
4. [Definitions](definitions.md)
5. [Evaluation model](evaluation.md)
6. [Randomness and reproducibility](randomness.md)
7. [Output model](output.md)
8. [Standard library boundary](library.md)
9. [Diagnostics](diagnostics.md)

Design rationale and unresolved questions live in [../docs/design.md](../docs/design.md).

## Current core idea

rad is a statically typed language for reproducible random test-case generation.

The core language provides syntax, typing, evaluation, randomness and output.
Common structures and generators such as arrays, trees and graphs are intended
to live in the standard library unless later specification work determines that
a facility must be a core language feature.
