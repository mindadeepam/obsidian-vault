https://chatgpt.com/share/66e3714a-5588-800c-ae22-fee4f2536c5b

### `asyncio` Module Overview

- **`asyncio`** is a Python library for writing concurrent code using the `async`/`await` syntax. It’s used for managing I/O-bound operations and tasks that can be paused and resumed.
- asyncio is built around the concept of `coroutines`, which are functions that can pause and resume execution. The event loop manages these coroutines, switching between them based on their readiness (e.g., waiting for I/O, waiting for a timer).
- all this is donce in the same process, althiugh we do have utilitlies to launch new process/threads
- **asyncio allows functions to say, bhai isme mujhe time lagsakta hai baaki cheezein karni hai toh karlo (by doing await slow_func)**

## Key Concepts

1. **Event Loop**: Core of `asyncio`, managing and scheduling tasks.
```python
import asyncio
loop = asyncio.get_event_loop()

loop.run(coroutine)
# Executes a coroutine until it completes and blocks the current thread until the coroutine finishes.

task = loop.create_task(<some_coroutine>)
# Schedules a coroutine to be executed concurrently and returns a Task object. It does not block; the coroutine runs in the background.

# that task can be completed, canceled, check_status etc
task.cancel()
result = task.result()
await task
```

2. **Coroutine**: Function defined with `async def` that can be paused and resumed.
   ```python
   async def my_coroutine():
       await asyncio.sleep(1)
   ```

3. **Task**: Wraps a coroutine, allowing it to run concurrently.
	```python
	task = asyncio.create_task(my_coroutine()) 
	# here no matter from where we call this, another coroutine is just added non-blockingly to our eventloop.
	# <do stuff>
	await task    # ab actuall code runs
	```

4. **Future**: An object representing a result that hasn’t been computed yet.
   ```python
   future = asyncio.Future()
   ```

5. **`await`**: Pauses the coroutine until the awaited coroutine completes.
   ```python
   await asyncio.sleep(1)
   await some_coroutine()     # this means ab some_coroutine chalega before anything
   ```

6. **`async with`**: Manages asynchronous context managers.
   ```python
   async with aiofiles.open('file.txt', 'r') as f:
       content = await f.read()
   ```

#### Basic Usage

1. **Running Coroutines**:
   ```python
   async def main():
       await asyncio.gather(coroutine1(), coroutine2())
   
   asyncio.run(main())
   ```

2. **Waiting for Multiple Tasks**:
   ```python
   async def task1():
       await asyncio.sleep(2)
   
   async def task2():
       await asyncio.sleep(1)
   
   async def main():
       await asyncio.gather(task1(), task2())
   
   asyncio.run(main())
   ```

3. **Handling Exceptions**:
   ```python
   async def faulty_task():
       raise ValueError("An error occurred")
   
   async def main():
       try:
           await faulty_task()
       except ValueError as e:
           print(e)
   
   asyncio.run(main())
   ```

4. **Timeouts**:
   ```python
   async def timeout_example():
       try:
           await asyncio.wait_for(asyncio.sleep(2), timeout=1)
       except asyncio.TimeoutError:
           print("Task timed out")
   
   asyncio.run(timeout_example())
   ```

#### Gotchas

1. **Blocking Calls**: Avoid using synchronous code that blocks the event loop. Use asynchronous versions or run blocking code in a separate thread/process.
```
result = await loop.run_in_executor("process_or_thread", get_body_text_from_url, url)
```

2. **Thread Safety**: `asyncio` isn’t thread-safe. Use `asyncio.run` or manage event loops carefully when using threads.

3. **Nested Event Loops**: `asyncio.run` creates and manages the event loop, so avoid nesting event loops.

4. **Exception Handling**: Ensure that exceptions in tasks are handled, as unhandled exceptions can terminate the program.

5. **Resource Cleanup**: Use `async with` for resource management to ensure proper cleanup.

#### Advanced Usage

1. **Custom Event Loop**:
   ```python
   loop = asyncio.new_event_loop()
   asyncio.set_event_loop(loop)
   ```

