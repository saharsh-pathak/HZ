---
name: HZ
description: >-
  Activate this skill whenever the user asks to "humanize" code, make code
  look human-written, bypass AI code detection, or rewrite code so it passes
  AI code analyzers (e.g., GPTZero, Copyleaks, Sapling, ZeroGPT, ContentAtScale,
  Originality.ai, or similar). Primarily targets Python but the principles
  apply to any language. The agent MUST rewrite the provided code following
  every rule in this skill — no shortcuts, no partial applications.
---

# HZ — Human-Zone Code Rewriter

> **Mission**: Rewrite AI-generated Python code so that it is classified as
> **human-written** by every major AI code detection tool (GPTZero, Copyleaks,
> Sapling, ZeroGPT, Originality.ai, ContentAtScale, and others).

AI detectors evaluate code on three axes:

| Axis | What Detectors Look For |
|:-----|:------------------------|
| **Token Perplexity** | AI code is *predictable* — each token is the statistically most likely next token. Human code is *surprising*. |
| **AST Regularity** | AI code produces perfectly balanced, textbook-clean syntax trees. Human code has pragmatic irregularities. |
| **Stylistic Uniformity** | AI code is monotonously consistent in naming, spacing, and structure. Human code drifts naturally. |

The rewrite MUST attack all three axes simultaneously.

---

## Mandatory Rewrite Rules

When the user triggers this skill, you MUST apply **every single rule below**
to the code. Do not skip any rule. Do not ask which rules to apply — apply
them all. The output must compile and behave identically to the input.

---

### RULE 1 — Obliterate AI Comments

AI models write comments that narrate *what* code does line-by-line.
Humans comment only on *why*, *gotchas*, *edge cases*, or *business rules*.

**Actions:**
- **DELETE** every comment that restates the code (e.g., `# iterate through the list`).
- **DELETE** every docstring that is a generic boilerplate summary.
- **KEEP or ADD** comments only for:
  - Non-obvious business logic (`# HIPAA requires we mask SSN after 3rd digit`)
  - Workarounds or hacks (`# pandas 2.1 bug — .loc silent fail on MultiIndex`)
  - Performance notes (`# O(n²) but n < 50 in practice, not worth optimizing`)
  - TODO/FIXME markers with context (`# TODO(sahara): switch to async after v2 migration`)
- **Rewrite remaining docstrings** in a terse, pragmatic voice — not a textbook
  explanation. Use sentence fragments, not full formal sentences.

```python
# BAD (AI-style)
def calculate_total_price(items):
    """Calculate the total price of all items in the list."""
    total = 0  # Initialize total to zero
    for item in items:  # Iterate through each item
        total += item.price  # Add item price to total
    return total  # Return the total price

# GOOD (Human-style)
def total_price(items):
    """Sum of item prices. Excludes tax — see apply_tax()."""
    return sum(i.price for i in items)
```

---

### RULE 2 — Rename Everything with Domain Flavor

AI uses generic, textbook identifiers: `data`, `result`, `items`, `temp`,
`process_data`, `calculate_sum`, `item_list`, `output`.

Humans use **domain-specific**, **abbreviated**, or **idiosyncratic** names.

**Actions:**
- Replace generic names with domain terms from the user's codebase context.
- Use common human abbreviations: `cfg`, `ctx`, `req`, `resp`, `buf`, `idx`,
  `acc`, `err`, `msg`, `cb`, `fn`, `src`, `dst`, `prev`, `cur`, `nxt`.
- Use underscore-prefixed names for internal helpers: `_parse_row`, `_validate`.
- Vary naming style slightly — not every name needs to follow the exact same
  pattern. Humans are inconsistent.

```python
# BAD                          # GOOD
result = []                    matches = []
data = fetch_data()            raw_payload = fetch_data()
temp = process(data)           parsed = _parse_payload(raw_payload)
item_list = get_items()        sku_batch = pull_skus()
```

---

### RULE 3 — Break Structural Predictability

AI code follows textbook patterns: standard `if/elif/else` chains, perfectly
nested loops, one-pattern-fits-all. Humans mix idioms pragmatically.

**Actions:**
- **Guard clauses / early returns** instead of deep nesting:
  ```python
  # Instead of:
  if user:
      if user.is_active:
          if user.has_permission:
              do_thing()

  # Write:
  if not user or not user.is_active:
      return
  if not user.has_permission:
      raise PermissionError(f"denied for {user.id}")
  do_thing()
  ```

- **Inline expressions** where a human would:
  ```python
  # Instead of:
  filtered = []
  for x in items:
      if x.active:
          filtered.append(x)

  # Write:
  active = [x for x in items if x.active]
  ```

- **Mix styles within the same file** — use a comprehension in one function,
  a regular loop in another. Humans don't robotically apply the same pattern
  everywhere.

- **Reorder independent statements** — swap the order of variable declarations,
  imports, or independent operations where it doesn't affect logic.

- **Use Python idioms** naturally but not exhaustively:
  - `any()` / `all()` instead of flag-loop patterns
  - `collections.defaultdict` or `Counter` instead of manual dicts
  - `enumerate()` instead of manual index tracking
  - Walrus operator (`:=`) occasionally (not everywhere)
  - `f-strings` with inline expressions
  - `pathlib.Path` over `os.path`

---

### RULE 4 — Inject Production Realism

AI-generated code lives in a vacuum. Real human code shows evidence of a
living codebase.

