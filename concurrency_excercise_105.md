## 5️⃣ Bounded Parallelism (Rate Limiting)

Problem:

You have 100 URLs.
You must:

* Allow only 5 concurrent downloads at any time
* But keep executor max_workers=20

Implement using:

* `threading.Semaphore`

Goal:
Understand external concurrency control.
