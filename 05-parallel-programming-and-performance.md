# 05 - Parallel Programming and Performance

## 1) Amdahl’s Law?
- Maximum speedup is limited by serial fraction of program.
- Even infinite cores cannot speed up serial parts.
- **Direct example:** if 20% is serial, max speedup is ~5x regardless of cores.
- **Real-world application:** Parsing or final aggregation stages cap total pipeline gains.

## 2) Gustafson’s Law?
- For scaled workloads, parallel speedup can grow with problem size.
- Often more realistic for large data processing.
- **Direct example:** increasing data size with core count keeps per-core load useful.
- **Real-world application:** Large batch analytics where each new node processes extra partitions.

## 3) Strong scaling vs weak scaling?
- Strong scaling: fixed workload, more processors.
- Weak scaling: workload grows with processor count.
- **Direct example:** 1GB fixed input (strong) vs 1GB per core (weak).
- **Real-world application:** HPC teams report both metrics to assess architecture fit.

## 4) What is granularity in parallel tasks?
- Task size relative to scheduling/synchronization overhead.
- Too fine-grained tasks can lose performance.
- **Direct example:** one task per pixel may be too fine; tile-based tasks are better.
- **Real-world application:** Video encoding chunks frames into macroblocks/tiles.

## 5) Why can “more threads” reduce performance?
- Contention, synchronization overhead, context switching, cache misses.
- **Direct example:** 64 CPU-bound threads on 8 cores often lowers throughput.
- **Real-world application:** Over-threaded microservices show higher p99 latency.

## 6) What is NUMA and why does it matter?
- Non-Uniform Memory Access: memory latency depends on socket/node locality.
- Poor memory placement can hurt parallel performance.
- **Direct example (Linux):** pin thread and allocate memory on same NUMA node.
- **Real-world application:** In-memory databases place shards near worker cores.

## 7) Cache locality and data-oriented design?
- Keep frequently accessed data close and contiguous.
- Avoid pointer-heavy/random access when possible.
- **Direct example (C++):** `std::vector<Struct>` iteration is often cache-friendlier than linked lists.
- **Real-world application:** Physics/game loops use SoA layouts for SIMD/cache efficiency.

## 8) What is false sharing and how to diagnose it?
- Independent data on same cache line causes invalidation traffic.
- Use profiler/perf counters and padding/alignment.
- **Direct example:** align per-thread counters to 64 bytes.
- **Real-world application:** High-throughput counters in API gateways.

## 9) How do you benchmark concurrent code?
- Warm up first, measure multiple runs, report variance.
- Pin environment (CPU governor, affinity) where practical.
- Measure latency percentiles and throughput.
- **Direct example:** run 30 iterations, report p50/p95/p99 + ops/s.
- **Real-world application:** Capacity planning before production launch.

## 10) Throughput vs latency trade-offs?
- Batch processing may improve throughput but hurt tail latency.
- Lock-free structures may reduce contention but increase complexity.
- **Direct example:** batch size 1 vs 100 in queue consumer.
- **Real-world application:** Payment systems prioritize tail latency over max throughput.

## 11) Parallel STL in C++?
- Execution policies like `std::execution::par` and `par_unseq`.
- Requires understanding algorithm suitability and side effects.
- **Direct example:**
  ```cpp
  std::sort(std::execution::par, v.begin(), v.end());
  ```
- **Real-world application:** Large in-memory datasets sorted/transformed for analytics.

## 12) How to avoid oversubscription?
- Do not run more active CPU-bound workers than useful cores.
- Coordinate thread pools in mixed libraries.
- **Direct example:** cap each pool size and avoid nested parallelism by default.
- **Real-world application:** ML inference service with preprocessing and model pools.

## 13) Why profile before optimizing?
- Intuition is often wrong in parallel systems.
- Use flame graphs, perf, VTune, or similar tools to find bottlenecks.
- **Direct example (Linux):** `perf record` + `perf report` before code changes.
- **Real-world application:** Teams avoid wasting time optimizing non-hot paths.

## 14) What is lock convoying?
- Many threads queue behind one lock; unlock wakes one thread repeatedly.
- Reduces throughput and increases tail latency.
- **Direct example:** global queue lock under burst load creates convoy behavior.
- **Real-world application:** Job dispatch latency spikes in overloaded worker coordinators.

## 15) Parallel design interview prompt: “How do you speed up file processing pipeline?”
- Split stages (read, parse, transform, write) with bounded queues.
- Balance stage throughput.
- Preserve ordering only if required.
- **Direct example (C++):** one thread pool per stage and backpressure via fixed-capacity queues.
- **Real-world application:** ETL/log ingestion pipelines in observability stacks.