**Actions:**
- **Custom exceptions** instead of generic ones:
  ```python
  # BAD
  raise ValueError("not found")

  # GOOD
  raise TenantNotFoundError(f"tenant_id={tid!r} missing from registry")
  ```

- **Logging** — add `logging.getLogger(__name__)` calls where appropriate.
  Use `logger.debug()` for flow tracing, `logger.warning()` for edge cases.

- **Constants** — extract magic numbers/strings into module-level constants
  with `UPPER_SNAKE_CASE` names:
  ```python
  MAX_RETRY_ATTEMPTS = 3
  DEFAULT_TIMEOUT_SEC = 30
  ```

- **Type hints** — add them pragmatically, not exhaustively. Humans type-hint
  function signatures but often skip obvious local variables:
  ```python
  def fetch_user(uid: str, *, include_deleted: bool = False) -> User | None:
  ```

- **Defensive code** — add bounds checks, `assert` statements, or fallback
  values that reflect real-world paranoia:
  ```python
  page_size = min(raw_page_size, 200)  # cap to avoid OOM on large queries
  ```

---

### RULE 5 — Sabotage Token Predictability

This is the most critical rule for defeating perplexity-based detectors.
Every token the AI would predict as "most likely next" must be replaced with
a valid but *less predictable* alternative.

**Actions:**
- **Vary string quote styles** — use double quotes sometimes, single quotes
  other times. Humans are inconsistent.
- **Vary whitespace around operators** — occasionally add or omit a space
  where PEP-8 allows flexibility (e.g., keyword arguments, slice notation).
- **Use alternative expressions** for common patterns:
  ```python
  # Instead of the most predictable:
  if len(items) == 0:       →   if not items:
  x = x + 1                →   x += 1
  if x == True:             →   if x:
  for i in range(len(lst)): →  for i, val in enumerate(lst):
  ```
- **Break long lines differently** — sometimes use backslash continuation,
  sometimes use parenthetical grouping, sometimes restructure the expression.
- **Vary import style** — mix `import module` with `from module import name`
  across the file where both are reasonable.
- **Occasional one-liner** where a human would write one, mixed with
  multi-line blocks elsewhere.

---

### RULE 6 — Introduce Controlled Imperfection

Humans are imperfect. Perfectly clean code is a red flag.

**Actions:**
- Leave one or two **minor style inconsistencies** (e.g., an extra blank line
  somewhere, a slightly longer line that a human wouldn't bother reformatting).
- Add a **TODO or FIXME** comment that reflects genuine future work:
  ```python
  # FIXME: this silently drops unicode surrogates — revisit for i18n
  ```
- Use a **slightly suboptimal but readable** approach in one place instead of
  the theoretically perfect solution. Humans trade elegance for readability.
- Occasionally use a **ternary expression** in one place and an `if/else`
  block for a similar pattern elsewhere.

---

### RULE 7 — Restructure Function and Class Design

AI tends to create functions that map 1:1 to the prompt description.
Humans factor code based on reuse, testing, and cognitive load.

**Actions:**
- **Extract helpers**: Pull repeated logic into small private functions
  prefixed with `_`.
- **Merge trivial functions**: If the AI created a one-liner function that's
  called once, inline it.
- **Reorder methods**: Don't keep the "obvious" top-down order. Humans often
  put the most important public method first, then helpers, then dunder
  methods — or the reverse. Pick a natural-looking order.
- **Add `__repr__`** or `__str__` to classes — humans almost always add these
  for debugging convenience.

---

### RULE 8 — Humanize Error Handling

AI error handling is either absent or generic `try/except Exception`.
Human error handling is specific, layered, and often paranoid.

**Actions:**
- Catch **specific exceptions** (`KeyError`, `ConnectionTimeout`, `JSONDecodeError`).
- Add **context to re-raised exceptions**:
  ```python
  except KeyError as exc:
      raise ConfigError(f"missing key {key!r} in {path}") from exc
  ```
- Use **`else` and `finally`** clauses in `try` blocks where appropriate —
  AI almost never uses `else` on `try`.
- Add **retry logic** or **fallback values** for I/O operations.

---

## Output Requirements

After applying ALL rules above, the rewritten code MUST:

1. **Be functionally identical** — same inputs produce same outputs.
2. **Pass `python -m py_compile`** — no syntax errors.
3. **Look like it was written by a mid-to-senior Python developer** working on
   a real production codebase — not a student, not a textbook, not an AI.
4. **Read naturally** — the code should feel like it belongs in a real project
   with history, context, and pragmatic tradeoffs.

Present the full rewritten code to the user. Do NOT present a diff or partial
snippets — output the complete rewritten file(s).

---

## Verification Checklist

Before presenting the output, mentally verify each item:

- [ ] No line-by-line narrator comments remain
- [ ] No generic variable names (`data`, `result`, `temp`, `items`, `output`)
- [ ] At least 2 different control-flow styles used across the file
- [ ] At least 1 custom exception or domain-specific error
- [ ] At least 1 logging call or assertion
- [ ] At least 1 constant extracted to module level
- [ ] Quote styles vary (mixed `'` and `"`)
- [ ] At least 1 TODO/FIXME that references a realistic concern
- [ ] Function/method order is not the "obvious" textbook order
- [ ] Type hints on function signatures (but not on every local variable)
- [ ] Code compiles and is functionally equivalent to the original

---

## Additional References

For deeper guidance on specific transformation techniques, see:
- [Naming Strategies](./references/naming_strategies.md)
- [Structural Patterns](./references/structural_patterns.md)
