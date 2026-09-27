# Diagnostics

Status: Draft.

## Design direction

rad should reject errors as early as practical and distinguish different failure
classes instead of collapsing all failures into generation-time errors.

Likely categories include:

- lexical error;
- syntax error;
- type error;
- semantic error;
- unsupported static analysis;
- impossible generation request;
- generation-time failure.

## To decide

- Exact categories and exit codes.
- Which errors must be diagnosed statically.
- Source-location requirements.
- Whether diagnostics are part of the compatibility guarantee.
