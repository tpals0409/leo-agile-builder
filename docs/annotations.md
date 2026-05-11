# Annotation Registry

Register annotation fields here before using them in repo-authored source, test,
commentable schema, or migration files.

## Fields

| Annotation | Required? | Value type | Purpose |
|---|---|---|---|
| `@domain` | required | string | DDD domain, listed in `docs/glossary.md`. |
| `@layer` | required | enum: `domain` / `application` / `infrastructure` / `interfaces` | DDD layer. |
| `@purpose` | required | one-line string | Why the file exists. |
| `@sprint` | optional | string | Sprint that created the file. |
| `@agents` | optional | array | Role agents that commonly touch the file. |
| `@stability` | optional | enum: `experimental` / `stable` / `deprecated` | Review signal. |
| `@dependencies` | optional | array of paths | Non-obvious cross-file dependencies. |
| `@last-touched` | optional | sprint id | Most recent sprint that modified the file. |

New field names should be lowercase with no separators. Existing kebab-case
fields such as `@last-touched` are grandfathered; do not add aliases for the
same concept.

Annotation checks should allow every field registered in this table, including
grandfathered kebab-case fields, and reject unregistered fields.

## Syntax

Put one annotation per line in the language's normal comment syntax at the top
of the file.

```ts
/**
 * @domain auth
 * @layer application
 * @purpose Login use-case: verify credentials and issue a session token.
 * @sprint sprint-003-add-login
 */
```

```python
"""
@domain auth
@layer application
@purpose Login use-case: verify credentials and issue a session token.
@sprint sprint-003-add-login
"""
```

## Worked Example

```ts
/**
 * @domain auth
 * @layer application
 * @purpose Login use-case: verify credentials and issue a session token.
 * @sprint sprint-003-add-login
 * @agents [architect, developer, qa-reviewer]
 * @stability stable
 * @dependencies [src/domain/auth/session.ts]
 */
```

An agent working on `billing` can read the header and skip this `auth` file.
