# Concurrency

Process, thread, Global Interpreter Lock (GIL)
- Each instance of Python interpreter is a process.
  - Additional processes: `multiprocessing`, `concurrent.futures`
  - `subprocess` is made to run an external program
- GIL
  - Access to object reference counts and other internal interpreter state is controlled by GIL.
  - Only one Python thread can hold the GIL at any time. This means 1 thread can execute Python code regardless of number of CPU cores
  - To prevent a Python thread from holding the GIL forever
    - Python's bytecode interpreter pauses the current Python thread every 5 ms (default), releasing the GIL.
    - The thread can try to reacquire the GIL but if there are threads in the waiting queue, OS scheduler might pick one of them to proceed.
  - High level Python code doesn't have control over GIL but built-in function or an extension written in lower-level language (e.g: C) can release the GIL while runing time-consuming tasks.
  - Functions in library that make syscall release the GIL. E.g: time.sleep(), functions perform disk I/O or network I/O,...
  - CPU-intensive functions in NumPy/SciPy libraries, compressing/decompressing functions from `zlib` and `bz2` modules, also release the GIL. 
  - Extensions that integrate at the Python/C APi level can also launch Python threads that are not affected by the GIL.
    - GIL-free threads cannot change Python objects but can read from and write to memory underlying objects that support buffer protocol (`bytearray`, `array.array`, `NumPy` arrays)
  - **I/O functions release GIL** -> GIL's effect on network programming with Python threads is relatively small. Network latency > memory latency
  - Consequently, each individual thread spends a lot of time waiting, execution can be interleaved without major impact on the overall throughput. "Python threads are great at doing nothing".
  - Contention over the GIL slows down compute-intensive Python threads -> can be done faster with single-threaded code (simpler as well).
- To run CPU-intensive Python code on multiple cores, we need multiple Python processes.

> CPython implementation detail: In CPython, due to the Global Interpreter Lock, only one thread can execute Python code at once (even though certain performance-oriented libraries might overcome this limitation). If you want your application to make better use of the computational resources of multi-core machines, you are advised to use multiprocessing or concurrent.futures.ProcessPoolExecutor. However, threading is still an appropriate model if you want to run multiple I/O-bound tasks simultaneously.

from [Python `threading`](https://docs.python.org/3/library/threading.html#thread-objects)

Note:
- GIL is not part of the Python language definition.
