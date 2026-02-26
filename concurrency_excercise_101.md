# 🔹 Level 1 — Core API Mastery (Must Know)

## 1️⃣ Submit vs Map (Ordering Semantics)

Write a program that:

* Submits 10 tasks with random sleep (0–3 sec)
* Prints results in:

  * Submission order
  * Completion order

You must use:

* `executor.submit`
* `as_completed`

Goal:
Understand ordering guarantees.

-------------------------------------------------

## Solution Code

```python
import random
import time
from concurrent.futures import ThreadPoolExecutor, as_completed

def do_work(task_id):
    delay = random.randint(0, 3)
    time.sleep(delay)
    return (f"Task_id: {task_id}, delay: {delay} seconds")

start = time.perf_counter()

with ThreadPoolExecutor() as executor:
    futures = [executor.submit(do_work, task_id) for task_id in range(10)]
    print("Printing in Submission order:")
    for future in futures:
        print(future.result())
    
    print("Printing in Completion order:")
    for future in as_completed(futures):
        try:
            data = future.result()
            print(future.result())
        except Exception as e:
            print(f"generated an exception: {e}")

finish = time.perf_counter()
print(f"Total execution time: {finish - start} seconds")
```

## Explanation:
The `do_work()` method is invoked **the moment you call `executor.submit()**`.

It does not wait for the loop to finish or for you to ask for the result. As soon as a thread in the pool becomes available, it grabs the task and starts running it.

---

### The Lifecycle of a Task

To visualize the timing, think of it in these three distinct stages:

1. **Submission (`submit`):** You hand the function and its arguments to the Executor. The Executor puts this into a **Work Queue**.
2. **Execution (The "Invocation"):** If there is an idle thread, it immediately pulls the task from the queue and runs `do_work()`. This happens **asynchronously** in the background while your main Python script continues to the next line.
3. **Resolution (`result()`):** This is just you checking the "receipt" to see what the function returned or if it crashed.

### Why this matters

In your original code, you used a **List Comprehension** to submit tasks:

```python
futures = [executor.submit(do_work, i) for i in range(10)]

```

By the time Python reaches the `[` at the end of that line, **all 10 tasks have already been submitted**, and several of them are likely already running (or even finished) before you ever start the `for` loop below it.

---
### Summary

* **Invocation happens:** Immediately upon `submit()` (if a thread is free).
* **Blocking happens:** Only when you call `.result()` or use `as_completed()`, which forces the main thread to wait for the background threads to catch up.

## What about 'Head-of-line Blocking'

That is a great catch. You are exactly right: the **function** itself has finished its work in the background. The "blocking" doesn't happen to the worker thread; it happens to **your main loop** (the one trying to print the results).

When we say "Head-of-Line Blocking" in this context, we are talking about the **Main Thread's** ability to process the results.

---

### The "Slow Task" Scenario

Imagine you submit 3 tasks:

1. **Task 1:** Takes **10 seconds** to finish.
2. **Task 2:** Takes **1 second** to finish.
3. **Task 3:** Takes **1 second** to finish.

#### 1. Using `as_completed` (No Blocking)

The loop asks: *"Is **anything** done yet?"*

* **After 1 second:** Task 2 is done. The loop immediately prints Task 2.
* **After 2 seconds:** Task 3 is done. The loop immediately prints Task 3.
* **After 10 seconds:** Task 1 is finally done. The loop prints Task 1.
**Result:** Your program starts showing data almost immediately.

#### 2. Using Submission Order (`for f in futures`)

The loop asks: *"Is **Task 1** done yet?"*

* **Seconds 1 through 9:** Task 2 and Task 3 are sitting in memory, finished and ready to go. **But your loop is stuck.** It is "blocked" waiting for `futures[0].result()`.
* **After 10 seconds:** Task 1 finishes. The loop prints Task 1, then Task 2, then Task 3 instantly.
**Result:** Your program looks like it's frozen for 10 seconds, then suddenly "bursts" all the data at once.

---

### Why does this matter?

If you are writing a **GUI** or a **Web Server**, blocking the main thread like this is bad.

* **With `as_completed`:** You can update a progress bar or show results to a user as they arrive.
* **With `submission order`:** The user sees a spinning wheel for 10 seconds, even though 90% of the work was done in the first second.

### Summary Table: Where the "Wait" Happens

| Scenario | Worker Threads | Main Thread (The Loop) |
| --- | --- | --- |
| **`as_completed`** | Work as fast as they can. | Processes whatever is ready first. |
| **`Submission Order`** | Work as fast as they can. | **Blocked** until the *specific* next task in the list is ready. |

> **Note:** Even if Task 2 is "done," its result just sits in a memory buffer until you specifically call `.result()` on it.
