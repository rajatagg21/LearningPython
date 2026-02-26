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
