### Understanding List References in Python: Mutable vs. Immutable

The core reason why `[[0]*3]*4` fails while `[0]*4` works as expected is the difference between **Value Replacement** and **In-place Mutation**.

---

## 1. The Simple Case: `[0] * 4` (Safe)
When you create a list of integers using multiplication, Python creates a list where each index points to the same integer object `0`.

* **The Logic:** Integers are **immutable**. You cannot "change" the number 0.
* **The Action:** When you run `arr[0] = 7`, you are not modifying the object at that index; you are **replacing the reference**. You are telling the first slot to stop looking at `0` and start looking at `7`.

```python
# Code Sample: Immutable Elements
arr = [0] * 4
arr[0] = 7

print(arr)  # Result: [7, 0, 0, 0]
# Each slot is independent because integers can't be mutated.
```

---

## 2. The 2D Case: `[[0]*3] * 4` (The Trap)
When you multiply a list containing another list, you are copying the **memory address** of that inner list.

* **The Logic:** Lists are **mutable**. You can change their contents without changing their identity (memory address).
* **The Action:** `mat_bad[0][0] = 5` says: "Go to the list at index 0, and change its first element to 5." Since indexes 1, 2, and 3 are pointing to that **exact same list object**, they all reflect the change.



```python
# Code Sample: Mutable Shared References
mat_bad = [[0]*3] * 4
mat_bad[0][0] = 5

print(mat_bad) 
# Result: [[5, 0, 0], [5, 0, 0], [5, 0, 0], [5, 0, 0]]
# All rows changed because they are actually the SAME list.
```

---

## 3. The Correct Way: List Comprehension
To create a true 2D matrix where rows are independent, you must ensure that a **new list object** is instantiated for every row.

* **The Logic:** The code inside the brackets `[0]*3` is executed fresh for every iteration of the `for` loop.
* **The Action:** This creates 4 distinct objects in memory. Changing one has zero effect on the others.



```python
# Code Sample: Independent Objects (Notes Version)
# Creating a 4x3 matrix correctly
mat_fixed = [[0]*3 for _ in range(4)]

mat_fixed[0][0] = 5

print(mat_fixed)
# Result: [[5, 0, 0], [0, 0, 0], [0, 0, 0], [0, 0, 0]]
# Each row is its own object in memory.
```

---

### Summary for Notes
> **Rule of Thumb:** Use `*` for 1D lists of primitives (integers, strings). Always use **List Comprehension** for 2D lists or lists of objects to avoid the "Shared Reference" bug.
