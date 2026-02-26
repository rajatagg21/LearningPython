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

## Code

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
