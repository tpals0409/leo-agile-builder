# Code Conventions

This template is language-neutral. Each project chooses its own stack, but must
define enough tooling for agents to verify their work.

## Required Project Choices

- Lint/format/check command, either local or CI-backed.
- Test command and test directory convention.
- DDD layer names used by the project.
- Annotation enforcement for required file headers.

Record these choices in the first sprint design if the project has not already
documented them. When no annotation check exists yet, the first sprint that
touches repo-authored source or tests should add a minimal stack-appropriate
check or document why tooling is out of scope for that sprint.

## File Annotations

Required on repo-authored source, test, commentable schema, and migration files:

```text
@domain
@layer
@purpose
```

Excluded: generated files, vendored files, lockfiles, binaries, docs, sprint
artifacts, and formats that cannot carry comments unless the project defines a
wrapper convention.

## DDD Layers

Default names:

```text
src/
├── domain/
├── application/
├── infrastructure/
└── interfaces/
```

Projects may collapse layers for small or framework-driven codebases, but
`02-design.md` must state the chosen layering.

## Tests

Developer writes tests for approved success criteria unless `02-design.md`
explicitly justifies why no test is needed. QA reviews tests and reports missing
coverage; QA does not edit test files.

## Comments and Documentation

- Delete commented-out code.
- Add comments only when they explain non-obvious intent or constraints.
- Public APIs crossing layer boundaries should have short documentation when
  names and types are not enough.
