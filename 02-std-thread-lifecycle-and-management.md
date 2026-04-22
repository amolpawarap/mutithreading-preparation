# 02 - std::thread Lifecycle and Management

## 1) How do you create a thread in C++?
- Use `std::thread` with a callable (function, lambda, functor).
- Arguments are copied/moved by default; use `std::ref` for references.
- **Direct example:**
  ```cpp
  void work(int id);
  std::thread t(work, 7);
  t.join();
  ```
- **Real-world application:** Create workers for image resizing, log parsing, or request handling.

## 2) What is joinable?
- `std::thread::joinable()` indicates a thread object represents an active thread.
- A joinable thread must be `join()`ed or `detach()`ed before destruction.
- **Direct example:**
  ```cpp
  if (t.joinable()) t.join();
  ```
- **Real-world application:** Safe shutdown in services to avoid crash during object teardown.

## 3) What happens if a joinable thread is destroyed?
- Program calls `std::terminate()`.
- This is a common interview trap.
- **Direct example:** returning from scope with still-joinable `std::thread` aborts process.
- **Real-world application:** Production crashes during shutdown if lifecycle ownership is unclear.

## 4) `join()` vs `detach()`?
- `join()`: wait for thread completion and synchronize with it.
- `detach()`: thread runs independently; no direct lifetime synchronization.
- Prefer `join()` unless detached behavior is clearly required and safe.
- **Direct example:**
  ```cpp
  std::thread t(worker);
  t.join(); // preferred for controlled lifetime
  ```
- **Real-world application:** Background telemetry flush may detach only if it owns all resources safely.

## 5) How do exceptions behave across threads?
- Exceptions do not automatically propagate to parent thread.
- Catch inside worker and communicate via `std::exception_ptr`, future/promise, or custom channel.
- **Direct example:**
  ```cpp
  std::exception_ptr ep;
  std::thread t([&]{ try { run(); } catch (...) { ep = std::current_exception(); } });
  t.join(); if (ep) std::rethrow_exception(ep);
  ```
- **Real-world application:** Worker parse failure propagated to main orchestration thread.

## 6) What is RAII for thread joining?
- Wrap thread in a guard class that joins in destructor.
- Prevents accidental `std::terminate` due to missed join.
- **Direct example:**
  ```cpp
  struct Joiner{ std::thread& t; ~Joiner(){ if(t.joinable()) t.join(); } };
  ```
- **Real-world application:** Safer thread handling in exception-heavy code paths.

## 7) What is `std::jthread` (C++20)?
- Auto-joins on destruction.
- Supports cooperative cancellation with `std::stop_token`.
- Often safer default than `std::thread`.
- **Direct example:**
  ```cpp
  std::jthread jt([](std::stop_token st){ while(!st.stop_requested()) work_once(); });
  jt.request_stop();
  ```
- **Real-world application:** Periodic monitoring workers with clean stop semantics.

## 8) How can thread IDs be used?
- `std::this_thread::get_id()` and `std::thread::get_id()`.
- Useful for logging, tracing, diagnostics.
- **Direct example:**
  ```cpp
  std::cout << "tid=" << std::this_thread::get_id() << "\n";
  ```
- **Real-world application:** Correlate deadlock/latency events with specific threads.

## 9) What are `std::this_thread::sleep_for` and `sleep_until`?
- Block current thread for a duration or until time point.
- Useful for pacing/retry/backoff but avoid in tight control loops when precise timing is required.
- **Direct example:**
  ```cpp
  std::this_thread::sleep_for(std::chrono::milliseconds(50));
  ```
- **Real-world application:** Exponential backoff for retrying flaky network calls.

## 10) Why avoid unbounded thread creation?
- High memory overhead (stack per thread).
- Scheduler overhead and context switching.
- Use thread pools/executors for many tasks.
- **Direct example:** fixed worker vector processes queue instead of one thread per request.
- **Real-world application:** Chat service handling 100k connections uses event loop + bounded worker pool.

## 11) What is thread affinity (Linux concept)?
- Binding thread to selected CPU cores.
- Can reduce migration and improve cache locality.
- Usually optimized only in performance-critical systems.
- **Direct example (POSIX):** `pthread_setaffinity_np(pthread_self(), ...)`.
- **Real-world application:** Packet processing threads pinned to dedicated cores.

## 12) What is a daemon/background worker pattern risk?
- Detached threads may outlive resources they access.
- Need explicit shutdown protocol and safe resource ownership.
- **Direct example:** detached thread reading object after owner destruction causes use-after-free.
- **Real-world application:** Background file cleaner must stop before storage subsystem shutdown.

## 13) How do you pass ownership to a thread?
- Move `std::unique_ptr` or move-only objects into thread function.
- Ensure object lifetime is clearly owned by that thread or synchronized.
- **Direct example:**
  ```cpp
  auto p = std::make_unique<Job>();
  std::thread t([](std::unique_ptr<Job> j){ j->run(); }, std::move(p));
  t.join();
  ```
- **Real-world application:** Worker takes exclusive ownership of a request context.

## 14) What is cooperative cancellation?
- Worker periodically checks cancellation token/flag and exits cleanly.
- Avoid forced termination of threads.
- **Direct example:**
  ```cpp
  std::atomic<bool> stop{false};
  while (!stop.load(std::memory_order_relaxed)) do_work();
  ```
- **Real-world application:** Graceful deployment rollout where in-flight tasks finish safely.

## 15) Interview follow-up: “How would you shut down worker threads gracefully?”
- Signal shutdown flag or stop token.
- Wake blocked workers (e.g., via condition variable).
- Join all workers before destroying shared resources.
- **Direct example:**
  ```cpp
  stop = true; cv.notify_all();
  for (auto& t : workers) if (t.joinable()) t.join();
  ```
- **Real-world application:** Service shutdown hook drains queue and exits cleanly without data loss.
