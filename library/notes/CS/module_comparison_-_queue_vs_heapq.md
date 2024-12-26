## #queue vs #heapq: 
  
https://chatgpt.com/share/d359b7bd-eb03-429c-98f4-0e66d5faef00

**tldr**: 
- queue is thread-safe while heapq is not, ie (we must use a **lock** (lock.acquire() and lock.release()) to ensure that only one thread modifies the heap at a time.)
- binary heap is used to implement priority queue

**queue library:**
•	Designed for multithreaded programming.
•	Provides a FIFO (First In, First Out) queue (queue.Queue), a LIFO (Last In, First Out) stack (queue.LifoQueue), and a priority queue (queue.PriorityQueue).
•	Thread-safe, meaning it’s designed for safe access in multithreaded environments.
•	Operations like put(), get(), and task_done() are blocking, and you can specify timeouts or whether an operation should block until data is available.

**heapq library:**
•	Provides functions to implement a binary heap (min-heap by default) in Python.
•	Used for implementing a priority queue, but it’s not thread-safe.
•	Works directly on a Python list, treating it as a heap structure, where the smallest element is always at the root.
•	Basic operations include heapq.heappush() (to push elements) and heapq.heappop() (to pop the smallest element).
•	More efficient for non-blocking priority queues when thread safety is not a concern.


## All the ways you could #queue 

Python's standard library provides multiple options for implementing different types of queues, including FIFO, LIFO, double-ended queues, and priority queues. Let's explore the main libraries and their functionalities:

### 1. **`collections.deque`**
   - **FIFO (First-In-First-Out):** You can use a `deque` from the `collections` module to implement a simple queue.
   - **LIFO (Last-In-First-Out):** Although `deque` is not a stack by design, you can use it to implement a LIFO queue by using `.append()` to push and `.pop()` to pop elements.
   - **Both-side Insertion (Double-Ended Queue):** `deque` allows insertion and deletion from both ends using `.append()`, `.appendleft()`, `.pop()`, and `.popleft()`.
   - **Methods:**
     ```python
     from collections import deque
     d = deque()
     d.append(1)        # Insert at the right (FIFO, LIFO push)
     d.appendleft(2)    # Insert at the left
     d.pop()            # Remove from the right (FIFO/LIFO pop)
     d.popleft()        # Remove from the left
     ```

### 2. **`queue.Queue`**
   - **FIFO Queue:** The `queue.Queue` class provides a thread-safe implementation of a FIFO queue. It is primarily used in multithreaded programs.
   - **LIFO Queue:** `queue.LifoQueue` is a thread-safe LIFO queue implementation.
   - **Priority Queue:** `queue.PriorityQueue` allows for priority-based ordering of elements. Elements must be comparable because the priority is determined by sorting.
   - **Methods:**
     ```python
     import queue
     q = queue.Queue()      # FIFO
     q.put(10)
     q.get()

     lq = queue.LifoQueue()  # LIFO
     lq.put(10)
     lq.get()

     pq = queue.PriorityQueue()  # Priority Queue
     pq.put((1, 'low'))          # Lower number = higher priority
     pq.put((0, 'high'))
     pq.get()  # ('high')
     ```

### 3. **`heapq` for Priority Queues**
   - **Priority Queue (Min-Heap):** The `heapq` module provides heap operations for implementing priority queues. By default, it creates a min-heap.
   - **Max-Heap:** You can simulate a max-heap by storing negative values.
   - **Both-side Insertion (Double-ended Priority Queue):** Not directly supported. `heapq` is designed only for priority-based ordering.
   - **Methods:**
     ```python
     import heapq
     h = []
     heapq.heappush(h, (1, 'low'))
     heapq.heappush(h, (0, 'high'))
     heapq.heappop(h)  # (0, 'high')
     ```

### 4. **`multiprocessing.Queue`**
   - **FIFO Queue:** This queue is used for inter-process communication. It is similar to `queue.Queue` but allows sharing of data between processes.
   - **LIFO and Priority Queues:** Not provided by default in `multiprocessing`.
   - **Methods:**
     ```python
     from multiprocessing import Queue
     q = Queue()
     q.put(10)
     q.get()
     ```

### Comparison Summary:
- **`collections.deque`:** Flexible double-ended queue with fast O(1) append/pop operations on both ends.
- **`queue.Queue` and `queue.LifoQueue`:** Thread-safe FIFO and LIFO implementations, commonly used in multi-threading.
- **`queue.PriorityQueue`:** Thread-safe priority queue with automatic sorting.
- **`heapq`:** Efficient heap-based priority queue implementation.
- **`multiprocessing.Queue`:** FIFO queue for process-based communication.

Each of these libraries provides different features depending on your specific use case, whether it's FIFO, LIFO, priority, or inter-process communication.