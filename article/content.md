Article topics to cover:

* What I want to achieve with this article
  - Improve PGO visibility across the industry
  - Give you as much PGO-related materials as possible
* Describe my experience with PGO
* Describe my PGO way - why I am doing all these things in this way
  - Showcases - because developers like numbers
  - Examples - easier to start with integration
* Note about spamming PGO issues
  - No one still complained about it (at least I didn’t know about that)
  - Improving PGO visibility - unfortunately, many developers does not know about such optimization technique
  - A good “first PGO issue” list to start playing with for a potential PGO newcomer
* What is PGO
* What does it inside of your program
* Overview of the PGO state across programming languages
  - C/C++ - good
  - Rust - good
  - Go - will be good sometime (I hope)
  - Java: GraalVM (I don’t have much experience with that, sorry)
  - Zig: write a story about a new Zig compiler and how it affects PGO dreams in this language
* Overview of the PGO state across compilers
* Overview of the PGO state across build systems
  - Add a note about Bazel integration
  - Add links to the issues in other build systems like CMake, Meson, etc. (maybe I need to check it across all popular build systems and create proper issues for that)
* Overview PGO tooling across different ecosystems
  - Rust: cargo-pgo
  - Go: default.pgo integration
  - C/C++: no comments… :(
* Instrumentation PGO: problems
  - The instrumented binary is slower. How much - usually is unpredictable
  - Double compilation - it hurts a lot smaller platforms like PowerPC in different CI pipelines
  - Increased compilation resource usage during the compilation - increased memory usage by the compiler
  - Increased binary size
  - Exit-code issues - for some applications in the end the profile is not dumped (like Clangd). You need to tweak the application manually
  - Not all software can be built with PGO (Envoy) - some dependencies does not support PGO for some reason
* Sampling-based PGO: benefits
  - You can tweak PGO instrumentation overhead (usually): perf frequency, then collect profiles, etc
  - From the point above - it’s usually okay to run Sampling in production
* Sampling-based PGO: problems
  - AutoFDO tooling is buggy
  - According to the tests, it is usually less efficient than Instrumented PGO (but we need more tests)
  - AutoFDO is limited only to perf. Want another platform? Write your own sampling and conversion into the proper formats
* PGO profile file formats:
  - You need to know them if you implement your own profiler
  - They are undocumented. They are not interchangeable between compilers (usually). No backward/forward compatibility guarantees (check guarantees across different compilers).
* PGO at scale
  - How Google implements their “PGO all the things” policy - a short overview. And why we cannot do it easily for our own IT landscapes
* How PGO can be integrated into your projects - different levels (we can call it PGO maturity levels):
  - Do nothing
  - Just benchmark it and write a note about PGO results for your application 
  - Integrate building with PGO into the build scripts (and write about it in the documentation)
  - Optimize with PGO binaries in CI
* How to collect PGO profiles?
  - Tests - please no, too many differences with your actual workload
  - Benchmarks - not so bad, but be careful regarding coverage and happy paths
  - Generating sample load and using it - a good one. But you need to maintain it and track that it does not differ much from your actual workload
  - Collecting directly from production - a good one too. But you need good tooling for that + Instrumentation PGO usually is not usable for that
* Using PGO profile from one platform on another
  - usually, it is fine since the majority of your code is platform-independent
  - but beware about platform-specific code - it will not be PGO-optimized
* Profile skew - what is it and how to fix it?
  - Is it a problem in practice?
  - How can you avoid it?
* PGO and different application domains
  - Write here domains where PGO according to the tests helped with performance
  - The list is not final, of course
* Application delivery pipeline - where PGO can be integrated:
  - On the application developer side distributed by the developer binaries
  - By maintainers of your favorite distribution
  - By maintainers in your company (if you are large enough or have some specific security concerns)
  - On your own
  - Integrated into the build scripts PGO does not mean that you will get PGO-optimized binary
* PGO implementation improvements:
  - Tooling. PGO at scale is simply not possible right now in an easy way with available open-source tooling
* PGO FAQ:
  - What if I have different workloads for the application? Well, here there are multiple options:
  - Merge profiles (could be less efficient sometimes)
  - Prepare one binary per workload and deploy them separately if you are able to do it.
  - Do I need to use LTO with PGO?
  - Good question. No - they are independent optimizations. However, usually people enables LTO at first (because it’s easier to do) and then start thinking about PGO, not vice versa. But e.g. MSVC compiler does not allow you to enable PGO without LTO (I do not know why - maybe some internal implementation difficulties - how knows)
  - Could I put information from PGO profile directly into the sources?
  - Well, sort of. Like usually you able to write inline attribute here and there based on PGO profiles. But it can be hard to maintain with a little or no benefits
* Why am I doing all this stuff with PGO
  - Better performance overall -> easier to resolve NFR for the systems
  - Popularization using runtime information during the compilation for better optimizations
  - As one more step into the ML-based optimization techniques for the AOT compilation model.
* Write a note about software distributors like Percona (PostgreSQL) and Altinity (ClickHouse)
* Write about my day-to-day experience with PGO: checking corresponding Reddits, HackerNews, GitHub Trendings, dbdb.io, AwesomeX repos, etc., googling about PGO
