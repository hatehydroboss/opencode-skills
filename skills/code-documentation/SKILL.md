---
name: code-documentation
description: Write clear code documentation including JSDoc/docstrings, inline comments, architecture decision records, and API docs. Use when adding comments to code, documenting functions or classes, writing ADRs, creating technical documentation, or explaining complex logic. Trigger on "document this", "add comments", "JSDoc", "docstring", "API docs", "architecture decision", "ADR", or "explain this code".
---

# Code Documentation

Document intent, not implementation. Good docs explain *why*, not *what*.

## Docstring / JSDoc Standards

Document the public interface. Skip trivial implementations.

```python
def calculate_shipping(weight_kg: float, destination: str, express: bool = False) -> ShippingRate:
    """Calculate shipping cost based on weight and destination zone.

    Args:
        weight_kg: Total package weight in kilograms.
        destination: ISO 3166-1 alpha-2 country code.
        express: If True, returns express shipping rate.

    Returns:
        ShippingRate with cost and estimated delivery days.

    Raises:
        ValueError: If weight exceeds 30kg (courier limit).

    Example:
        >>> calculate_shipping(2.5, "US")
        ShippingRate(cost=12.99, days=5)
    """
```

### Language-Specific Formats
- **Python**: Google-style or NumPy-style docstrings
- **TypeScript/JS**: JSDoc `@param`, `@returns`, `@throws`, `@example`
- **Go**: standard godoc comments above the function
- **Rust**: `///` doc comments with markdown

## What to Document

| Document | Don't Document |
|----------|---------------|
| Public API surface | Private helpers (unless complex) |
| Why a decision was made | What the code does (readable code > comments) |
| Non-obvious side effects | Obvious parameter meanings |
| Edge cases and constraints | Restating the function name |
| Business rules and invariants | TODO comments without ticket numbers |

## Inline Comments

Use comments to explain surprising or non-obvious code:

```python
# GOOD
# Retry with exponential backoff because the payment gateway
# rate-limits after 5 failures/second (see INC-2024-0142)
for attempt in range(3):
    ...

# BAD
# Loop 3 times
for attempt in range(3):
    ...
```

## Architecture Decision Records (ADR)

For significant technical decisions, write an ADR:

```markdown
# ADR-001: Use PostgreSQL for primary datastore

## Status
Accepted

## Context
We need a relational database with JSON support and strong consistency.
Team has Postgres experience. Data is highly relational.

## Decision
Use PostgreSQL 16 as the primary datastore.

## Consequences
- Must run migrations carefully (no blue-green for schema changes)
- Can use JSONB for flexible attributes
- Team onboarding is faster (existing expertise)
```

## API Documentation

Maintain OpenAPI/Swagger specs alongside code. Document:
- Every endpoint with example requests/responses
- Authentication requirements
- Error codes and their meanings
- Rate limits
- Breaking changes and migration notes

## README Standards

A good README answers:
1. What is this? (one paragraph)
2. How do I install it? (copy-paste commands)
3. How do I use it? (minimal working example)
4. How do I contribute? (or link to CONTRIBUTING.md)

## Documentation Checklist

- [ ] Public functions have docstrings with params, returns, examples
- [ ] Complex logic has a "why" comment
- [ ] ADRs exist for irreversible architectural decisions
- [ ] README enables a new developer to run the project in <5 minutes
- [ ] API spec matches implementation (verify in CI)
- [ ] No stale docs (docs updated in same PR as code changes)
