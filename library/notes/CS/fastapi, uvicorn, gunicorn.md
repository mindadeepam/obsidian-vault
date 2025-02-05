
### Q In a fast api script, what part is called on every request and what part is like global variable if im depoying using uviciorn?

Let me break down what happens in a FastAPI application when running with uvicorn:

```python:app.py
from fastapi import FastAPI

# RUNS ONCE AT STARTUP
# These are module-level operations that run when the module is first imported
app = FastAPI()
global_variable = "I'm loaded once"
expensive_operation_result = load_something_expensive()

# RUNS ONCE AT STARTUP
@app.on_event("startup")
async def startup_event():
    # Initialize connections, load configs, etc.
    print("Application is starting up")

# RUNS ON EVERY REQUEST
@app.get("/")
async def root():
    # This entire function runs on every request
    local_variable = "I'm created every time"
    return {"message": local_variable}

# RUNS ON EVERY REQUEST
@app.get("/items")
async def get_items(query: str):
    # Variables inside route functions are recreated each time
    items = []  # This list is created fresh for each request
    return items
```

Here's what happens when:

**Runs Once (At Startup)**:
1. All module-level code (outside functions)
2. Global variables
3. FastAPI app initialization
4. Cached functions (with `@lru_cache`)
5. Code in `@app.on_event("startup")` handlers
6. Class definitions
7. Function definitions (but not their contents)

**Runs On Every Request**:
1. Everything inside route functions (`@app.get()`, `@app.post()`, etc.)
2. Local variables inside functions
3. Dependencies that aren't cached (those without `@lru_cache`)

Example with dependencies:
```python:app.py
from fastapi import FastAPI, Depends
from functools import lru_cache

app = FastAPI()

# RUNS ONCE due to @lru_cache
@lru_cache()
def get_settings():
    print("Loading settings")  # This prints only once
    return {"api_key": "secret"}

# RUNS ON EVERY REQUEST
def get_db():
    print("Getting DB connection")  # This prints on every request
    return Database()

# RUNS ON EVERY REQUEST but uses cached settings
@app.get("/")
async def root(
    settings=Depends(get_settings),  # Uses cached result
    db=Depends(get_db)  # Creates new connection each time
):
    return {"settings": settings}
```

Important Notes:
- When using `uvicorn --reload` in development, the "runs once" code will actually run again whenever you change your code
- In production (without reload), "runs once" truly means once when the server starts
- If you're running multiple worker processes (using Gunicorn or uvicorn workers), the "runs once" code runs once per worker
- Be careful with global mutable state as it can cause issues with concurrent requests

#### FuQ. suppose i put my secrets in a global variable in a diff secret.py script that i import in this script as a global var. is that then called only once?



Yes! If you store your secrets in a separate `secrets.py` file and import it, those secrets will be loaded only once when the module is first imported. Here's how it works:

```python:secrets.py
# This code runs once when the module is first imported
from your_secret_manager import get_secrets

SECRETS = get_secrets()  # This API call happens only once
API_KEY = SECRETS['api_key']
DATABASE_URL = SECRETS['db_url']
# ... other secrets ...
```

```python:app.py
from fastapi import FastAPI
from . import secrets  # Module is imported once

app = FastAPI()

@app.get("/")
async def root():
    # Using the secrets here doesn't trigger new API calls
    return {"using_key": secrets.API_KEY}

@app.get("/other")
async def other():
    # Same here - still using the same cached values
    return {"db_url": secrets.DATABASE_URL}
```

>**This works because Python's module system has an important characteristic: modules are only loaded once and then cached in `sys.modules`. Even if you import the same module multiple times in different places, Python will use the cached version after the first import.**

However, be aware of two things:

1. If you're using `uvicorn --reload` in development:
```bash
# This will reload all modules when code changes
uvicorn app:app --reload  # Secrets will be reloaded on code changes

# To reload only specific files, use --reload-dir
uvicorn app:app --reload --reload-dir=app  # Won't reload if only secrets.py changes
```

2. If you're using multiple workers:
```bash
# Each worker will load its own copy of the secrets
uvicorn app:app --workers 4  # Secrets will be loaded once per worker
```

But in both cases, each individual worker/process will still only load the secrets once, not on every request, so it's still an efficient approach!

In a serverless environment (like AWS Lambda), each "cold start" of your function will load the module again, causing the secrets to be fetched again. This is because serverless functions don't maintain state between invocations.

### Forking a working vs making a new one?? What does it mean which one does fastapi / uvicorn / gunicorn do?

When discussing “forking a working” vs. “making a new one,” the context typically applies to processes in operating systems or software development. Here’s what each term generally means, followed by how it applies to FastAPI:

Forking a Working:
	•	Refers to the act of creating a child process by duplicating an existing one (the “working” process).
	•	This is commonly done in operating systems via the fork() system call.
	•	In web servers or frameworks, “forking” can mean duplicating a process to handle new tasks without starting from scratch.
	•	Forking retains the parent process’s state, memory, and resources.

Advantages:
	•	Efficient because it reuses the state of the parent process.
	•	Useful for tasks that require sharing or reusing data initialized by the parent.

Example:
	•	Web servers like Apache or some setups of Gunicorn (a Python WSGI server) use forking to create worker processes.

Making a New One:
	•	Refers to starting a completely new process independently.
	•	This involves initializing memory, loading libraries, and setting up from scratch.
	•	Common in scenarios where independence or isolation between processes is essential.

