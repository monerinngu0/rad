# Randomness and reproducibility

Status: Draft.

## Currently agreed

rad should reproduce results across operating systems and C++ standard-library
implementations.

Observable output must therefore not depend on implementation-specific behavior
of facilities such as `std::uniform_int_distribution` or `std::shuffle`.

rad will define or adopt and fix:

- a pseudorandom bit stream / engine;
- bounded integer sampling;
- shuffle/permutation sampling where relevant;
- stochastic evaluation order.

Reproducibility is version-scoped.

The intended guarantee is:

> same rad version + same canonical execution plan + same seed
> produces the same output bytes.

Different rad versions are not required to produce identical output for the
same seed.

Textually different source may produce identical output when it lowers to the
same canonical execution plan.

## To decide

- PRNG algorithm.
- Seed width and syntax.
- Bounded integer algorithm.
- Whether standard-library generators each have explicitly specified sampling
  distributions.
- Versioning rules for changes to random algorithms.
