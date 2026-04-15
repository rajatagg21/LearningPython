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
