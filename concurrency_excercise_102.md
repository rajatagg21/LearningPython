## 2️⃣ Exception Propagation

Create a function:

```python
def fragile_task(x):
```

* Raises exception when `x == 5`
* Returns `x*x` otherwise

Requirements:

* Submit 10 tasks
* Catch exceptions per-future
* Ensure executor does not silently swallow errors

Test:
Do you understand that exceptions surface only when calling `future.result()`?
---------------------------------------------------
## Solution Code-1 (Using executor.submit)
```python
import random
from concurrent.futures import ThreadPoolExecutor, as_completed

def fragile_task(x):
    """Raises ValueError if x is 5, otherwise returns x squared."""
    if x == 5:
        raise ValueError(f"Constraint Violation: Number {x} is not allowed")
    return x * x

def run_demonstration():
    results = []
    # Using a list to track tasks 1-10
    tasks = [random.randint(3, 7) for _ in range(10)]
    
    print(f"Submitting tasks: {tasks}\n" + "-"*30)

    with ThreadPoolExecutor(max_workers=5) as executor:
        # Map futures to their original input so we can identify them in logs
        future_to_num = {executor.submit(fragile_task, num): num for num in tasks}
        
        for future in as_completed(future_to_num):
            num = future_to_num[future]
            try:
                data = future.result()
                results.append(data)
                print(f"Success: {num}^2 = {data}")
            except Exception as e:
                # This ensures the executor does not silently swallow the error
                print(f"Error processing task ({num}): {e}")

    print("-"*30)
    print(f"Successfully completed {len(results)}/10 tasks.")
    print(f"Final results list: {results}")

if __name__ == "__main__":
    run_demonstration()

```

---------------------------------------------------
## Solution Code-2 (Using executor.map)

```python

import random
from concurrent.futures import ThreadPoolExecutor, as_completed

def fragile_task(x):
    """Raises ValueError if x is 5, otherwise returns x squared."""
    if x == 5:
        raise ValueError(f"Constraint Violation: Number {x} is not allowed")
    return x * x

def run_demonstration():
    results = []
    # Using a list to track tasks 1-10
    tasks = [random.randint(3, 7) for _ in range(10)]
    
    print(f"Submitting tasks: {tasks}\n" + "-"*30)

    with ThreadPoolExecutor(max_workers=5) as executor:
        iterator = executor.map(fragile_task, tasks)

        while True:
            try:
                result = next(iterator)
                results.append(result)
                print(f"Success: {result}")
            except StopIteration:
                break
            except Exception as e:
                print(f"Task failed")

    print(f"Successfully completed {len(results)}/10 tasks.")
    print(f"Final results list: {results}")

if __name__ == "__main__":
    run_demonstration()
```

## Explanation
Using `executor.map()` is a cleaner, more functional approach, but it behaves differently than `as_completed()`. While `as_completed()` yields results as soon as they are ready (out of order), `executor.map()` yields results in the **exact same order** the inputs were provided.

### The `executor.map()` Approach

With `map`, you don't manually manage `Future` objects. Instead, you iterate over a generator. However, the exception still "surfaces" only when the iteration reaches the failing item.
---

### Key Comparison: `as_completed` vs `map`

| Feature | `as_completed(futures)` | `executor.map(func, items)` |
| --- | --- | --- |
| **Order** | Yields as they finish (fastest first). | Yields in the order of the input list. |
| **Error Handling** | Wrap `future.result()` in a `try/except`. | Wrap the `next()` call or the `for` loop iteration. |
| **Flexibility** | High (can associate data with futures). | Low (strict 1-to-1 mapping). |
| **Best Use Case** | When tasks vary wildly in duration. | When you need the output to match the input order. |

### Why "Silently Swallowing" Happens

If you use `executor.submit()` but **never** loop through the futures to call `.result()`, the thread will die quietly. The program will finish, and you’ll be left wondering why your database wasn't updated or your files weren't saved. Always ensure you have a "reaper" loop (like the ones we've written) to collect those results or errors.
