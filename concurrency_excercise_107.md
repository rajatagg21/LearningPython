# 🔹 Level 3 — GIL + Performance Awareness

## 7️⃣ IO-bound vs CPU-bound Benchmark

Create:

### A) IO task

```python
time.sleep(1)
```

### B) CPU task

Prime checking loop

Run both with:

* max_workers=1
* max_workers=4

Measure total time.

Explain:
Why IO scales but CPU doesn’t (GIL).
