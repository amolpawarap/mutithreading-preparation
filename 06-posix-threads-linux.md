# 06 - POSIX Threads on Linux

## 1) What is POSIX threads (pthreads)?
- Standard C API for threading on Unix-like systems.
- Linux `pthread_*` APIs are the foundation under many higher-level abstractions.
- **Direct example:** `pthread_create`, `pthread_join`, `pthread_mutex_lock`.
- **Real-world application:** Legacy C services and embedded Linux software.

## 2) Basic thread creation flow in pthreads?
- `pthread_create` to start a thread with function + argument.
- `pthread_join` to wait for completion and collect result.
- **Direct example:**
  ```c
  pthread_t t;
  pthread_create(&t, NULL, worker, arg);
  pthread_join(t, NULL);
  ```
- **Real-world application:** Parallel data chunk processing in C applications.

## 3) `pthread_t` vs `std::thread`?
- `pthread_t` is C-level thread handle.
- `std::thread` is C++ abstraction built for safer RAII-style usage.
- **Direct example:** `std::thread` destructor rules vs explicit pthread lifecycle APIs.
- **Real-world application:** Mixed C/C++ codebases wrap pthreads in C++ classes.

## 4) What are pthread mutex APIs?
- `pthread_mutex_init`, `pthread_mutex_lock`, `pthread_mutex_unlock`, `pthread_mutex_destroy`.
- Equivalent role to `std::mutex`.
- **Direct example:**
  ```c
  pthread_mutex_lock(&m);
  shared++;
  pthread_mutex_unlock(&m);
  ```
- **Real-world application:** Protect shared stats/state in multithreaded daemons.

## 5) What are pthread condition variables?
- `pthread_cond_wait`, `pthread_cond_signal`, `pthread_cond_broadcast`.
- Must be paired with mutex and predicate checks.
- **Direct example:**
  ```c
  pthread_mutex_lock(&m);
  while (!ready) pthread_cond_wait(&cv, &m);
  pthread_mutex_unlock(&m);
  ```
- **Real-world application:** Blocking queues in producer-consumer systems.

## 6) What is `pthread_rwlock_t`?
- Reader-writer lock in POSIX.
- Multiple readers or one writer at a time.
- **Direct example:** `pthread_rwlock_rdlock` for reads, `pthread_rwlock_wrlock` for writes.
- **Real-world application:** Read-heavy shared caches.

## 7) What is `pthread_spinlock_t` and when is it used?
- Busy-wait lock; thread spins instead of sleeping.
- Useful only for very short critical sections and specific low-latency scenarios.
- **Direct example:**
  ```c
  pthread_spin_lock(&s);
  // tiny critical section
  pthread_spin_unlock(&s);
  ```
- **Real-world application:** Low-latency packet counters where blocking cost is too high.

## 8) How does thread cancellation work in pthreads?
- `pthread_cancel` sends cancellation request.
- Thread must reach cancellation point or explicitly test.
- Cleanup handlers are critical for resource safety.
- **Direct example:** worker blocked in `read()` can be canceled at cancellation point.
- **Real-world application:** Service shutdown while worker threads block on I/O.

## 9) What are cleanup handlers?
- `pthread_cleanup_push/pop` register cleanup actions on cancellation/thread exit paths.
- **Direct example:** unlock mutex in cleanup handler to avoid deadlock after cancel.
- **Real-world application:** Robust cleanup in legacy systems with cancellation enabled.

## 10) How do you set thread attributes?
- `pthread_attr_init` and related setters.
- Configure detached state, stack size, scheduling attributes.
- **Direct example:** `pthread_attr_setstacksize(&attr, 1<<20);`
- **Real-world application:** Reduce memory footprint in high-thread-count applications.

## 11) What is `PTHREAD_MUTEX_RECURSIVE` and caution?
- Allows same thread to lock same mutex multiple times.
- Can hide design issues; use sparingly.
- **Direct example:** nested API calls acquiring same mutex avoid self-deadlock.
- **Real-world application:** Transitional fix in refactoring legacy layered code.

## 12) What is robust mutex (`PTHREAD_MUTEX_ROBUST`)?
- Helps detect owner death while lock held.
- Requires recovery path to restore consistent protected state.
- **Direct example:** lock returns `EOWNERDEAD`; recover state then call `pthread_mutex_consistent`.
- **Real-world application:** Shared-memory process crash recovery.

## 13) How do POSIX semaphores fit in?
- Named (`sem_open`) and unnamed (`sem_init`) semaphores.
- Useful for signaling/counting resource availability.
- **Direct example:**
  ```c
  sem_t slots; sem_init(&slots, 0, N);
  sem_wait(&slots); // acquire
  sem_post(&slots); // release
  ```
- **Real-world application:** Limit concurrent DB connections or worker slots.

## 14) Linux-specific tuning examples often discussed in interviews?
- CPU affinity (`pthread_setaffinity_np`).
- Real-time scheduling considerations (`SCHED_FIFO`, `SCHED_RR`) with care.
- Priority and privilege implications.
- **Direct example:** pin real-time audio thread to dedicated core.
- **Real-world application:** Audio/video or industrial control workloads with timing constraints.

## 15) How to interoperate C++ and pthreads in legacy systems?
- Keep C-compatible thread entry functions.
- Bridge to C++ objects carefully (lifetime + exception boundaries).
- Avoid throwing exceptions through C ABI boundaries.
- **Direct example:** pass `this` pointer via `void*` and catch exceptions inside thread entry.
- **Real-world application:** Incremental modernization of old pthread-based services using modern C++ internals.
