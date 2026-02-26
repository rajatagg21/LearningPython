
# 🔹 Advanced (Interview-Level)

## 11️⃣ Implement Your Own Mini Thread Pool

Using:

* `queue.Queue`
* `threading.Thread`

Recreate:

* submit()
* graceful shutdown()

If you can implement this cleanly, you truly understand thread pools.

---

## 12️⃣ Nested Executors Problem

Design a scenario where:

* Outer threadpool submits tasks
* Tasks use inner threadpool

Avoid:

* Deadlock
* Oversubscription

Explain tradeoffs.

---

# 🔹 Conceptual Questions You Must Be Able To Answer

1. Why is `ThreadPoolExecutor` bad for CPU-bound tasks?
2. What happens if you don’t call `shutdown()`?
3. Does `submit()` block?
4. How are exceptions stored internally?
5. What is the default `max_workers`?
6. Why can threads share memory but processes can’t?
7. How does the GIL affect context switching?
8. What happens if a thread crashes?
9. When should you prefer `asyncio` over thread pools?
10. Can thread pools leak memory?
