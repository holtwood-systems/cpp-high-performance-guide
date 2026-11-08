# C++ 高性能指南

> 📚 Portfolio map: <https://github.com/holtwood-systems/portfolio>

<p align="center">
  <a href="https://github.com/holtwood-systems/cpp-high-performance-guide/stargazers">
    <img src="https://img.shields.io/github/stars/holtwood-systems/cpp-high-performance-guide?style=flat-square&color=yellow" alt="Stars">
  </a>
  <a href="https://github.com/holtwood-systems/cpp-high-performance-guide/actions/workflows/ci.yml">
    <img src="https://img.shields.io/github/actions/workflow/status/holtwood-systems/cpp-high-performance-guide/ci.yml?branch=master&label=CI&style=flat-square" alt="CI">
  </a>
  <a href="LICENSE">
    <img src="https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square" alt="License">
  </a>
</p>

<p align="center">
  <a href="https://holtwood-systems.github.io/cpp-high-performance-guide/">文档站</a> | <a href="https://github.com/holtwood-systems/cpp-high-performance-guide/discussions">讨论</a>
</p>

<p align="center">
  C++20 系统性能工程的公开实现：每个优化结论都带可运行的示例、基准与测量协议。
</p>

> **项目状态**：**stable**。九个主题域已完成，只修正确性 bug 与文档；权威状态注册表见
> [holtwood-systems/portfolio](https://github.com/holtwood-systems/portfolio)。

## ⚡ 核心亮点（30 秒速览）

| 维度 | 内容 |
| --- | --- |
| **9 个主题域，三层结构** | 每个模块都带 `src/` 可运行示例 + `bench/` Google Benchmark 基准 + 测试，可阅读、可构建、可测量 |
| **端到端案例：CPU GEMM 四阶段** | `examples/09-matrix-multiply/`：naive → tiling → SIMD(FMA) → OpenMP，一条完整的优化迭代叙事 |
| **五条验证 preset** | debug / release / asan / tsan / ubsan 全矩阵，正确性与并发缺陷分开验证 |
| **测量方法论** | Google Benchmark + perf + `perf_event_open` PMU 硬件计数器 + FlameGraph：一切性能结论先测量后声称 |
| **与 GPU 主线同构** | CPU 侧的 cache line / 伪共享 ↔ GPU 侧的 bank conflict / 访存合并；roofline 思路两侧通用，是理解 [cuda-foundations](https://github.com/holtwood-systems/cuda-foundations) 与 [cuflash](https://github.com/holtwood-systems/cuflash) 优化叙事的地基 |

## 这个仓库解决什么问题

关注那些"很容易被谈论、但不容易被验证"的 C++ 性能主题：

- 现代 CMake 与 preset 驱动的构建流程
- 内存与缓存布局取舍
- 现代 C++ 的性能实践（含并行 STL 执行策略）
- SIMD 与向量化（SSE2/AVX2/AVX-512/NEON）
- 并发与无锁基础、线程亲和与 NUMA
- 分布式内存与 MPI（可选模块）
- 文件 I/O 性能（read/pread/mmap 与写缓冲）
- CPU 微架构（分支预测、ILP、PMU 计数器）
- 端到端优化案例（GEMM：tiling → SIMD → OpenMP）
- 基于 benchmark 与 profiling 的性能分析与测量方法论

每个主题都尽量做到 **可阅读、可构建、可测量**。

## 你可以从哪里开始

| 区域 | 内容 |
| --- | --- |
| `examples/01-cmake-modern/` | 现代 CMake 结构与反模式 |
| `examples/02-memory-cache/` | AOS vs SOA、伪共享、对齐、预取 |
| `examples/03-modern-cpp/` | constexpr、移动语义、reserve、ranges |
| `examples/04-simd-vectorization/` | 自动向量化、intrinsics、SIMD 封装 |
| `examples/05-concurrency/` | 原子操作、无锁队列、OpenMP、线程亲和、NUMA |
| `examples/06-distributed-mpi/` | MPI 点对点、集合通信、域分解（可选：`-DHPC_ENABLE_MPI=ON`） |
| `examples/07-io-performance/` | read/pread/mmap 读路径对照、写缓冲开销（Linux） |
| `examples/08-cpu-microarch/` | 分支预测、延迟链 vs ILP、perf_event_open PMU 计数器 |
| `examples/09-matrix-multiply/` | GEMM 四阶段：naive → tiling → SIMD(FMA) → OpenMP |
| `docs/` | VitePress 文档站，覆盖深度专题 / 实战指南 / API 参考 |

## 快速开始

```bash
git clone https://github.com/holtwood-systems/cpp-high-performance-guide.git
cd cpp-high-performance-guide

cmake --preset=release
cmake --build build/release
```

运行一个 benchmark：

```bash
./build/release/examples/02-memory-cache/aos_vs_soa_bench
```

## 常用验证命令

```bash
cmake --preset=debug && cmake --build build/debug && ctest --preset=debug
cmake --preset=release && cmake --build build/release && ctest --preset=release

cmake --preset=asan && cmake --build build/asan && ctest --preset=asan
cmake --preset=tsan && cmake --build build/tsan && ctest --preset=tsan
cmake --preset=ubsan && cmake --build build/ubsan && ctest --preset=ubsan
```

## 文档入口

- **文档站：** <https://holtwood-systems.github.io/cpp-high-performance-guide/>
- **深度专题：** [`docs/zh/deep-dives/`](docs/zh/deep-dives/)
- **API 参考：** [`docs/zh/reference/api.md`](docs/zh/reference/api.md)

## 面试考点映射

| 高频面试题 | 对应示例 |
| --- | --- |
| 什么是 false sharing？怎么定位与修复？ | [`examples/02-memory-cache/src/false_sharing.cpp`](examples/02-memory-cache/src/false_sharing.cpp) + 同名 bench |
| AOS vs SOA 怎么选？对向量化有什么影响？ | [`examples/02-memory-cache/src/aos_vs_soa.cpp`](examples/02-memory-cache/src/aos_vs_soa.cpp) + `aos_soa_bench.cpp` |
| C++ memory ordering：relaxed / acquire / release / seq_cst 差在哪？ | [`examples/05-concurrency/src/atomic_ordering.cpp`](examples/05-concurrency/src/atomic_ordering.cpp) |
| 无锁队列怎么写？ABA 问题怎么处理？ | [`examples/05-concurrency/src/lock_free_queue.cpp`](examples/05-concurrency/src/lock_free_queue.cpp) |
| 分支预测为什么影响性能？怎么减少 branch miss？ | [`examples/08-cpu-microarch/src/branch_prediction.cpp`](examples/08-cpu-microarch/src/branch_prediction.cpp) |
| 依赖链 vs ILP：同样指令数为什么耗时差几倍？ | [`examples/08-cpu-microarch/src/dependency_chains.cpp`](examples/08-cpu-microarch/src/dependency_chains.cpp) |
| 怎么用 PMU 计数器（ipc / cache-miss）定位瓶颈？ | [`examples/08-cpu-microarch/src/pmu_counters.cpp`](examples/08-cpu-microarch/src/pmu_counters.cpp)（`perf_event_open`） |
| read / pread / mmap 的取舍？写缓冲为什么关键？ | `examples/07-io-performance/`（读路径对照 + 写缓冲开销） |
| NUMA 与线程亲和怎么设置？ | [`examples/05-concurrency/src/numa_binding.cpp`](examples/05-concurrency/src/numa_binding.cpp) · `thread_affinity.cpp` |
| 自动向量化为什么没发生？intrinsics 怎么用？ | `examples/04-simd-vectorization/`（SSE2/AVX2/AVX-512/NEON） |
| CPU 上的 GEMM 优化路径怎么走？ | [`examples/09-matrix-multiply/`](examples/09-matrix-multiply/)：naive → tiling → SIMD(FMA) → OpenMP + `gemm_bench.cpp` 逐级量化 |

> 方法论强调：本仓所有性能结论都要求「先测量、后声称」——每级优化都有独立
> before/after 数字与正确性校验，负结果同样记录。这套习惯与 GPU 主线
> （[cuda-foundations](https://github.com/holtwood-systems/cuda-foundations) 的
> cuBLAS 对照、[cuflash](https://github.com/holtwood-systems/cuflash) 的
> benchmark 证据协议）完全一致。

## 技术栈

- **语言：** C++20
- **构建：** CMake 3.20+、Ninja
- **测试：** Google Test、RapidCheck
- **基准测试：** Google Benchmark（MPI 通信基准为手动计时，避免集合通信死锁）
- **可选依赖：** MPI（06 模块）、Intel TBB（并行 STL par 策略）、libnuma（NUMA 示例）
- **文档：** VitePress + GitHub Pages
- **性能分析：** perf、FlameGraph、Valgrind、VTune

## 贡献

贡献流程与 hooks 配置见 [`CONTRIBUTING.md`](CONTRIBUTING.md)。

---

> 本仓方法论的来源是大规模测序数据处理中的系统性能工程：先定位瓶颈，再改数据布局
> 与并行结构，最后用测量证明收益。
