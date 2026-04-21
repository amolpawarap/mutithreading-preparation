# 06 - POSIX Threads on Linux

## 1) What is POSIX threads (pthreads)?
- Standard C API for threading on Unix-like systems.
- Linux `pthread_*` APIs are the foundation under many higher-level abstractions.

## 2) Basic thread creation flow in pthreads?
- `pthread_create` to start a thread with function + argument.
- `pthread_join` to wait for completion and collect result.

## 3) `pthread_t` vs `std::thread`?
- `pthread_t` is C-level thread handle.
- `std::thread` is C++ abstraction built for safer RAII-style usage.

## 4) What are pthread mutex APIs?
- `pthread_mutex_init`, `pthread_mutex_lock`, `pthread_mutex_unlock`, `pthread_mutex_destroy`.
- Equivalent role to `std::mutex`.

## 5) What are pthread condition variables?
- `pthread_cond_wait`, `pthread_cond_signal`, `pthread_cond_broadcast`.
- Must be paired with mutex and predicate checks.

## 6) What is `pthread_rwlock_t`?
- Reader-writer lock in POSIX.
- Multiple readers or one writer at a time.

## 7) What is `pthread_spinlock_t` and when is it used?
- Busy-wait lock; thread spins instead of sleeping.
- Useful only for very short critical sections and specific low-latency scenarios.

## 8) How does thread cancellation work in pthreads?
- `pthread_cancel` sends cancellation request.
- Thread must reach cancellation point or explicitly test.
- Cleanup handlers are critical for resource safety.

## 9) What are cleanup handlers?
- `pthread_cleanup_push/pop` register cleanup actions on cancellation/thread exit paths.

## 10) How do you set thread attributes?
- `pthread_attr_init` and related setters.
- Configure detached state, stack size, scheduling attributes.

## 11) What is `PTHREAD_MUTEX_RECURSIVE` and caution?
- Allows same thread to lock same mutex multiple times.
- Can hide design issues; use sparingly.

## 12) What is robust mutex (`PTHREAD_MUTEX_ROBUST`)?
- Helps detect owner death while lock held.
- Requires recovery path to restore consistent protected state.

## 13) How do POSIX semaphores fit in?
- Named (`sem_open`) and unnamed (`sem_init`) semaphores.
- Useful for signaling/counting resource availability.

## 14) Linux-specific tuning examples often discussed in interviews?
- CPU affinity (`pthread_setaffinity_np`).
- Real-time scheduling considerations (`SCHED_FIFO`, `SCHED_RR`) with care.
- Priority and privilege implications.

## 15) How to interoperate C++ and pthreads in legacy systems?
- Keep C-compatible thread entry functions.
- Bridge to C++ objects carefully (lifetime + exception boundaries).
- Avoid throwing exceptions through C ABI boundaries.