2. **Synchronization Primitives**:
   - **Locks**: `asyncio.Lock()`
   - **Events**: `asyncio.Event()`
   - **Semaphores**: `asyncio.Semaphore()`

3. **Subprocesses**:
```python
process = await asyncio.create_subprocess_exec('ls', '-l', stdout=asyncio.subprocess.PIPE)
stdout, stderr = await process.communicate()
```

4. **Streams**:
```python
reader, writer = await asyncio.open_connection('example.com', 80)
writer.write(b'GET / HTTP/1.0\r\n\r\n')
await writer.drain()
response = await reader.read()
```




%%  not visible>>>>>>>>>>>>>>>>>>>>>>
**1. Coroutines aka async def functions():**
-  When you define a function with async def, it allows the function to use await expressions within it.
- This makes the function non-blocking, meaning it can “pause” at certain points (using await) to allow other tasks to run while waiting for I/O operations (like file reading, network requests, etc.)
- Instead of returning a value directly, an async function returns a **coroutine object**, which needs to be awaited or run using an event loop.

```python
import asyncio

async def some_func_that_runs_in_bg():
	for i in range(100):
		print(2)
		await asyncio.sleep(1)             # non-blocking code, control back to its parent

asyncio.run(some_func_that_runs_in_bg)     # blocking code, run launches an event loop that will run this coroutine     
print("outer side")
```
not vicible >>>>>>>>>>>>>>>>>>>>>>
%%

# QnA

Q. **`asyncio.run` vs `asyncio.create_task`**
	
	- Neither asyncio.create_task nor asyncio.run launch a new process or thread. They operate within the same process, leveraging the power of cooperative multitasking through the event loop.
	
	- asyncio.run does indeed launch an event loop. This event loop is responsible for managing the execution of coroutines and tasks. It switches between them based on their readiness (e.g., waiting for I/O, waiting for a timer, etc.).
	- asyncio.create_task doesn't launch an event loop itself. It assumes an event loop is already running. It creates a Task object, which is essentially a wrapper around a coroutine, and schedules it for execution within the existing event loop.
	
	In simpler terms:
	- asyncio.run sets up the stage (the event loop) and then runs the coroutine.
	- asyncio.create_task adds a new actor (the task) to the stage that's already running.
	
	Key Points:
	- No new processes or threads: asyncio achieves concurrency by switching between coroutines within the same process, not by creating new processes or threads.
	- Event loop is the conductor: The event loop manages the execution of coroutines and tasks, ensuring they run smoothly and efficiently.

Q. **Can asyncio create threads?**
	
	No, asyncio itself does not directly create threads. It focuses on achieving concurrency through cooperative multitasking within a single process using an event loop.
	
	Here's why:
	- asyncio's core principle: asyncio is built around the concept of coroutines, which are functions that can pause and resume execution. The event loop manages these coroutines, switching between them based on their readiness (e.g., waiting for I/O, waiting for a timer).
	- Threads vs. Coroutines: Threads are separate units of execution within a process, each with its own stack and memory. Coroutines, on the other hand, are functions that share the same memory space and stack.
	- Concurrency, not parallelism: asyncio provides concurrency, meaning it allows multiple tasks to appear to run simultaneously, but they are actually interleaved by the event loop. Threads, on the other hand, can achieve true parallelism, where multiple tasks run truly simultaneously on different CPU cores.
	However, you can use threads alongside asyncio if needed:
	- asyncio.to_thread: This function allows you to run a function in a separate thread from within an asyncio coroutine. This can be useful for tasks that are inherently blocking or require thread-specific operations.
	- ThreadPoolExecutor: You can use the concurrent.futures.ThreadPoolExecutor to create a thread pool and submit tasks to it. This can be helpful for managing multiple threads efficiently.

