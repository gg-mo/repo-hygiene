---
name: writing-docstrings
description: Use when writing or modifying any function, method, class, or source file in a repo using repo-hygiene — public API, internal helpers, and new files all require docstrings; only one-line lambdas and trivial accessors are excluded
---

# Writing Docstrings

## Overview

In a repo using `repo-hygiene`, docstrings on functions/classes and headers on source files are required. **This OVERRIDES the default "no comments unless WHY is non-obvious" stance.** The override exists because docstrings ARE the API contract — a future reader (human or agent) shouldn't have to read the body to know what to expect.

**Core principle:** A docstring earns its place by telling the reader something the signature does NOT tell them.

## When To Use

- Writing a new function, method, or class
- Modifying a function's signature, return type, or contract
- Creating a new source file (write a header)
- Reviewing your own code before commit

## What Earns a Docstring Its Place

A good docstring covers what the signature CAN'T:

1. **Contract** — one-sentence promise, present tense
2. **Parameters** — only those whose meaning isn't obvious from type/name
3. **Returns** — only if non-obvious from the return type
4. **Raises / errors** — failure modes the caller must handle
5. **Caveats** — non-obvious behavior, side effects, performance, thread-safety

Skip anything the signature already conveys. The goal is information, not ceremony.

## Before / After

❌ **Restates the signature (worthless):**
```python
def parse_date(raw: str) -> datetime:
    """Parse a string and return a datetime."""
    return datetime.strptime(raw, "%Y-%m-%d")
```

❌ **Empty ceremony:**
```python
def parse_date(raw: str) -> datetime:
    """
    :param raw: the raw string
    :returns: the parsed datetime
    """
```

✅ **Pins what the signature can't:**
```python
def parse_date(raw: str) -> datetime:
    """Parse a YYYY-MM-DD date. Raises ValueError on any other format."""
    return datetime.strptime(raw, "%Y-%m-%d")
```

## File Headers

Every source file gets a header explaining its role. **A header that just paraphrases the filename is worse than no header.**

❌ **Paraphrases filename:**
```python
"""user_repository.py — the user repository."""
```

✅ **Explains role + boundary:**
```python
"""Persistence layer for User aggregates.

Owns reads/writes to the `users` table. Callers should never use raw SQL
against this table — go through this module so audit logs stay consistent.
Domain type lives in domain/user.py.
"""
```

A good header answers: what does this file own, what's the boundary, who calls it, what's a closely related file?

## Narrow Exceptions

These don't need a docstring (the signature IS the spec):

- One-line lambdas / arrow functions
- Trivial accessors: `def name(self): return self._name`
- Single-expression private helpers under 3 lines

For everything else: if you can't think of a useful one-liner, the function probably does too much or is misnamed. Fix the function before skipping the docstring.

## Quick Reference

| Language | Pattern |
|---|---|
| Python | `"""..."""` after `def`/`class`; module header at top of file |
| TypeScript / JavaScript | JSDoc `/** ... */` above function/class; top-of-file block |
| Go | `// FuncName ...` above declaration; `// Package x ...` at top |
| Rust | `///` for items, `//!` for module-level |
| Java | Javadoc `/** ... */` |

Mirror the repo's existing convention. If unclear, copy a well-documented file in the same language.

## Common Mistakes

| Mistake | Fix |
|---|---|
| Restating the signature in prose | Cut. If the signature says it, the docstring is noise. |
| Documenting every param regardless of clarity | Only the ones whose meaning isn't obvious |
| File header paraphrases filename | Explain role + boundary + collaborators |
| "My default is no-comments" | This skill OVERRIDES that default. Intentional. |
| Skipping docs on "internal" helpers | Internal becomes public. Document the contract anyway. |

## Red Flags — STOP

- "My default is no comments unless WHY is non-obvious" — that default does not apply in repo-hygiene repos
- About to ship a function with no docstring (outside the narrow exceptions)
- About to create a new file with no header
- Writing a docstring that just rephrases the function name — rewrite or you're adding noise
