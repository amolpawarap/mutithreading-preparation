# 01 - Multithreading Fundamentals

## 1) What is the difference between concurrency and parallelism?
- **Concurrency**: multiple tasks make progress during overlapping time periods.
- **Parallelism**: tasks run literally at the same time on multiple cores.
- Concurrency is about **structure**; parallelism is about **execution**.

## 2) Process vs thread?
- A process has its own address space.
- Threads in a process share address space, heap, globals, and file descriptors.
- Each thread has its own stack, registers, and instruction pointer.

## 3) Why use multithreading?
- Better responsiveness (UI + background work).
- Throughput improvements for independent tasks.
- Better utilization of multicore CPUs.

## 4) What kinds of problems are not good for multithreading?
- Highly sequential workloads.
- Workloads with frequent shared-state synchronization.
- Tiny tasks where thread overhead dominates.

## 5) What is a data race in C++?
- Two or more threads access the same memory location concurrently.
- At least one access is a write.
- No synchronization/happens-before relation exists.
- Data races are **undefined behavior** in C++.

## 6) What is the C++ memory model?
- Defines visibility and ordering guarantees for multi-threaded execution.
- Uses atomic operations and synchronization primitives to establish ordering.
- Prevents relying on “works on my machine” behavior.

## 7) What is happens-before?
- A formal relation guaranteeing that effects of one operation are visible to another.
- Established by mutex lock/unlock, condition variable signaling, thread start/join, and atomics with proper ordering.

## 8) What is false sharing?
- Multiple threads modify different variables that live on the same cache line.
- Causes heavy cache-coherency traffic and performance degradation.
- Often mitigated by padding/alignment or data layout redesign.

## 9) User-level thread vs kernel thread?
- Kernel threads are scheduled by OS (Linux scheduler).
- User-level threads/coroutines are scheduled in user space and may need runtime support.

## 10) Common thread lifecycle states (conceptually)?
- Created, runnable, running, blocked/waiting, terminated.
- State transitions are controlled by scheduler, sync primitives, and I/O waits.

## 11) What is context switching cost?
- Switching CPU from one thread to another saves/restores state.
- Too many context switches increase overhead and reduce throughput.

## 12) How do you decide thread count?
- For CPU-bound tasks: near number of cores (`std::thread::hardware_concurrency()` as hint).
- For I/O-bound tasks: often more threads than cores can help.
- Measure and tune; avoid hard-coded assumptions.

## 13) Why are locks needed despite multicore CPUs?
- Cores have private caches and out-of-order execution.
- Locks and atomics provide ordering + mutual exclusion for correctness.

## 14) What is lock contention?
- Multiple threads frequently compete for the same mutex.
- Leads to waiting, lower parallel efficiency, and possible convoy effects.

## 15) How do you design thread-safe code?
- Prefer immutable data and message passing.
- Minimize shared mutable state.
- Keep lock scopes small and consistent.
