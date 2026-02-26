## 8️⃣ Shared State Race Condition

Create shared counter:

```python
counter = 0
```

Launch 100 threads incrementing it.

Observe incorrect results.

Fix using:

* `threading.Lock`

Goal:
Prove you understand thread safety.
