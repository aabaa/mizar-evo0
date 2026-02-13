# Internal Design Specifications

This directory contains internal implementation specifications that map **1:1** to Rust source files.

## Convention

```
doc/design/<crate-name>/<module>.md  →  crates/<crate-name>/src/<module>.rs
```

## Specification Template

Each internal spec should follow this structure:

```markdown
# Module: <module_name>

## Purpose
Brief description of what this module does.

## Public API
List of public functions, structs, traits, and their signatures.

## Dependencies
- Internal: which other modules this depends on
- External: crate dependencies

## Data Structures
Detailed description of key types.

## Algorithm / Logic
Step-by-step description of the core logic.

## Error Handling
Expected error conditions and how they are handled.

## Tests
Key test scenarios that must pass.
```

## Workflow

1. Write or update the spec in `doc/design/<crate>/<module>.md`
2. Implement (or ask AI to implement) `crates/<crate>/src/<module>.rs`
3. Run tests to verify the implementation matches the spec
4. Keep spec and code in sync — update both together
