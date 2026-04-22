# 01 - Multithreading Fundamentals

## 1) What is the difference between concurrency and parallelism?
- **Concurrency**: multiple tasks make progress during overlapping time periods.
- **Parallelism**: tasks run literally at the same time on multiple cores.
- Concurrency is about **structure**; parallelism is about **execution**.
- **Direct example (C++):**
  ```cpp
  std::thread t1(download_chunk), t2(parse_chunk); // parallel if 2+ cores
  t1.join(); t2.join();
  ```
- **Real-world application:** A web server handles many connections concurrently; request parsing and compression may also run in parallel on multicore CPUs.

## 2) Process vs thread?
- A process has its own address space.
- Threads in a process share address space, heap, globals, and file descriptors.
- Each thread has its own stack, registers, and instruction pointer.
- **Direct example (POSIX C):**
  ```c
  pid_t p = fork();      // new process
  pthread_t t;           // new thread in same process
  pthread_create(&t, NULL, worker, NULL);
  ```
- **Real-world application:** Browser tabs often use multiple processes for isolation, while each process uses threads for rendering/network tasks.

## 3) Why use multithreading?
- Better responsiveness (UI + background work).
- Throughput improvements for independent tasks.
- Better utilization of multicore CPUs.
- **Direct example (C++):**
  ```cpp
  std::thread ui(render_loop);
  std::thread bg(sync_to_cloud);
  ui.join(); bg.join();
  ```
- **Real-world application:** IDE keeps editor responsive while indexing source code in background.

## 4) What kinds of problems are not good for multithreading?
- Highly sequential workloads.
- Workloads with frequent shared-state synchronization.
- Tiny tasks where thread overhead dominates.
- **Direct example (C):** small per-item work can be slower with thread setup than a single loop.
  ```c
  for (int i = 0; i < n; ++i) sum += a[i]; // often faster than spawning per-item threads
  ```
- **Real-world application:** Simple config parsing at startup is usually better single-threaded.

## 5) What is a data race in C++?
- Two or more threads access the same memory location concurrently.
- At least one access is a write.
- No synchronization/happens-before relation exists.
- Data races are **undefined behavior** in C++.
- **Direct example (bug):**
  ```cpp
  int x = 0;
  std::thread t1([&]{ ++x; });
  std::thread t2([&]{ ++x; });
  // race unless x is atomic or protected by mutex
  ```
- **Real-world application:** Shared request counters in servers must be atomic/mutex-protected to avoid corruption.

## 6) What is the C++ memory model?
- Defines visibility and ordering guarantees for multi-threaded execution.
- Uses atomic operations and synchronization primitives to establish ordering.
- Prevents relying on “works on my machine” behavior.
- **Direct example (publish data):**
  ```cpp
  data = payload;
  ready.store(true, std::memory_order_release);
  // consumer: if (ready.load(std::memory_order_acquire)) use(data);
  ```
- **Real-world application:** Lock-free handoff paths in low-latency trading or telemetry pipelines.

## 7) What is happens-before?
- A formal relation guaranteeing that effects of one operation are visible to another.
- Established by mutex lock/unlock, condition variable signaling, thread start/join, and atomics with proper ordering.
- **Direct example (mutex):**
  ```cpp
  { std::lock_guard<std::mutex> g(m); value = 42; } // unlock
  { std::lock_guard<std::mutex> g(m); read(value); } // lock sees 42
  ```
- **Real-world application:** Producer writes task payload then signals worker; worker must see the written payload.

## 8) What is false sharing?
- Multiple threads modify different variables that live on the same cache line.
- Causes heavy cache-coherency traffic and performance degradation.
- Often mitigated by padding/alignment or data layout redesign.
- **Direct example (C++):**
  ```cpp
  struct alignas(64) Counter { std::atomic<long> v{0}; };
  Counter c1, c2; // separate cache lines
  ```
- **Real-world application:** High-frequency per-core metrics counters in observability systems.

## 9) User-level thread vs kernel thread?
- Kernel threads are scheduled by OS (Linux scheduler).
- User-level threads/coroutines are scheduled in user space and may need runtime support.
- **Direct example:** `pthread_create` creates kernel-managed threads; coroutine frameworks multiplex many tasks over fewer kernel threads.
- **Real-world application:** Async runtimes (network proxies) use many lightweight tasks on a small thread pool.

## 10) Common thread lifecycle states (conceptually)?
- Created, runnable, running, blocked/waiting, terminated.
- State transitions are controlled by scheduler, sync primitives, and I/O waits.
- **Direct example (POSIX):** `pthread_create` → runnable/running, `pthread_cond_wait` → blocked, `pthread_exit`/return → terminated.
- **Real-world application:** Worker threads in message brokers repeatedly run, block on queue, then run again.

## 11) What is context switching cost?
- Switching CPU from one thread to another saves/restores state.
- Too many context switches increase overhead and reduce throughput.
- **Direct example:** thousands of runnable threads cause scheduler churn even if each does tiny work.
- **Real-world application:** API gateways limit worker count to avoid latency spikes due to scheduler pressure.

## 12) How do you decide thread count?
- For CPU-bound tasks: near number of cores (`std::thread::hardware_concurrency()` as hint).
- For I/O-bound tasks: often more threads than cores can help.
- Measure and tune; avoid hard-coded assumptions.
- **Direct example (C++):**
  ```cpp
  unsigned n = std::max(1u, std::thread::hardware_concurrency());
  ```
- **Real-world application:** File indexing may use `cores` workers; network crawling may use more due to I/O wait.

## 13) Why are locks needed despite multicore CPUs?
- Cores have private caches and out-of-order execution.
- Locks and atomics provide ordering + mutual exclusion for correctness.
- **Direct example:**
  ```cpp
  std::mutex m;
  std::lock_guard<std::mutex> g(m); // serialize shared state update
  ```
- **Real-world application:** Updating shared in-memory session maps in a backend service.

## 14) What is lock contention?
- Multiple threads frequently compete for the same mutex.
- Leads to waiting, lower parallel efficiency, and possible convoy effects.
- **Direct example:** one global `stats_mutex` around all counters becomes a bottleneck.
- **Real-world application:** E-commerce checkout throughput drops when all orders serialize on one inventory lock.

## 15) How do you design thread-safe code?
- Prefer immutable data and message passing.
- Minimize shared mutable state.
- Keep lock scopes small and consistent.
- **Direct example (producer/consumer handoff):**
  ```cpp
  std::queue<Task> q; std::mutex m; std::condition_variable cv;
  // producer pushes under lock; consumer waits on cv with predicate
  ```
- **Real-world application:** Log processing pipeline where each stage owns its data and communicates via bounded queues.
