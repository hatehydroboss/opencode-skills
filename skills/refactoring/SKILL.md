---
name: refactoring
description: Safely refactor and restructure code without changing behavior. Use when code is messy, duplicated, hard to test, too long, poorly named, or when extracting methods/classes, inverting dependencies, or simplifying complex logic. Trigger on "refactor", "clean up", "restructure", "extract method", "extract class", "technical debt", "code smell", or "too complex".
---

# Refactoring

Improve internal structure without changing external behavior. Every refactoring must preserve observable behavior.

## Golden Rules

1. **Small steps** — one refactoring at a time, verify between each
2. **Tests first** — if no tests exist, write characterization tests before touching code
3. **Commit often** — each safe step is a commit; easy to revert
4. **No behavior changes** — behavior changes are separate commits from refactoring

## Safe Refactoring Sequence

1. Understand current behavior (read code + tests)
2. Write/verify tests cover the behavior
3. Make one small structural change
4. Run tests
5. Repeat

## Code Smells and Fixes

| Smell | Refactoring |
|-------|------------|
| Long method (>20 lines) | Extract Method |
| Long class | Extract Class / Extract Interface |
| Duplicate code | Extract Method → Pull Up (if inheritance) |
| Long parameter list | Introduce Parameter Object / Preserve Whole Object |
| Feature envy (uses other class's data) | Move Method |
| Data clumps (always-tied data) | Extract Class |
| Primitive obsession | Replace Primitive with Value Object |
| Switch statements on type | Polymorphism / Strategy pattern |
| Divergent change (same class, different reasons to change) | Split into separate classes |
| Speculative generality | Collapse Hierarchy, Inline Class |

## Extract Method Template

```python
# Before
def process_order(order):
    total = 0
    for item in order.items:
        if item.taxable:
            total += item.price * 1.08
        else:
            total += item.price
    discount = total * 0.1 if order.is_vip else 0
    total -= discount
    # ... 30 more lines

# After
def process_order(order):
    subtotal = calculate_subtotal(order.items)
    discount = calculate_discount(subtotal, order.is_vip)
    return apply_final_total(subtotal, discount)
```

## Dependency Inversion

Before (high-level depends on low-level):
```python
class OrderService:
    def __init__(self):
        self.repo = MySQLOrderRepository()  # hard dependency
```

After (both depend on abstraction):
```python
class OrderService:
    def __init__(self, repo: OrderRepository):  # interface
        self.repo = repo
```

## Simplification Techniques

- **Replace nested conditionals with guard clauses** — return early
- **Replace conditional with polymorphism** — when switching on types
- **Decompose conditional** — extract if/else into named methods
- **Introduce explaining variable** — complex expressions get named intermediates
- **Inline variable** — when the name adds no clarity

## When to Stop

- Tests still pass
- Cyclomatic complexity reduced
- No duplication between old and new code
- Reader can understand intent without comments

## What NOT to Do

- Don't refactor and add features in the same commit
- Don't rewrite from scratch ("big bang refactoring")
- Don't "improve" naming across an entire codebase in one pass — do it file by file
- Don't remove "dead code" without checking for dynamic references
