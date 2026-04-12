These notes synthesize the official documentation with the practical behavior of `defaultdict`.

---

# Python `collections.defaultdict`

A `defaultdict` is a subclass of the built-in `dict`. It is designed to handle missing keys gracefully by automatically initializing them using a **factory function**.

## 1. Core Mechanics: `__missing__(key)`
The "magic" of `defaultdict` happens via the `__missing__` method:
* **Trigger:** Called only by `__getitem__` (e.g., `d[key]`) when a key is not found.
* **Action:** * If `default_factory` is **None**: Raises `KeyError`.
    * If `default_factory` is **not None**: Calls the factory (no arguments), inserts the result into the dictionary for that key, and returns the value.
* **Limitation:** It is **not** called by `.get()`. Using `d.get(missing_key)` still returns `None` (or your specified default), rather than triggering the factory.

---

## 2. Constructor & Initialization
```python
defaultdict(default_factory=None, /, mapping_or_iterable, **kwargs)
```
* **`default_factory`**: A callable (function, class, or lambda) used to create default values.
* **`mapping/iterable`**: You can initialize it just like a regular dict: `defaultdict(int, {'a': 1})`.

---

## 3. Common Use Cases & Factories

| Factory | Behavior | Best Used For... |
| :--- | :--- | :--- |
| **`list`** | Returns `[]` | **Grouping:** `d[k].append(v)` |
| **`int`** | Returns `0` | **Counting:** `d[k] += 1` |
| **`set`** | Returns `set()` | **Unique Collections:** `d[k].add(v)` |
| **`lambda`**| Returns constant | **Default Labels:** `lambda: "N/A"` |

### Constant Value Factory
To return a specific default string or number other than 0:
```python
def constant_factory(value):
    return lambda: value

d = defaultdict(constant_factory('<missing>'))
# Any missing key now defaults to the string '<missing>'
```

---

## 4. `defaultdict` vs `dict.setdefault()`
The documentation notes that `defaultdict` is generally **faster and simpler** than using `setdefault()`.

| Method | Syntax | Note |
| :--- | :--- | :--- |
| **`defaultdict`** | `d[k].append(v)` | Faster; logic is defined at creation. |
| **`setdefault`** | `d.setdefault(k, []).append(v)` | Slower; creates a new list object every call. |

---

## 5. Technical Summary Table

| Attribute/Method | Description |
| :--- | :--- |
| **`default_factory`** | The instance variable storing the callable function. It can be updated at runtime. |
| **Memory** | Like `dict`, but creates an entry the moment a missing key is **accessed** via `d[key]`. |
| **Operators** | Supports `|` (merge) and `|=` (update) as of Python 3.9. |

> **Warning:** Be careful with **read-only** lookups. Simply checking a value (`print(d['missing'])`) will permanently add that key to the dictionary with a default value. Use `if key in d:` to check for existence without triggering the factory.
