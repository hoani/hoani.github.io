---
title: "Concurrent Python"
excerpt: "Multithreading and Multiprocessing in Python"
permalink: /guides/software/python/concurrent
toc: true
categories:
  - guide
  - python
---

# Multithreading

Because of the GIL (global interpretter lock), python byte code cannot run in parallel. This means that multithreading doesn't improve performance for CPU-bound tasks; however, it is useful for taking care of multiple IO requests where normally our programs spend a lot of time waiting.

Basic multithreading is similar to threads in c++:

```python
from threading import Thread

def printer(a, b):
    print(f"{a} {b}")

t1 = Thread(target=printer, args=("Hello", "World"))
t2 = Thread(target=printer, args=("Good", "night"))

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

## Data Sharing

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

# Multiprocessing

Multiprocessing spawns new processes, each has their own GIL, so we can run CPU-bound tasks in parallel.

```py
from multiprocessing import Process

def printer(a, b):
    print(f"{a} {b}")

if __name__ == '__main__':
    p1 = Process(target=printer, args=("Hello", "World"))
    p2 = Process(target=printer, args=("Good", "night"))

    p1.start()
    p2.start()

    p1.join()
    p2.join()
```

Pools are also available. There are a few ways of using these, such as `pool.map` where a list is passed. Callbacks can also be applied to get results from execution.

```py
from multiprocessing import Pool

def formatter(a, b):
    return(f"{a} {b}")

if __name__ == '__main__':
    with Pool(processes=10) as pool:
        for i in range(100):
            pool.apply_async(formatter, ("Hello", i), 
                callback=print)
        
        pool.close()
        pool.join()
```

### Queues

```py
from multiprocessing import Process, Queue

def printer(q):
    while (v := q.get()) is not None:
        a, b = v
        print(f"{a} {b}")

if __name__ == '__main__':
    q = Queue()
    p = Process(target=printer, args=(q,))
   
    p.start()
    q.put(("Hello", "World"))
    q.put(("Good", "Night"))
    q.put(None)
    p.join()
```

### Pipes

```py
from multiprocessing import Process, Pipe

def piper(conn):
    rx = conn.recv()
    print(f"process in {rx}")
    conn.send(["Good", "Night"])
    conn.close()

if __name__ == '__main__':
    p_conn, c_conn = Pipe()
    p = Process(target=piper, args=(c_conn,))
   
    p.start()
    p_conn.send(["Hello", "World"])
    rx = p_conn.recv()
    print(f"process out {rx}")
    p_conn.close()
    p.join()
```


