## Python `asyncio` — Expert Notes (Concise)

### Core Idea

* `asyncio` = concurrency framework using **event loop + coroutines**
* Enables **single-threaded async I/O concurrency**
* Best for: network calls, file I/O, APIs, DB calls

---

### Key Components

#### 1. Event Loop

* Central execution engine
* Schedules and runs async tasks
* One loop per thread (usually main thread)

```python
asyncio.get_event_loop()
asyncio.run(main())
```

---

#### 2. Coroutine

* Defined using `async def`
* Suspends execution at `await`

```python
async def foo():
    await bar()
```

---

#### 3. Task

* Wrapper around coroutine for concurrent execution
* Scheduled on event loop

```python
task = asyncio.create_task(coro())
```

---

#### 4. Future

* Low-level placeholder for result
* Task is a subclass of Future

---

### `await`

* Pauses coroutine until result is ready
* Gives control back to event loop
* Only inside `async def`

---

### Running Coroutines

```python
asyncio.run(main())
```

* Creates event loop
* Runs until completion
* Closes loop automatically

---

### Concurrency Patterns

#### 1. Sequential (blocking)

* One by one execution

#### 2. Concurrent (async)

```python
await asyncio.gather(coro1(), coro2())
```

* Runs tasks concurrently
* Waits for all to finish

---

### Task Management

```python
t1 = asyncio.create_task(coro1())
t2 = asyncio.create_task(coro2())
await t1
await t2
```

* Start immediately
* Controlled waiting

---

### Synchronization Primitives

* `Lock` → mutual exclusion
* `Event` → signaling
* `Semaphore` → limit concurrency
* `Queue` → producer-consumer pattern

```python
async with asyncio.Lock():
```

---

### Common Functions

* `asyncio.sleep()` → non-blocking delay
* `asyncio.gather()` → run multiple coroutines
* `asyncio.wait()` → lower-level task control
* `asyncio.shield()` → prevent cancellation

---

### Cancellation

* `task.cancel()` stops coroutine
* Must handle `asyncio.CancelledError`

---

### Exception Handling

* Exceptions propagate from tasks
* `gather(..., return_exceptions=True)` prevents crash

---

### Performance Notes

* No parallel CPU execution (GIL still exists)
* Ideal for I/O-bound workloads only
* CPU-bound → use multiprocessing

---

### Common Mistakes

* Blocking calls inside async code (`time.sleep`, sync I/O)
* Not awaiting coroutines
* Creating too many tasks without control
* Mixing threads unnecessarily

---

### Mental Model

* Event loop = scheduler
* Coroutines = paused functions
* Await = “yield control”
* Tasks = scheduled coroutines running concurrently

-----------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------

## `asyncio` — Interview + Production ML Engineer Code Snippets

---

# 1. Basic Event Loop Execution (Modern Standard)

**Use case:** running async inference / API pipeline entrypoint

```python
import asyncio

async def main():
    print("Start pipeline")
    await asyncio.sleep(1)
    print("End pipeline")

if __name__ == "__main__":
    asyncio.run(main())
```

---

# 2. Concurrent API Calls (VERY COMMON IN ML SYSTEMS)

**Use case:** calling multiple model services / feature stores

```python
import asyncio
import aiohttp

async def fetch(session, url):
    async with session.get(url) as resp:
        return await resp.text()

async def main(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, u) for u in urls]
        results = await asyncio.gather(*tasks)
        return results
```

---

# 3. Parallel Model Inference (Async Wrapper)

**Use case:** batch scoring / multiple models / ensemble APIs

```python
import asyncio

async def model_a(x):
    await asyncio.sleep(0.2)
    return x * 2

async def model_b(x):
    await asyncio.sleep(0.3)
    return x + 10

async def main(x):
    a_task = asyncio.create_task(model_a(x))
    b_task = asyncio.create_task(model_b(x))

    a, b = await asyncio.gather(a_task, b_task)
    return a, b
```

---

# 4. Rate Limiting (Production API Protection)

**Use case:** prevent hitting downstream ML / external APIs

```python
import asyncio

sem = asyncio.Semaphore(5)

async def limited_call(i):
    async with sem:
        await asyncio.sleep(1)
        return i

async def main():
    tasks = [limited_call(i) for i in range(20)]
    return await asyncio.gather(*tasks)
```

---

# 5. Producer–Consumer Pipeline (ML Inference Queue)

**Use case:** streaming inference (Kafka-like behavior)

