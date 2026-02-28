# 🔹 Level 2 — Concurrency Reasoning

## 4️⃣ Deadlock Scenario (Must Understand)

Recreate the classic self-deadlock:

* max_workers=1
* Task submits another task
* Calls `.result()`

Then fix it in two different ways.

Goal:
Prove you understand starvation.

---------------------------------------------------
## Solution Code

```python
from concurrent.futures import ThreadPoolExecutor
import time

def wait_on_b():
    delay = 2
    print(f"wait_on_b(): start sleeping for {delay} seconds...")
    time.sleep(delay)
    print(f"wait_on_b(): slept for {delay} seconds!")
    print(b.result(timeout=2))  # b will never complete because it is waiting on a.
    print(f"wait_on_b(): obtained result from b")
    return 5

def wait_on_a():
    delay = 3
    print(f"wait_on_a(): start sleeping for {delay} seconds...")
    time.sleep(delay)
    print(f"wait_on_a(): slept for {delay} seconds!")
    print(a.result(timeout=2))  # a will never complete because it is waiting on b.
    print(f"wait_on_a(): obtained result from a")
    return 6

# This wont throw exception ------------------------
# executor = ThreadPoolExecutor(max_workers=2)
# a = executor.submit(wait_on_b)
# b = executor.submit(wait_on_a)


# This will throw exception as result() is invoked ------------------------
with ThreadPoolExecutor(max_workers=2) as executor:
    a = executor.submit(wait_on_b)
    b = executor.submit(wait_on_a)

    for future in [a, b]:
        print(future.result())
```
--------------------------------------------------------------
## Explanation
This is a classic case of a **Circular Dependency Deadlock** (often called the "Deadly Embrace").

Even though you have **2 workers** and only **2 tasks**, the deadlock doesn't happen because of a lack of threads (like the previous example); it happens because the logic of the tasks creates an infinite waiting loop.

---

### The Anatomy of the Deadlock

Here is the play-by-play of why your code freezes:

1. **Worker 1** picks up `a` (`wait_on_b`). It starts sleeping for 2 seconds.
2. **Worker 2** picks up `b` (`wait_on_a`). It starts sleeping for 3 seconds.
3. **At T+2 seconds:** `wait_on_b` wakes up and calls `b.result()`. This tells Worker 1: *"Stop here and wait until Task B finishes."*
4. **At T+3 seconds:** `wait_on_a` wakes up and calls `a.result()`. This tells Worker 2: *"Stop here and wait until Task A finishes."*

### Why it never resolves:

* **Task A** cannot finish until it gets the result of **Task B**.
* **Task B** cannot finish until it gets the result of **Task A**.
* Both workers are now "blocked" (waiting). They aren't doing any work; they are just sitting there holding their respective threads open, waiting for a signal that will never come.

---

### Key Takeaway: The "Golden Rule" of Futures

**Never make a Future depend on the result of another Future that is also waiting on it.**

If you need data from both tasks, the coordination should happen in the **Main Thread**, not inside the worker threads themselves.

#### The Correct Pattern:

```python
executor = ThreadPoolExecutor(max_workers=2)

# Submit them independently
future_a = executor.submit(do_math_part_1)
future_b = executor.submit(do_math_part_2)

# Main thread coordinates the results
result_a = future_a.result()
result_b = future_b.result()

final_calculation = result_a + result_b

```

### Quick Test

If you changed `max_workers` to **100**, would this code still deadlock?

* **Yes.** Because the number of threads isn't the problem; the logic of `A` waiting for `B` while `B` waits for `A` is a logical loop that no amount of hardware can solve.

* It actually **is** raising an exception! You just aren't seeing it because of how `ThreadPoolExecutor` handles background crashes.

In your previous examples, we discussed how exceptions only "surface" when you call `.result()` in the **main thread**. In this specific code, the exceptions are happening **inside** the worker threads, and since no one in the main thread is watching them, they fail silently.

---

### The "Silent Crash" Timeline

Here is exactly what happens step-by-step:

1. **T+4s:** `wait_on_b` wakes up and calls `b.result(timeout=2)`.
2. **T+5s:** `wait_on_a` wakes up and calls `a.result(timeout=2)`.
3. **T+6s:** `wait_on_b` hits its 2-second timeout. A `concurrent.futures.TimeoutError` is raised **inside Worker 1**. Because there is no `try/except` inside `wait_on_b`, the function crashes and the thread exits.
4. **T+7s:** `wait_on_a` hits its 2-second timeout. A `TimeoutError` is raised **inside Worker 2**. It also crashes and exits.

### Why your console looks "stuck"

Even though the worker threads have crashed and are technically "done," your **Main Thread** is still alive. Because you didn't tell the Main Thread to wait for `a` or `b`, it reaches the end of your script.

However, because you used `executor = ThreadPoolExecutor()`, Python automatically triggers an **implicit shutdown** at the end of the script. This shutdown waits for all threads to finish. Since the threads are already dead (due to the exceptions), the program should eventually exit, but it won't print the errors unless you ask for them.

---

### How to see the Exceptions

To actually see the `TimeoutError`, you must inspect the futures from the **Main Thread**:

```python
# Add this to the bottom of your script
time.sleep(10) # Wait long enough for the timeouts to happen
print("Checking for errors...")

try:
    a.result()
except Exception as e:
    print(f"Captured error from a: {type(e).__name__}")

try:
    b.result()
except Exception as e:
    print(f"Captured error from b: {type(e).__name__}")

```
