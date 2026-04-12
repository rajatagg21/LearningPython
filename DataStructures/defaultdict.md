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



When you need something more complex than a basic `int`, `list`, or `set`, you have two main options: **Lambda functions** for simple nested structures, or **Custom Functions** for logic-heavy initialization.

---

## 1. Using `lambda` for Nested Structures
A `lambda` is an anonymous, one-line function. It’s perfect for creating a "dictionary of dictionaries" or a "dictionary of lists."

### Example: A 2D Dictionary
If you are building a coordinate system or a nested map:
```python
from collections import defaultdict

# Factory: a function that returns a new dictionary
matrix = defaultdict(lambda: defaultdict(int))

matrix['Row1']['Col1'] += 5
print(matrix['Row1']['Col1']) # 5
```
* **How it works:** When `matrix['Row1']` is accessed, the lambda runs and creates a `defaultdict(int)`. Then, the `['Col1']` access triggers that inner dictionary's default behavior.

---

## 2. Using Custom Functions for Complex Objects
If your default value requires logic—like a custom class or a list pre-filled with specific data—you can pass the name of a standard function.

### Example: Pre-filled Data
Suppose every new key should start as a list containing a "header" or a specific timestamp.

```python
def initial_data():
    return {"status": "pending", "attempts": 0, "logs": []}

user_tracker = defaultdict(initial_data)

user_tracker['Alice']['attempts'] += 1
print(user_tracker['Alice']) 
# {'status': 'pending', 'attempts': 1, 'logs': []}
```

---

## 3. Real-World Use Case: Graph with Metadata
If you are building a graph where every node needs to track its neighbors (a `set`) and its visit status (a `bool`), you can combine these.

```python
# Each node starts with a set of neighbors and a visited flag
graph = defaultdict(lambda: {"neighbors": set(), "visited": False})

graph['A']["neighbors"].add('B')
print(graph['A']) 
# {'neighbors': {'B'}, 'visited': False}
```

---

## Summary of Factory Options

| Complexity | Factory Syntax | Best For |
| :--- | :--- | :--- |
| **Basic** | `defaultdict(list)` | Simple grouping. |
| **Nested** | `defaultdict(lambda: defaultdict(int))` | Multi-dimensional maps/grids. |
| **Object-Based** | `defaultdict(MyClass)` | Custom objects or complex state. |
| **Logic-Based** | `defaultdict(my_custom_func)` | Data that needs calculation on creation. |



### Important Note on the "Callable"
The `defaultdict` requires a **callable** (the function name).
* **Correct:** `defaultdict(list)` — You are passing the function itself.
* **Wrong:** `defaultdict(list())` — You are passing an empty list (the *result* of the function), which will cause a `TypeError`.

Does this help you see how to scale `defaultdict` for more advanced algorithms?
