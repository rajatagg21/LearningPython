
# 🔹 Level 4 — Real Backend Simulation

## 9️⃣ Simulate Web Server Handler

Implement:

```python
def handle_request(request_id):
```

* Random sleep (simulate DB)
* Sometimes throw exception

Requirements:

* Handle 50 concurrent requests
* Log failed ones
* Ensure executor shutdown is graceful

Bonus:
Add retry mechanism.
