# 04 - Advanced Concurrency Patterns

## 1) What is a producer-consumer pattern?
- Producers generate work items.
- Consumers process items from shared queue.
- Requires synchronization (mutex + condition variable or lock-free queue).

## 2) What is a thread pool and why use it?
- Fixed set of worker threads execute queued tasks.
- Reduces thread creation/destruction overhead.
- Improves control over concurrency and resource usage.

## 3) What are futures and promises?
- `std::promise` sets a value/exception.
- `std::future` receives it asynchronously.
- Useful for one-shot result passing across threads.

## 4) What does `std::async` do?
- Runs function asynchronously (possibly deferred depending on policy).
- Returns `std::future` for result/exception.
- Use explicit launch policy when behavior matters.

## 5) What is task-based concurrency vs thread-based concurrency?
- Task-based: define units of work; runtime decides scheduling.
- Thread-based: manually manage thread lifecycles and synchronization.

## 6) What is work stealing?
- Idle worker threads steal tasks from others’ queues.
- Improves load balancing in irregular workloads.

## 7) What are lock-free and wait-free algorithms?
- Lock-free: system-wide progress is guaranteed.
- Wait-free: every thread completes in bounded steps.
- Very hard to implement correctly.

## 8) ABA problem in lock-free programming?
- Value changes A→B→A; CAS sees A and assumes unchanged.
- Mitigate with tagged pointers, version counters, hazard pointers, epoch reclamation.

## 9) What is safe memory reclamation in lock-free structures?
- Prevent freeing nodes while other threads may still read them.
- Techniques: hazard pointers, RCU-like schemes, epochs.

## 10) What is backpressure in concurrent systems?
- Mechanism to slow producers when consumers are overloaded.
- Prevents unbounded queue growth and memory blow-ups.

## 11) What is a barrier/latch?
- Coordination point where threads wait until a count/phase is reached.
- C++20 provides `std::barrier` and `std::latch`.

## 12) What is a readers-writer lock?
- Multiple readers can hold lock concurrently.
- Writers require exclusive access.
- In C++: `std::shared_mutex`.

## 13) What is priority inversion?
- High-priority thread waits on lock held by low-priority thread.
- Can be worsened by medium-priority threads preempting low-priority one.

## 14) How do you design cancellation-safe concurrent code?
- Use cooperative cancellation checks.
- Make operations idempotent where possible.
- Ensure rollback/cleanup in all termination paths.

## 15) Interview scenario: “Design a bounded blocking queue.”
- Define thread-safe push/pop APIs.
- Use mutex + condition variables for not-empty/not-full conditions.
- Handle shutdown, wakeups, and spurious notifications correctly.
