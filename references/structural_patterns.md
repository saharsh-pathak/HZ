# Structural Patterns — Deep Reference

This document provides exhaustive structural transformation patterns for the
HZ skill, focused on Python.

---

## 1. Control Flow Transformations

### Flatten Nested Conditionals (Guard Clauses)

AI-generated code loves deep nesting. Human code uses early returns.

```python
# ❌ AI Pattern — deeply nested
def process_order(order):
    if order is not None:
        if order.status == "pending":
            if order.total > 0:
                if order.customer.is_verified:
                    return finalize(order)
                else:
                    return {"error": "unverified"}
            else:
                return {"error": "empty order"}
        else:
            return {"error": "not pending"}
    else:
        return {"error": "no order"}

# ✅ Human Pattern — guard clauses
def process_order(order):
    if order is None:
        return {"error": "no order"}
    if order.status != "pending":
        return {"error": "not pending"}
    if order.total <= 0:
        return {"error": "empty order"}
    if not order.customer.is_verified:
        return {"error": "unverified"}

    return finalize(order)
```

### Mix Comprehensions with Loops

Don't uniformly convert everything to comprehensions OR keep everything as
loops. **Mix them** based on complexity:

```python
# Simple filter → comprehension
active_users = [u for u in users if u.is_active]

# Complex operation → regular loop (humans don't shove everything into a comprehension)
enriched = []
for u in active_users:
    profile = fetch_profile(u.id)
    if profile and profile.last_login:
        u.score = compute_engagement(profile)
        enriched.append(u)
```

### Use `any()` / `all()` to Replace Flag Loops

```python
# ❌ AI pattern
has_admin = False
for user in users:
    if user.role == "admin":
        has_admin = True
        break

# ✅ Human pattern
has_admin = any(u.role == "admin" for u in users)
```

### Walrus Operator (Sparingly)

Use `:=` in 1-2 places per file max. Humans adopt new syntax gradually.

```python
# Natural usage in a while-read loop
while chunk := stream.read(8192):
    hasher.update(chunk)

# Natural usage in a conditional
if (m := re.match(r"v(\d+)", tag)):
    version = int(m.group(1))
```

---

## 2. Function Design Transformations

### Inline Trivial Functions

AI creates tiny functions called exactly once. Humans inline them.

```python
# ❌ AI — unnecessary abstraction
def is_valid_email(email):
    return "@" in email

def process_user(user):
    if is_valid_email(user.email):
        ...

# ✅ Human — inline the obvious check
def process_user(user):
    if "@" not in user.email:  # basic sanity check, real validation in forms
        raise ValueError(f"bad email for uid={user.id}")
    ...
```

### Extract Meaningful Helpers

Conversely, when there's repeated logic, extract it into a private helper:

```python
def _retry(fn, *, attempts=3, backoff=1.5):
    """Exponential backoff wrapper. Raises last exc on exhaustion."""
    last_err = None
    for attempt in range(attempts):
        try:
            return fn()
        except (ConnectionError, TimeoutError) as exc:
            last_err = exc
            sleep(backoff ** attempt)
    raise last_err
```

### Keyword-Only Arguments

Humans use `*` to force keyword-only args for clarity:

```python
def create_user(name: str, email: str, *, role: str = "member", notify: bool = True):
    ...
```

---

## 3. Data Structure Patterns

### Use `collections` Module

```python
from collections import defaultdict, Counter, namedtuple

# Instead of manual dict accumulation:
freq = Counter(word for line in text for word in line.split())

# Instead of checking key existence:
groups = defaultdict(list)
for item in records:
    groups[item.category].append(item)
```

### Use `dataclasses` or `NamedTuple` Over Raw Dicts

```python
from dataclasses import dataclass, field

@dataclass(frozen=True)
class CacheEntry:
    key: str
    value: bytes
    ttl_sec: int = 300
    tags: list[str] = field(default_factory=list)

    def is_expired(self, now: float) -> bool:
        return (now - self.created_at) > self.ttl_sec
```

---

## 4. Import Ordering and Style

AI imports are mechanically sorted. Human imports have a pragmatic flow:

```python
# stdlib — but humans sometimes group related ones on one line
import os
import sys
import logging
from pathlib import Path
from datetime import datetime, timedelta

# third-party
import requests
from sqlalchemy import create_engine, Column, String

# local — humans sometimes use relative imports in packages
from .config import settings
from .exceptions import TenantNotFoundError
```

**Mix `import X` with `from X import Y`** — don't use only one style.
Humans tend to do `import os` but `from pathlib import Path`.

---

## 5. String and Formatting Patterns

### Quote Style Variation

```python
name = "Alice"                    # double quotes for visible strings
query = 'SELECT * FROM users'     # single quotes for SQL/code-like strings
msg = f"Hello {name}, welcome!"   # f-strings for interpolation
template = "Dear {name},"         # .format() for templates (occasionally)
```

### Multi-line Strings

Humans use different approaches in different places:

```python
# Parenthetical concatenation (common in real codebases)
query = (
    "SELECT u.id, u.name "
    "FROM users u "
    "WHERE u.active = true"
)

# Triple-quote (for larger blocks)
HELP_TEXT = """\
Usage: manage.py <command> [options]

Commands:
  migrate    Run database migrations
  serve      Start the dev server
"""
```

---

## 6. Error Handling Sophistication

### Use `try/except/else/finally`

AI almost never uses `else` on `try`. Humans do:

```python
try:
    conn = db.connect(dsn, timeout=DEFAULT_TIMEOUT_SEC)
except ConnectionError:
    logger.warning("db unreachable, falling back to cache")
    return _read_from_cache(key)
else:
    # only runs if no exception — human developers use this
    result = conn.execute(query)
finally:
    metrics.track("db_query", duration=timer.elapsed())
```

### Exception Chaining

```python
try:
    cfg = json.loads(raw)
except json.JSONDecodeError as exc:
    raise ConfigParseError(f"invalid JSON in {path}") from exc
```

### Context Managers for Cleanup

```python
from contextlib import contextmanager

@contextmanager
def temp_directory():
    path = Path(tempfile.mkdtemp())
    try:
        yield path
    finally:
        shutil.rmtree(path, ignore_errors=True)
```

---

## 7. Decorator and Metaprogramming Patterns

Humans use decorators for cross-cutting concerns:

```python
import functools
import time

def timed(fn):
    @functools.wraps(fn)
    def wrapper(*args, **kwargs):
        t0 = time.perf_counter()
        rv = fn(*args, **kwargs)
        elapsed = time.perf_counter() - t0
        if elapsed > 0.5:
            logger.info(f"{fn.__name__} took {elapsed:.2f}s")
        return rv
    return wrapper
```

---

## 8. File and Path Handling

Humans prefer `pathlib` over `os.path`:

```python
from pathlib import Path

base_dir = Path(__file__).resolve().parent
config_path = base_dir / "config" / "settings.toml"

if not config_path.exists():
    raise FileNotFoundError(f"missing config at {config_path}")

content = config_path.read_text(encoding="utf-8")
```
