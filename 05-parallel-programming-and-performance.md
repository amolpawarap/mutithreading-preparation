# 05 - Parallel Programming and Performance

## 1) Amdahl’s Law?
- Maximum speedup is limited by serial fraction of program.
- Even infinite cores cannot speed up serial parts.

## 2) Gustafson’s Law?
- For scaled workloads, parallel speedup can grow with problem size.
- Often more realistic for large data processing.

## 3) Strong scaling vs weak scaling?
- Strong scaling: fixed workload, more processors.
- Weak scaling: workload grows with processor count.

## 4) What is granularity in parallel tasks?
- Task size relative to scheduling/synchronization overhead.
- Too fine-grained tasks can lose performance.

## 5) Why can “more threads” reduce performance?
- Contention, synchronization overhead, context switching, cache misses.

## 6) What is NUMA and why does it matter?
- Non-Uniform Memory Access: memory latency depends on socket/node locality.
- Poor memory placement can hurt parallel performance.

## 7) Cache locality and data-oriented design?
- Keep frequently accessed data close and contiguous.
- Avoid pointer-heavy/random access when possible.

## 8) What is false sharing and how to diagnose it?
- Independent data on same cache line causes invalidation traffic.
- Use profiler/perf counters and padding/alignment.

## 9) How do you benchmark concurrent code?
- Warm up first, measure multiple runs, report variance.
- Pin environment (CPU governor, affinity) where practical.
- Measure latency percentiles and throughput.

## 10) Throughput vs latency trade-offs?
- Batch processing may improve throughput but hurt tail latency.
- Lock-free structures may reduce contention but increase complexity.

## 11) Parallel STL in C++?
- Execution policies like `std::execution::par` and `par_unseq`.
- Requires understanding algorithm suitability and side effects.

## 12) How to avoid oversubscription?
- Do not run more active CPU-bound workers than useful cores.
- Coordinate thread pools in mixed libraries.

## 13) Why profile before optimizing?
- Intuition is often wrong in parallel systems.
- Use flame graphs, perf, VTune, or similar tools to find bottlenecks.

## 14) What is lock convoying?
- Many threads queue behind one lock; unlock wakes one thread repeatedly.
- Reduces throughput and increases tail latency.

## 15) Parallel design interview prompt: “How do you speed up file processing pipeline?”
- Split stages (read, parse, transform, write) with bounded queues.
- Balance stage throughput.
- Preserve ordering only if required.
