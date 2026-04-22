# 03 - Synchronization and Thread Safety

## 1) What does `std::mutex` provide?
- Mutual exclusion: only one thread enters critical section at a time.
- Ensures safe access to shared mutable state.
- **Direct example:**
  ```cpp
  std::mutex m;
  { std::lock_guard<std::mutex> g(m); ++shared_counter; }
  ```
- **Real-world application:** Protect shared in-memory order book updates.

## 2) Why prefer `std::lock_guard` / `std::scoped_lock`?
- RAII lock management prevents forgotten unlocks.
- Exception-safe and less error-prone than manual lock/unlock.
- **Direct example:** `std::scoped_lock l(m1, m2);` locks both safely.
- **Real-world application:** Avoid lock leaks during exception in payment processing code.

## 3) `std::unique_lock` vs `std::lock_guard`?
- `unique_lock` is flexible (defer, unlock/relock, timed locks).
- `lock_guard` is minimal and slightly lighter for simple scope locking.
- **Direct example:** condition variables require `std::unique_lock<std::mutex>`.
- **Real-world application:** Queue consumer unlocks while processing long job.

## 4) What is deadlock?
- Circular wait where threads each hold resources needed by others.
- Avoid via lock ordering, `std::scoped_lock`, reduced lock scope.
- **Direct example:** T1 locks A then B, T2 locks B then A.
- **Real-world application:** Cross-account transfer service deadlocks when lock order differs.

## 5) What is livelock?
- Threads keep reacting/retrying but make no progress.
- Unlike deadlock, threads are active but ineffective.
- **Direct example:** two workers repeatedly yield and retry lock simultaneously.
- **Real-world application:** Congestion-control loops that back off in lockstep.

## 6) What is starvation?
- A thread may never get access to needed resources due to unfair scheduling/locking.
- **Direct example:** high-priority workers repeatedly win lock, low-priority worker waits indefinitely.
- **Real-world application:** Background compaction never runs in overloaded databases.

## 7) How does `std::condition_variable` work?
- One/more threads wait until a condition becomes true.
- Must be used with a predicate to handle spurious wakeups.
- Typical pattern: lock, wait(predicate), process.
- **Direct example:**
  ```cpp
  std::unique_lock<std::mutex> lk(m);
  cv.wait(lk, [&]{ return !q.empty() || stop; });
  ```
- **Real-world application:** Work queue that blocks consumers when empty.

## 8) Why are spurious wakeups important?
- Waiting thread can wake without notification.
- Always re-check predicate in loop or predicate overload of `wait`.
- **Direct example:**
  ```cpp
  while (!ready) cv.wait(lk);
  ```
- **Real-world application:** Prevent processing invalid/empty state after accidental wakeup.

## 9) `notify_one` vs `notify_all`?
- `notify_one`: wakes one waiter, efficient for single-consumer progress.
- `notify_all`: wakes all waiters, useful when many may proceed.
- **Direct example:** `notify_all` on shutdown so all workers can exit.
- **Real-world application:** Config reload event wakes all dependent worker threads.

## 10) What are `std::atomic` variables for?
- Lock-free/synchronized operations on single variables.
- Avoid full mutex for simple counters/flags where appropriate.
- **Direct example:**
  ```cpp
  std::atomic<uint64_t> requests{0};
  requests.fetch_add(1, std::memory_order_relaxed);
  ```
- **Real-world application:** High-rate request metrics counters.

## 11) When is `memory_order_relaxed` acceptable?
- When atomicity is required but cross-thread ordering/visibility for other data is not.
- Example: statistics counters.
- **Direct example:** per-thread hit counters merged later.
- **Real-world application:** Monitoring counters where exact order between threads is irrelevant.

## 12) Acquire-release semantics in simple terms?
- Release write publishes data.
- Acquire read on corresponding atomic sees published data.
- Together create a happens-before edge.
- **Direct example:**
  ```cpp
  payload = p;
  flag.store(true, std::memory_order_release);
  if (flag.load(std::memory_order_acquire)) consume(payload);
  ```
- **Real-world application:** Single-producer/single-consumer handoff.

## 13) Sequential consistency (`memory_order_seq_cst`)?
- Strongest and easiest-to-reason ordering.
- Can be slower than weaker orders on some architectures.
- **Direct example:** default atomic operations are seq_cst unless specified.
- **Real-world application:** Correctness-first control paths (feature flags, global state transitions).

## 14) What is double-checked locking and common pitfall?
- Checking initialization condition before and after locking.
- Incorrect implementations can break due to reordering/visibility issues unless atomics and memory ordering are correct.
- **Direct example:** prefer `std::call_once` instead of custom DCL.
- **Real-world application:** Lazy singleton initialization in high-traffic services.

## 15) How do you test thread safety?
- Stress tests with many iterations.
- Run with ThreadSanitizer (`-fsanitize=thread`) when possible.
- Include deterministic synchronization tests for key invariants.
- **Direct example:** run workload loop with 100+ threads and randomized scheduling points.
- **Real-world application:** Catch flaky races before production rollout.