Q. how is asyncio paralelism if only one thread/process is being used?
	
	You're absolutely right to point out that in `asyncio`, only one line of code is being executed at any given time within the event loop. This is **not true parallelism** (like with threads or processes), but rather **concurrency** achieved via **cooperative multitasking**. Here's how it works and how it's implemented.
	
	### How `asyncio` Concurrency Works
	
	`asyncio` is single-threaded and uses an event loop to manage tasks. While at any given moment only one coroutine is being executed, coroutines can "cooperate" by **yielding control** when they are waiting on something (like I/O, a timer, or a network request). This allows the event loop to switch to other tasks in the meantime.
	
	### Core Mechanism: The Event Loop
	
	1. **Task Scheduling**:
	   - When you create a task using `asyncio.create_task()`, the task is scheduled and added to the event loop’s list of tasks.
	   
	2. **Task Execution**:
	   - The event loop runs one task at a time, executing it until the task hits an `await` statement (which internally invokes `yield`). When a task is waiting (e.g., for I/O), it gives up control, and the event loop switches to another task.
	
	3. **Switching Between Tasks**:
	   - The event loop keeps track of tasks that are waiting on I/O, timers, or other coroutines. It runs each task until the task either completes or hits a point where it must wait for some event (like a network response or a timer). This allows tasks to "take turns" without blocking the entire program.
	
	4. **Non-blocking I/O**:
	   - The real power of `asyncio` comes from its ability to perform non-blocking I/O. While one task is waiting for I/O (e.g., file read, HTTP request), the event loop runs other tasks. This is very efficient because I/O operations often take much longer than CPU computations, and without `asyncio`, you'd waste time waiting for those operations to complete.
	
	### Coroutine Suspension via `await`
	
	The key mechanism behind `asyncio` concurrency is that when a coroutine encounters an `await` statement, it doesn't block the entire event loop; instead, it suspends the coroutine and **yields control** back to the event loop. Here's how that happens:
	
	- When a coroutine calls `await`, the event loop looks for the next task that’s ready to continue (usually because the awaited I/O has completed).
	- The original coroutine that awaited is suspended until its awaited operation is complete.
	- Once the operation completes (e.g., an I/O result is available), the event loop resumes the coroutine at the point where it was paused.
	
	### Why It's Not Parallelism
	
	- **Single Thread**: Only one coroutine is actually running at any point in time. There is no true parallel execution unless you use threads or processes.
	- **No Preemption**: Unlike multi-threading, coroutines voluntarily give up control (via `await`) instead of being interrupted by the system. This is why it's called **cooperative multitasking**.
	  
	  In contrast, threads or processes can be preemptively switched by the OS at any time, allowing for true parallelism. But with `asyncio`, a coroutine keeps running until it hits an `await`, at which point it yields control back to the event loop.
	
	### Visualization of `asyncio` Workflow
	
	Imagine tasks as cars driving on a single-lane road:
	
	- When a task (car) reaches a red light (`await`), it stops and lets another task move forward. Once the light turns green (I/O is done), it resumes driving.
	- The event loop (traffic controller) ensures that no task is blocked forever and each task gets a chance to progress when it's ready.
	
	### Example to Illustrate Concurrency
	
	Here's a simple `asyncio` example to show how it runs tasks concurrently even though only one task runs at a time:
	
```python
	import asyncio
	
	async def coro1():
	    print("coro1: Starting")
	    await asyncio.sleep(2)  # Simulate I/O
	    print("coro1: Finishing after 2 seconds")
	
	async def coro2():
	    print("coro2: Starting")
	    await asyncio.sleep(1)  # Simulate I/O
	    print("coro2: Finishing after 1 second")
	
	async def main():
	    task1 = asyncio.create_task(coro1())
	    task2 = asyncio.create_task(coro2())
	    
	    await task1
	    await task2
	
	asyncio.run(main())
	```
	
	In this example:
	- Both `coro1` and `coro2` are scheduled concurrently.
	- Even though `await asyncio.sleep()` pauses execution of the current coroutine, the event loop switches to the other coroutine in the meantime.
	- `coro2` finishes after 1 second, while `coro1` finishes after 2 seconds. If they were run sequentially, the total time would be 3 seconds, but with `asyncio` concurrency, the total time is only 2 seconds.
	
	### How It's Implemented
	
	Under the hood:
	- **Coroutines**: Python coroutines are implemented as generators with `yield` statements. When a coroutine is paused at an `await`, it's effectively yielding control back to the event loop.
	- **Event Loop**: The event loop is a **scheduler** that manages the lifecycle of tasks, switching between them when they are ready to run or have yielded control (hit `await`).
	- **Non-blocking I/O**: `asyncio` interacts with non-blocking I/O APIs. For example, when waiting on a network socket, the task is suspended until data arrives, but the event loop keeps running other tasks during that wait.
	
	### Concurrency vs Parallelism
	
	- **Concurrency**: Multiple tasks can make progress by taking turns (like `asyncio`). This is efficient for I/O-bound tasks.
	- **Parallelism**: Multiple tasks are truly running at the same time (using multiple CPU cores), which can be done using threads or processes.
	
	If you need true parallelism (for CPU-bound tasks), you’d need to use something like **multi-threading** or **multi-processing**. For I/O-bound tasks, `asyncio`'s concurrency is typically sufficient and much more efficient than using multiple threads.
	
	### Summary:
	- **Single-threaded**: Only one task executes at a time, but `asyncio` achieves concurrency by suspending tasks during I/O waits.
	- **Efficient I/O**: Tasks yield control (via `await`) to the event loop, which manages switching between tasks, ensuring efficient use of resources.
	- **Not Parallel**: Tasks run concurrently, not in parallel. For CPU-bound parallelism, you would need threads or processes.


