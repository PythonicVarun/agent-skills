---
name: Coding Style for Python and JS/TS
description: >
    Coding style rules for Python and JS/TS. Apply this skill
    whenever writing, editing, reviewing, or generating any code - Python scripts,
    TypeScript modules, React components, CLI tools, notebooks, or any other code
    artifact. Do not skip this skill just because the task seems small or
    self-contained; even a 5-line helper function must follow these rules.
---

# Coding Style

These rules are non-negotiable and apply to every piece of code written or edited.

---

## Python

- **Type hints on all function signatures.** Every parameter and return value must be annotated. Avoid `Any` unless there is genuinely no better alternative - if you reach for `Any`, document why.
- **`pathlib` over `os.path`.** Use `Path` objects for all filesystem operations. Avoid `os.path.join`, `os.path.exists`, etc.
- **f-strings for all string formatting.** No `%`-style or `.format()` calls.

```python
# ✅
from pathlib import Path

def read_config(path: Path) -> dict[str, str]:
    return json.loads(path.read_text())

# ❌
def read_config(path):
    return json.loads(open(os.path.join(path, "config.json")).read())
```

---

## JS / TS

- **ESM only.** Use `import`/`export`. Never `require()` or `module.exports`.
- **`const` by default.** Use `let` only when a variable genuinely needs to be reassigned. Never `var`.
- **Async/await over `.then()` chains.** All async logic must use `await` inside `async` functions.
- **Type annotations on all function signatures (TS).** Every parameter and return type must be explicitly annotated. Avoid `any` - prefer `unknown` and narrow it, or define a proper interface/type.

```ts
// ✅
import { readFile } from "fs/promises";

const fetchConfig = async (filePath: string): Promise<Config> => {
    const raw = await readFile(filePath, "utf-8");
    return JSON.parse(raw);
};

// ❌ - no types, uses .then()
const fetchConfig = (filePath) =>
    fs.readFile(filePath, "utf-8").then((raw) => JSON.parse(raw));
```

---

## General (applies to all languages)

### Explicit errors - never swallow them

- Python: no bare `except: pass` or `except Exception: pass`. Always catch a specific exception type and handle it or re-raise.
- JS/TS: no `.catch(() => {})` or unhandled promise rejections. Errors must either propagate or be explicitly logged and handled.

```python
# ✅
try:
    data = json.loads(raw)
except json.JSONDecodeError as exc:
    raise ValueError(f"Invalid config JSON: {exc}") from exc

# ❌
try:
    data = json.loads(raw)
except:
    pass
```

### Small, single-purpose functions

Each function does **exactly one thing**. If a function is fetching, parsing, _and_ saving - split it into three functions. A useful heuristic: if you struggle to name a function without using "and", it should be split.

```python
# ✅
def fetch_data(url: str) -> bytes: ...
def parse_response(raw: bytes) -> list[Record]: ...
def save_records(records: list[Record], dest: Path) -> None: ...

# ❌
def fetch_and_parse_and_save(url: str, dest: str) -> None: ...
```

### No repeated logic - extract shared code

If the same logic appears in more than one place, extract it into a named function. This includes repeated string patterns, validation checks, data transformations, and URL/path construction.

```ts
// ✅
const buildEndpoint = (resource: string) => `${BASE_URL}/api/v1/${resource}`;

const getUsers = async () => fetch(buildEndpoint("users"));
const getPosts = async () => fetch(buildEndpoint("posts"));

// ❌
const getUsers = async () => fetch(`${BASE_URL}/api/v1/users`);
const getPosts = async () => fetch(`${BASE_URL}/api/v1/posts`);
```