Advantages:
	•	Provides complete isolation between processes.
	•	Useful when shared state is not desirable or could lead to conflicts.

Example:
	•	Frameworks or servers using thread-based models often create new threads instead of forking processes.

Which One Does FastAPI Use?

FastAPI itself is a framework and doesn’t handle process management directly. However, the behavior depends on the ASGI server used to run FastAPI:
	1.	Uvicorn (default choice for FastAPI):
	•	Uvicorn uses asynchronous I/O (asyncio) for handling requests.
	•	It doesn’t “fork” or “make new processes” for each request; instead, it uses an event loop to handle multiple requests concurrently within a single process.
	2.	Gunicorn with Uvicorn Workers:
	•	Gunicorn uses a process-based model, where it can “fork” worker processes to handle requests. Each worker may run an instance of Uvicorn.
	•	This setup can use either “forking a working” or “making new ones,” depending on its configuration and the system it runs on.

Summary
	•	FastAPI itself doesn’t control forking or creating new processes; it depends on the ASGI server you use.
	•	Uvicorn (FastAPI’s typical choice) avoids forking and relies on a single event loop.
	•	Gunicorn with Uvicorn workers might fork or create new ones, depending on your configuration.
### Tell me possible combinations of unicorn and gunciron combinations for production env?

When deploying FastAPI applications in a production environment, you typically pair Uvicorn (an ASGI server) with Gunicorn (a process manager). Below are possible combinations and configurations of Uvicorn and Gunicorn for production deployments, along with their typical use cases:

1. Uvicorn Standalone
	•	Command:

uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4


	•	Description:
	•	Uvicorn manages everything, including workers.
	•	You can configure the number of workers directly using the --workers flag.
	•	Suitable for lightweight deployments or simple production setups.
	•	Pros:
	•	Simple to set up and manage.
	•	Async I/O ensures efficient handling of concurrent requests.
	•	Cons:
	•	Limited process management compared to Gunicorn.
	•	Less robust under heavy loads or crash scenarios.

2. Gunicorn with Uvicorn Workers
	•	Command:

gunicorn -w 4 -k uvicorn.workers.UvicornWorker app.main:app


	•	Description:
	•	Gunicorn acts as the process manager and spawns multiple workers.
	•	Uvicorn serves as the worker handling ASGI requests.
	•	This combination provides better process supervision and is more robust for production.
	•	Pros:
	•	Gunicorn offers advanced process management (e.g., restarting workers, logging).
	•	Scales well for multi-core CPUs with the -w (workers) flag.
	•	Cons:
	•	Slightly more complex setup than Uvicorn standalone.
	•	Overhead from running Gunicorn and Uvicorn together.

3. Gunicorn with Multiple Process Types
	•	Command:

gunicorn -w 4 -k uvicorn.workers.UvicornH11Worker app.main:app


	•	Description:
	•	Uses a specialized worker class (UvicornH11Worker) to handle HTTP/1.1 requests more efficiently.
	•	Similar to the default UvicornWorker but optimized for HTTP/1.1-heavy traffic.
	•	Pros:
	•	Great for environments where HTTP/1.1 is the dominant protocol.
	•	Fine-tuned for FastAPI’s asynchronous architecture.

4. Gunicorn with Sync Workers
	•	Command:

gunicorn -w 4 -k sync app.main:app


	•	Description:
	•	Gunicorn uses synchronous workers instead of Uvicorn workers.
	•	Not recommended for FastAPI because it negates the benefits of async I/O.
	•	Pros:
	•	May work better in legacy or low-concurrency environments.
	•	Cons:
	•	Poor utilization of FastAPI’s asynchronous capabilities.
	•	Less efficient under high-concurrency loads.

5. Gunicorn with Gevent Workers
	•	Command:

gunicorn -w 4 -k gevent app.main:app


	•	Description:
	•	Uses Gevent workers for handling requests.
	•	Gevent is a cooperative multitasking library that works well for I/O-bound applications.
	•	Pros:
	•	Scales well for I/O-bound tasks like database queries or API calls.
	•	More efficient than sync workers but less performant than UvicornWorker for async.
	•	Cons:
	•	Less common setup; potential compatibility issues.
	•	Adds a dependency on Gevent.

6. Gunicorn with Thread Workers
	•	Command:

gunicorn -w 4 --threads 4 app.main:app


	•	Description:
	•	Spawns a specified number of threads per worker.
	•	Each thread can handle a request, improving concurrency in CPU-bound scenarios.
	•	Pros:
	•	Simple and effective for some workloads.
	•	Allows for multi-threaded request handling.
	•	Cons:
	•	Threads share memory, which can lead to contention.
	•	Not as efficient as Uvicorn’s asyncio-based handling.

Choosing the Right Setup
	•	Small-scale applications: Use Uvicorn Standalone (uvicorn --workers).
	•	Production-scale deployments: Use Gunicorn with Uvicorn Workers (gunicorn -k uvicorn.workers.UvicornWorker).
	•	High CPU-bound workloads: Use Gunicorn with Thread Workers (--threads).
	•	I/O-bound tasks: Use Gunicorn with Gevent Workers (-k gevent).

For most FastAPI applications in production, Gunicorn with Uvicorn Workers is the preferred setup due to its balance of robustness, scalability, and performance.