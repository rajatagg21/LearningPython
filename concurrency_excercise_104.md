# 🔹 Level 2 — Concurrency Reasoning

## 4️⃣ Deadlock Scenario (Must Understand)

Recreate the classic self-deadlock:

* max_workers=1
* Task submits another task
* Calls `.result()`

Then fix it in two different ways.

Goal:
Prove you understand starvation.
