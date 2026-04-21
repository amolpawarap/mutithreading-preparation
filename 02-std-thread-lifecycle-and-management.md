# 02 - std::thread Lifecycle and Management

## 1) How do you create a thread in C++?
- Use `std::thread` with a callable (function, lambda, functor).
- Arguments are copied/moved by default; use `std::ref` for references.

## 2) What is joinable?
- `std::thread::joinable()` indicates a thread object represents an active thread.
- A joinable thread must be `join()`ed or `detach()`ed before destruction.

## 3) What happens if a joinable thread is destroyed?
- Program calls `std::terminate()`.
- This is a common interview trap.

## 4) `join()` vs `detach()`?
- `join()`: wait for thread completion and synchronize with it.
- `detach()`: thread runs independently; no direct lifetime synchronization.
- Prefer `join()` unless detached behavior is clearly required and safe.

## 5) How do exceptions behave across threads?
- Exceptions do not automatically propagate to parent thread.
- Catch inside worker and communicate via `std::exception_ptr`, future/promise, or custom channel.

## 6) What is RAII for thread joining?
- Wrap thread in a guard class that joins in destructor.
- Prevents accidental `std::terminate` due to missed join.

## 7) What is `std::jthread` (C++20)?
- Auto-joins on destruction.
- Supports cooperative cancellation with `std::stop_token`.
- Often safer default than `std::thread`.

## 8) How can thread IDs be used?
- `std::this_thread::get_id()` and `std::thread::get_id()`.
- Useful for logging, tracing, diagnostics.

## 9) What are `std::this_thread::sleep_for` and `sleep_until`?
- Block current thread for a duration or until time point.
- Useful for pacing/retry/backoff but avoid in tight control loops when precise timing is required.

## 10) Why avoid unbounded thread creation?
- High memory overhead (stack per thread).
- Scheduler overhead and context switching.
- Use thread pools/executors for many tasks.

## 11) What is thread affinity (Linux concept)?
- Binding thread to selected CPU cores.
- Can reduce migration and improve cache locality.
- Usually optimized only in performance-critical systems.

## 12) What is a daemon/background worker pattern risk?
- Detached threads may outlive resources they access.
- Need explicit shutdown protocol and safe resource ownership.

## 13) How do you pass ownership to a thread?
- Move `std::unique_ptr` or move-only objects into thread function.
- Ensure object lifetime is clearly owned by that thread or synchronized.

## 14) What is cooperative cancellation?
- Worker periodically checks cancellation token/flag and exits cleanly.
- Avoid forced termination of threads.

## 15) Interview follow-up: “How would you shut down worker threads gracefully?”
- Signal shutdown flag or stop token.
- Wake blocked workers (e.g., via condition variable).
- Join all workers before destroying shared resources.