Q. What is diff bw theese fucntions? excercise..
	
```python
def some_long_time_taking_task():
	time.sleep(5)   #3rd line execute, takes 5 seconds, nothng else can happen in the meantime
	# or any other func taking time like slow_func()
	print("inside some_long_time_taking_task")  # 4th line executes

def main():
	print("inside main")          # 1st line executes
	some_long_time_taking_task()  # 2nd line executes, (will take 5 seconds)
	print("function called and done")  # 5th line executes

main()
```


```python

async def some_long_time_taking_task():
	await asyncio.sleep(5)        # 3rd line, (non-blocking) this also takes 5 seconds, but eventloop knows and can do other tasks (but there arent any here so it will take long only) till then.
	print("inside some_long_time_taking_task") # 4th line executes

async def main():
	print("inside main")                # 1st line executes again
	await some_long_time_taking_task()  #  2nd line executes, we are awaiting it so it will finish before next line is executed
	print("function called and done")   # 5th line executes
	

asyncio.run(main())
```

they will run the same.

one way to make it efficient is to do
```python
async def some_long_time_taking_task():
	print("non blocking code still runs")
	await asyncio.sleep(5)        # 4th line, this also takes 5 seconds, blocking code here
	print("inside some_long_time_taking_task") # 5th line executes

async def main():
	print("inside main")                # 1st line executes again
	task = asyncio.create_task(some_long_time_taking_task())  #  2nd line added to task,and all non-blocking code inside it will run concurrently
	print("function called and done")   # 3rd line executes
	await task
	

asyncio.run(main())


```


Have a look at this
```python
async def some_long_time_taking_task():
	print("non blocking code still runs")
	await asyncio.sleep(5)       
	print("inside some_long_time_taking_task") 

async def main():
	print("inside main")               
	task = asyncio.create_task(some_long_time_taking_task())   
	print("function called and done")  
	await asyncio.sleep(6)
	print("slept for 6 sec")
	await task
	print("main over")


#>>>>>>>>>>
# inside main
# function called and done
# non blocking code still runs
# inside some_long_time_taking_task
# slept for 6 sec
# main over
```

the code flows like this here:
- `print("inside main")`: prints
- `task = asyncio.create_task(some_long_time_taking_task())`: task registered nothing happens here yet.
- `print("function called and done")  `: prints
- `await asyncio.sleep(6)`: 6 sec non-blocking code, so eventloop transfers control to other function
- `print("non blocking code still runs")`: prints
- `await asyncio.sleep(5) `: 5 sec non-blocking code again, so eventloop tranfers cotrol back to others if they are ready, but here the other func is taking longer. so we just wait here till this completes
- `print("inside some_long_time_taking_task") `
- `print("slept for 6 sec")`
- `await task`: by the time we are awaiting this, it has coimpleted,  bit if `some_long_time_taking_task` took say 10 sconds, we would get `main_over` printed before inside `some_long_time_taking_task`









