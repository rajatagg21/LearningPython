## 3️⃣ Timeout Handling

Submit a task that sleeps 5 seconds.

Call:

```python
future.result(timeout=1)
```

Handle:

* `concurrent.futures.TimeoutError`

Then:

* Cancel the future
* Verify if cancellation succeeded

Goal:
Understand cancellation semantics.

----------------------------------
## Solution Code
```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import time
import random

def do_something(task_id):
    delay = random.randint(0, 3)
    print(f"[task_id {task_id}] Started sleeping {delay}s...")
    time.sleep(delay)
    print(f"[task_id {task_id}] Finished (This still prints even after timeout)")
    return f"task_id: {task_id}, delay: {delay}"

results = []
tasks = [i for i in range(1, 11)]
with ThreadPoolExecutor(max_workers=5) as executor:
    future_to_task_id = {executor.submit(do_something, task_id): task_id for task_id in tasks}

    for future, task_id in future_to_task_id.items():
        try:
            result = future.result(timeout = 1)
            print(f"task_id: {future_to_task_id[future]} collected")
            results.append(result)
        except TimeoutError as e:
            print(f"task_id: {future_to_task_id[future]} timed out after 1 second.")

            # attempt to cancel
            was_cancelled = future.cancel()

            if was_cancelled:
                print(f"Status: task_id {task_id} was successfully cancelled.")
            else:
                print(f"Status: task_id {task_id} could not be cancelled as it was already running")
        
        except Exception as e:
            print(f"Unexpected Error:{e}")
```

## Explanation

Your solution demonstrates a good understanding of the `timeout` parameter, but it misses a critical nuance regarding **cancellation semantics** in Python's `ThreadPoolExecutor`.

### Evaluation of Your Code

* **The Timeout Logic:** You correctly use `future.result(timeout=1)`. This will indeed raise a `TimeoutError` if the task takes longer than one second.
* **The "Silent" Success:** In your code, if a timeout occurs, you catch the exception, but the thread **keeps running in the background**.
* **The Cancellation Gap:** The prompt specifically asked to **cancel the future** and **verify if it succeeded**. In your current code, there is no call to `future.cancel()`.

---

### The Reality of Cancellation

Here is the "gotcha" with `ThreadPoolExecutor`: **You cannot cancel a task that has already started running.** * `future.cancel()` only works if the task is still sitting in the queue (waiting for an available worker thread).

* Once `do_something` begins `time.sleep()`, Python has no built-in way to "kill" that thread forcefully.

### Key Takeaways for Your Goal

1. **`future.result(timeout=1)`**: This only stops the **Main Thread** from waiting. It does **not** stop the **Worker Thread** from working.
2. **`future.cancel()`**: Returns `True` if the task was removed from the queue. Returns `False` if the task is already running or has already finished.
3. **Zombie Threads**: If you have 100 timed-out tasks that you "cancelled" (but they were already running), those 100 threads will continue to consume CPU/Memory until they naturally finish.
