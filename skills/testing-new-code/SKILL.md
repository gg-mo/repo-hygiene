---
name: testing-new-code
description: Use when adding or modifying business logic, branching code, public API, or fixing a bug in a repo using repo-hygiene — tests for the new code paths must exist before the change is committed, not "later" or "if the user asks"
---

# Testing New Code

## Overview

In a repo using `repo-hygiene`, new code paths get tests before they're committed. **The default is YES; "tests-after-if-asked" is the wrong default.** Manual sanity checks are not tests — they leave no regression net for the next change.

**Core principle:** Untested code is one unrelated refactor away from broken.

**REQUIRED BACKGROUND:** If `superpowers:test-driven-development` is already active in this session, follow its RED-GREEN-REFACTOR cycle and consider this skill satisfied. Use this skill only when TDD is not driving the workflow.

## When To Use

- Wrote a new function with any branching or business logic
- Fixed a bug (regression test required — see below)
- Changed behavior of an existing function
- Added a public API entry point (CLI, HTTP, GraphQL, exported function)

## When NOT to Use (Genuine Triviality Only)

- Pure renames, formatting, or type-annotation-only changes
- Modifying a pass-through wrapper already tested at a higher layer
- Adding a constant
- Documentation-only changes
- Test code itself

**Not a valid exception:** "It's only 3 lines." Three lines of business logic still need a test.

## Bug Fixes Require Regression Tests

A bug fix without a regression test is incomplete. Verify the red-green cycle:

1. Write the test against the **pre-fix** code → should FAIL
2. Apply the fix → test should PASS
3. Revert the fix once → test should fail again (proves it covers the bug)
4. Re-apply the fix → test passes (final state)

If the test passes with the fix reverted, it isn't covering the bug.

## Where Do Tests Live?

Mirror the repo's existing convention. If the repo has no tests yet, default to:

| Language | Default |
|---|---|
| JavaScript / TypeScript | `*.test.ts` or `*.spec.ts` next to source, or `__tests__/` sibling |
| Python | `tests/test_<module>.py` at repo root |
| Go | `*_test.go` next to source |
| Rust | `#[cfg(test)] mod tests { ... }` in same file; `tests/` for integration |
| Java / Kotlin | `src/test/...` mirroring source |

Grep for existing test files first; mirror their style and runner.

## Quick Example

You wrote:
```python
def calculate_price(items: list[Item], discount: Decimal, tax: Decimal) -> Decimal:
    subtotal = sum(i.price for i in items)
    return (subtotal * (1 - discount)) * (1 + tax)
```

Tests to add:
```python
def test_calculate_price_no_discount_no_tax():
    assert calculate_price([Item(price=10)], Decimal(0), Decimal(0)) == Decimal(10)

def test_calculate_price_discount_then_tax():
    assert calculate_price([Item(price=100)], Decimal("0.1"), Decimal("0.1")) == Decimal(99)

def test_calculate_price_empty_cart():
    assert calculate_price([], Decimal(0), Decimal(0)) == Decimal(0)
```

Then run the suite and confirm green before claiming done.

## Common Mistakes

| Mistake | Fix |
|---|---|
| "User didn't ask for tests" | Tests are part of new code by default here |
| "It worked manually" | Manual ≠ test. No regression net. |
| "It's just 3 lines" | 3 lines of logic = at least 3 cases to cover |
| Bug fix without regression test | Add one. Verify red-green by reverting the fix once. |
| Bug-fix test passes on first run | Suspicious — confirm it would fail without the fix |
| Inventing a new test location | Mirror the repo's existing layout first |

## What if the Repo Has No Test Infra?

Don't silently skip. Tell the user explicitly:

> "This repo has no test runner configured. Adding tests for this change would require setting up <jest / pytest / etc.>. Want me to do that, or proceed without tests?"

Then defer to the user. Skipping silently is the failure mode this skill prevents.

## Red Flags — STOP

- About to commit new business logic with no test file changes
- "I'll add tests in a follow-up PR" — that PR usually never lands
- Fixed a bug; about to ship without a regression test
- "User said it's urgent" — urgent ≠ untested
- Repo has no test infra and you're skipping rather than flagging
