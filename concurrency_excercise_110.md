
## 🔟 Graceful Shutdown

Start long-running tasks.

From main thread:

* Sleep 2 seconds
* Call `executor.shutdown(wait=False, cancel_futures=True)`

Observe:

* Which tasks complete
* Which get cancelled

Goal:
Understand lifecycle management.