```python
import asyncio

queue = asyncio.Queue()

async def producer():
    for i in range(10):
        await queue.put(i)
    await queue.put(None)  # sentinel

async def consumer():
    while True:
        item = await queue.get()
        if item is None:
            break
        await asyncio.sleep(0.5)  # simulate inference
        print("processed:", item)

async def main():
    await asyncio.gather(producer(), consumer())
```

---

# 6. Timeout Control (VERY IMPORTANT IN PROD)

```python
import asyncio

async def slow_task():
    await asyncio.sleep(5)
    return "done"

async def main():
    try:
        result = await asyncio.wait_for(slow_task(), timeout=2)
        print(result)
    except asyncio.TimeoutError:
        print("Timed out")
```

---

# 7. Batch Processing (Inference Optimization Pattern)

```python
import asyncio

async def predict(batch):
    await asyncio.sleep(0.3)
    return [x * 2 for x in batch]

async def main(data):
    batch_size = 4
    tasks = []

    for i in range(0, len(data), batch_size):
        batch = data[i:i+batch_size]
        tasks.append(asyncio.create_task(predict(batch)))

    results = await asyncio.gather(*tasks)
    return results
```

---

# 8. Background Task (Logging / Metrics / Monitoring)

```python
import asyncio

async def log_metrics():
    while True:
        await asyncio.sleep(5)
        print("logging metrics...")

async def main():
    asyncio.create_task(log_metrics())
    await asyncio.sleep(20)
```

---

# 9. Safe Cancellation Handling (Production Stability)

```python
import asyncio

async def task():
    try:
        while True:
            await asyncio.sleep(1)
    except asyncio.CancelledError:
        print("cleanup before exit")
        raise

async def main():
    t = asyncio.create_task(task())
    await asyncio.sleep(3)
    t.cancel()
    await t
```

---

# 10. Mixed CPU + Async Pattern (ML REALITY)

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor

def cpu_heavy(x):
    return sum(i * i for i in range(x))

async def main():
    loop = asyncio.get_running_loop()

    with ThreadPoolExecutor() as pool:
        result = await loop.run_in_executor(pool, cpu_heavy, 10_0000)

    print(result)
```

---

# Interview Focus Summary (What matters most)

* `gather()` vs `create_task()`
* `Semaphore` for rate limiting APIs
* `Queue` for pipeline design
* `wait_for()` for production safety
* `run_in_executor()` for CPU-bound ML steps
* Cancellation handling (very important in services)
* Avoid blocking calls inside event loop

---

If you want, I can next give:

* 🔥 asyncio + FastAPI production architecture (ML inference server)
* 🔥 real LLM serving pipeline design using asyncio
* 🔥 interview questions + answers on asyncio (SDE2/ML Engineer level)

--------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------

You’re hitting a **very common asyncio interview trap**.

### 🔴 What’s happening

* `t.cancel()` → sends cancellation request
* Inside `task()` → `CancelledError` is caught → you **re-raise it**
* Then:

```python
await t
```

→ propagates `CancelledError` to `main()`
→ since you **don’t handle it**, program crashes

---

## ✅ Fix (Correct Production Pattern)

### Option 1 — Handle cancellation in `main` (recommended)

```python
import asyncio

async def task():
    try:
        while True:
            await asyncio.sleep(1)
    except asyncio.CancelledError:
        print("cleanup before exit")
        raise  # important

async def main():
    t = asyncio.create_task(task())
    await asyncio.sleep(3)
    t.cancel()

    try:
        await t
    except asyncio.CancelledError:
        print("task cancelled safely")

asyncio.run(main())
```

---

## ✅ Option 2 — Suppress cancellation (less preferred)

```python
try:
    await t
except asyncio.CancelledError:
    pass
```

---

## ⚠️ Critical Interview Insight

### Why `raise` is important inside task:

* Without `raise`, task looks **successfully completed**
* This breaks:

  * monitoring
  * retries
  * debugging

---

## 🧠 Mental Model

* `cancel()` ≠ immediate stop
* It injects:

```python
CancelledError
```

at next `await`

---

## 🚫 Common Mistake (DON’T DO THIS)

```python
except asyncio.CancelledError:
    print("cleanup")
    # NO raise ❌
```

→ Silent failure → dangerous in production ML pipelines

---

## 🔥 ML Production Relevance

This pattern is used in:

* model inference workers
* streaming pipelines
* async consumers (Kafka-like)
* graceful shutdown of services

---
