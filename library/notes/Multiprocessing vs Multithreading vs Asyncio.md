### High level difference bw them

In [multiprocessing](https://docs.python.org/3/library/multiprocessing.html) you leverage multiple CPUs to distribute your calculations. Since each of the CPUs runs in parallel, you're effectively able to run multiple tasks simultaneously. You would want to use multiprocessing for [CPU-bound](https://en.wikipedia.org/wiki/CPU-bound) tasks. An example would be trying to calculate a sum of all elements of a huge list. If your machine has 8 cores, you can "cut" the list into 8 smaller lists and calculate the sum of each of those lists separately on separate core and then just add up those numbers. You'll get a ~8x speedup by doing that.

In [multithreading](https://docs.python.org/3/library/threading.html) you don't need multiple CPUs. Imagine a program that sends lots of HTTP requests to the web. If you used a single-threaded program, it would stop the execution (block) at each request, wait for a response, and then continue once received a response. The problem here is that your CPU isn't really doing work while waiting for some external server to do the job; it could have actually done some useful work in the meantime! The fix is to use threads - you can create many of them, each responsible for requesting some content from the web. The nice thing about threads is that, even if they run on one CPU, the CPU from time to time "freezes" the execution of one thread and jumps to executing the other one (it's called context switching and it happens constantly at non-deterministic intervals). So if your task is [I/O bound](https://en.wikipedia.org/wiki/I/O_bound) - use threading.

[asyncio](https://docs.python.org/3/library/asyncio.html) is essentially threading where **not the CPU but you, as a programmer (or actually your application), decide where and when does the context switch happen**. In Python you use an `await` keyword to suspend the execution of your coroutine (defined using `async` keyword).

Refernece: https://stackoverflow.com/a/63519065

### Concurrent futures: best library for multi-** in python?

If u want to collect results of each thread, use `executor.submit` and `oncurrent.futures.as_completed` apis.

> [!note]
> if u are storing results, make sure function used consistently returns something, also instead of using global variables, make launch_jobs function

#### code example

```python
import concurrent.futures

def square(n):
	## try except block to ensure consistency
	try:
	    return n * n
	except:
		return None

def launch_jobs():

	results = []  # local variable

	with concurrent.futures.ThreadPoolExecutor(max_workers=500) as executor:
	    # Schedule the square function to run with different arguments concurrently
	    futures = [executor.submit(square, i) for i in range(10)]
	  
	    # Retrieve results as they become available
	    for future in concurrent.futures.as_completed(futures):
	        results.append(future.result())

	return results  

```

References:

- https://chatgpt.com/share/859ed908-64e0-4b9d-8070-5816cbbc4154
- https://medium.com/@rajputgajanan50/how-to-use-threadpoolexecutor-in-python-3-6819c7896e89
- more detail: https://stackoverflow.com/a/52082992
- a good resoruce with broader and deeper detail: https://dev.to/doziestar/concurrency-in-python-made-simple-51g3

### Memory consumption

One interesting thing I learned is that in python, memory occupied by a variable is only cleared when there remain no references to it (or u reassign a new value to the variable).

```python
import concurrent.futures
import gc

def scrape(n):
	## try except block to ensure consistency
	try:
	    ...
	    return df
	except:
		...
		return None

def launch_jobs():

	results = []  # local variable
	count=0
	args = [i for i in range(100000)]
	with concurrent.futures.ThreadPoolExecutor(max_workers=500) as executor:
	    # Schedule the square function to run with different arguments concurrently
		    futures = [executor.submit(scrape , i) for i in args]
	  
	    # Retrieve results as they become available
	    for future in concurrent.futures.as_completed(futures):
	        results.append(future.result())
		    count+=1
			if count%5000==0:
				save_results(results)
				results.empty()  # or results = []
				gc.collect()

	return results 
```

Lets take the above code example👆. Here I have a scraper function that fetches large 5-10k rows tables each from a site. Hence as I increase the total number of calls to the functions, the `results` variable they append their results to will grow very large in size and might overflow the RAM. One stratergy would be to save the file in memory/upload to bucket at regular intervals and reset the `results` variable.
But even agter doing this, I see that the memory cinsumption of my program increases linearly with time.
Upon further inspection and looking online, I find that different threads have interacted with the variable, this means they might still hold references to the variable and hence the memiry consumed is never truly cleared. (I tried explicit gc.collect(), results.empty(), etc).

Hence instead of sending all jobs to `launch_jobs` and saving checkpoints internally, I send one chunk at a time and store the returned results. This way after completng a chunk, the entire process is complete and a new one proces the new chunk. hence no references would remain.
And voila, my memory doesnt increase linearly anymore.

##### Final code

```python


def scrape():
	...

def launch_jobs(start_idx, end_idx):

	results = []  # local variable
	count=0
	args = [i for i in range(100000)][start_idx:end_idx]
	with concurrent.futures.ThreadPoolExecutor(max_workers=500) as executor:
	    # Schedule the square function to run with different arguments concurrently
		    futures = [executor.submit(scrape , i) for i in args]
	  
	    # Retrieve results as they become available
	    for future in concurrent.futures.as_completed(futures):
	        results.append(future.result())
		    count+=1
			if count%5000==0:
				print(f'{start_idx} to {end_idx} done')

	return results 

for i in range(0, 100000, 5000):
	results = launch_jobs(i, i+5000)
	save_results(results)

```

References:

- https://discuss.python.org/t/memory-managment-with-threading/37644
- https://peps.python.org/pep-0703/
