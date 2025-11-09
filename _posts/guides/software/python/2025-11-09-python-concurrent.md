---
title: "Concurrent Python"
excerpt: "Multithreading and Multiprocessing in Python"
permalink: /guides/software/python/concurrent
classes: wide
categories:
  - guide
  - python
---

# Multithreading

Because of the GIL (global interpretter lock), python byte code cannot run in parallel. This means that multithreading doesn't improve performance for CPU-bound tasks; however, it is useful for taking care of multiple IO requests where normally our programs spend a lot of time waiting.

Basic multithreading is similar to threads in c++:

```python
import threading

def printer(a, b):
    print(f"{a} {b}")

t1 = threading.Thread(target=printer, args=("Hello", "World"))
t2 = threading.Thread(target=printer, args=("Good", "night"))

t1.start()
t2.start()

t1.join()
t2.join()
```

In many cases, managing threads is simplified using a thread pool executor:

```py
from concurrent.futures import ThreadPoolExecutor

def printer(a, b):
    print(f"{a} {b}")

with ThreadPoolExecutor(max_workers=10) as executor:
    for i in range(100):
        executor.submit(printer, "Hello", i)
```

## Inter-thread communication

### Condition variables

```py
from concurrent.futures import ThreadPoolExecutor
import threading
import time

value = None
done = False

def producer():
    global value, done
    with cv:
        for i in range(10):
            time.sleep(0.1)
            value = i # Produce a new value
            cv.notify()
            cv.wait_for(lambda: value is None)
        done = True
        cv.notify()

def consumer():
    global value, done
    while True:
        with cv:
            cv.wait_for(lambda: value is not None or done)
            if done:
                return
            print(f"consumed {value}")
            value = None
            cv.notify()

cv = threading.Condition()
with ThreadPoolExecutor() as executor:
    executor.submit(producer)
    executor.submit(consumer)
```

### Queues

```py
from concurrent.futures import ThreadPoolExecutor
import queue
import time

def producer():
    for i in range(10):
        time.sleep(0.1)
        q.put(i)
    q.put(None) # Indicate we are done

def consumer():
    while (value := q.get()) is not None:
        print(f"consumed {value}")

q = queue.Queue()
with ThreadPoolExecutor() as executor:
    executor.submit(producer)
    executor.submit(consumer)
```