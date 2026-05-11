# Operating Principles — Examples

The four principles are defined in `AGENTS.md`. These examples show how to
apply them.

## 1. Think Before Coding

Request: "Export user data."

Do not assume all users, local file output, CSV columns, or sensitive fields.
Ask or record blocking questions for scope, delivery, fields, and volume.

## 2. Simplicity First

Request: "Calculate a percentage discount."

Prefer:

```python
def calculate_discount(amount: float, percent: float) -> float:
    return amount * (percent / 100)
```

Add strategies or configuration only when the approved criteria need them.

## 3. Surgical Changes

Request: "Fix empty emails crashing the validator."

Change the empty-email path. Do not rename adjacent symbols, tighten unrelated
validation, or reformat files unless required by the fix.

## 4. Goal-Driven Execution

Request: "Fix sessions staying valid after password change."

Use a verifiable loop:

1. Add a failing test: change password -> old session returns 401.
2. Implement invalidation.
3. Run the auth test suite.

Good code solves the approved problem without adding future complexity.
