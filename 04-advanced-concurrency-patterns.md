# 04 - Advanced Concurrency Patterns

## 1) What is a producer-consumer pattern?
- Producers generate work items.
- Consumers process items from shared queue.
- Requires synchronization (mutex + condition variable or lock-free queue).
- **Direct example:** `push()` signals `cv`, consumer `wait()`s on queue-not-empty predicate.
- **Real-world application:** Log ingestion pipeline (ingest -> parse -> persist).

## 2) What is a thread pool and why use it?
- Fixed set of worker threads execute queued tasks.
- Reduces thread creation/destruction overhead.
- Improves control over concurrency and resource usage.
- **Direct example:** N worker threads pop `std::function<void()>` from a protected queue.
- **Real-world application:** HTTP server processing requests with bounded concurrency.

## 3) What are futures and promises?
- `std::promise` sets a value/exception.
- `std::future` receives it asynchronously.
- Useful for one-shot result passing across threads.
- **Direct example:**
  ```cpp
  std::promise<int> p; auto f = p.get_future();
  std::thread t([&]{ p.set_value(42); });
  t.join(); int v = f.get();
  ```
- **Real-world application:** Offload expensive computation and retrieve result later.

## 4) What does `std::async` do?
- Runs function asynchronously (possibly deferred depending on policy).
- Returns `std::future` for result/exception.
- Use explicit launch policy when behavior matters.
- **Direct example:**
  ```cpp
  auto fut = std::async(std::launch::async, compute);
  auto out = fut.get();
  ```
- **Real-world application:** Parallel precomputation during request handling.

## 5) What is task-based concurrency vs thread-based concurrency?
- Task-based: define units of work; runtime decides scheduling.
- Thread-based: manually manage thread lifecycles and synchronization.
- **Direct example:** thread pool queueing lambdas (tasks) vs manually creating `std::thread` per operation.
- **Real-world application:** Batch analytics favors task model for dynamic load balancing.

## 6) What is work stealing?
- Idle worker threads steal tasks from others’ queues.
- Improves load balancing in irregular workloads.
- **Direct example:** per-worker deque with steal-from-back policy.
- **Real-world application:** Build systems and job schedulers with uneven task durations.

## 7) What are lock-free and wait-free algorithms?
- Lock-free: system-wide progress is guaranteed.
- Wait-free: every thread completes in bounded steps.
- Very hard to implement correctly.
- **Direct example:** CAS loop using `std::atomic_compare_exchange_weak`.
- **Real-world application:** Low-latency queues in telemetry and trading.

## 8) ABA problem in lock-free programming?
- Value changes A→B→A; CAS sees A and assumes unchanged.
- Mitigate with tagged pointers, version counters, hazard pointers, epoch reclamation.
- **Direct example:** store `(pointer, version)` pair instead of pointer-only CAS.
- **Real-world application:** Lock-free stack pop correctness under heavy contention.

## 9) What is safe memory reclamation in lock-free structures?
- Prevent freeing nodes while other threads may still read them.
- Techniques: hazard pointers, RCU-like schemes, epochs.
- **Direct example:** thread announces protected pointer before dereference.
- **Real-world application:** In-memory key-value engines with lock-free indexes.

## 10) What is backpressure in concurrent systems?
- Mechanism to slow producers when consumers are overloaded.
- Prevents unbounded queue growth and memory blow-ups.
- **Direct example:** bounded queue `push()` blocks or drops when full.
- **Real-world application:** Streaming platforms throttle ingest when sink lags.

## 11) What is a barrier/latch?
- Coordination point where threads wait until a count/phase is reached.
- C++20 provides `std::barrier` and `std::latch`.
- **Direct example:** worker threads finish stage A, then all continue to stage B.
- **Real-world application:** Simulation timesteps where all partitions sync between phases.

## 12) What is a readers-writer lock?
- Multiple readers can hold lock concurrently.
- Writers require exclusive access.
- In C++: `std::shared_mutex`.
- **Direct example:**
  ```cpp
  std::shared_mutex rw;
  std::shared_lock r(rw);   // read
  std::unique_lock w(rw);   // write
  ```
- **Real-world application:** Config/cache read-heavy workloads.

## 13) What is priority inversion?
- High-priority thread waits on lock held by low-priority thread.
- Can be worsened by medium-priority threads preempting low-priority one.
- **Direct example (POSIX):** priority inheritance mutex attribute mitigates inversion.
- **Real-world application:** Robotics/control loops missing deadlines due to lock ownership.

## 14) How do you design cancellation-safe concurrent code?
- Use cooperative cancellation checks.
- Make operations idempotent where possible.
- Ensure rollback/cleanup in all termination paths.
- **Direct example:** check stop token between stages and close resources via RAII.
- **Real-world application:** Graceful shutdown of ETL jobs without partial corruption.

## 15) Interview scenario: “Design a bounded blocking queue.”
- Define thread-safe push/pop APIs.
- Use mutex + condition variables for not-empty/not-full conditions.
- Handle shutdown, wakeups, and spurious notifications correctly.
- **Direct example (core operations):** `cv_not_full.wait(...)` in push, `cv_not_empty.wait(...)` in pop.
- **Real-world application:** Job scheduler queue protecting memory under burst traffic.
