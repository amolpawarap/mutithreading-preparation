# 03 - Synchronization and Thread Safety

## 1) What does `std::mutex` provide?
- Mutual exclusion: only one thread enters critical section at a time.
- Ensures safe access to shared mutable state.

## 2) Why prefer `std::lock_guard` / `std::scoped_lock`?
- RAII lock management prevents forgotten unlocks.
- Exception-safe and less error-prone than manual lock/unlock.

## 3) `std::unique_lock` vs `std::lock_guard`?
- `unique_lock` is flexible (defer, unlock/relock, timed locks).
- `lock_guard` is minimal and slightly lighter for simple scope locking.

## 4) What is deadlock?
- Circular wait where threads each hold resources needed by others.
- Avoid via lock ordering, `std::scoped_lock`, reduced lock scope.

## 5) What is livelock?
- Threads keep reacting/retrying but make no progress.
- Unlike deadlock, threads are active but ineffective.

## 6) What is starvation?
- A thread may never get access to needed resources due to unfair scheduling/locking.

## 7) How does `std::condition_variable` work?
- One/more threads wait until a condition becomes true.
- Must be used with a predicate to handle spurious wakeups.
- Typical pattern: lock, wait(predicate), process.

## 8) Why are spurious wakeups important?
- Waiting thread can wake without notification.
- Always re-check predicate in loop or predicate overload of `wait`.

## 9) `notify_one` vs `notify_all`?
- `notify_one`: wakes one waiter, efficient for single-consumer progress.
- `notify_all`: wakes all waiters, useful when many may proceed.

## 10) What are `std::atomic` variables for?
- Lock-free/synchronized operations on single variables.
- Avoid full mutex for simple counters/flags where appropriate.

## 11) When is `memory_order_relaxed` acceptable?
- When atomicity is required but cross-thread ordering/visibility for other data is not.
- Example: statistics counters.

## 12) Acquire-release semantics in simple terms?
- Release write publishes data.
- Acquire read on corresponding atomic sees published data.
- Together create a happens-before edge.

## 13) Sequential consistency (`memory_order_seq_cst`)?
- Strongest and easiest-to-reason ordering.
- Can be slower than weaker orders on some architectures.

## 14) What is double-checked locking and common pitfall?
- Checking initialization condition before and after locking.
- Incorrect implementations can break due to reordering/visibility issues unless atomics and memory ordering are correct.

## 15) How do you test thread safety?
- Stress tests with many iterations.
- Run with ThreadSanitizer (`-fsanitize=thread`) when possible.
- Include deterministic synchronization tests for key invariants.
