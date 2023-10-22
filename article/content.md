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
* Different PGO names (PGO, FDO, PDF, etc.)
  - usually, PGO is used but be ready for other abbreviations as well
* Types of PGO
  - Instrumentation-based
  - Sampling-based
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
* PGO state across OS distributions
  - Sometimes PGO is disabled on some platforms due to a lack of build resources
  - Sometimes PGO is disabled due to reproducible builds concerns. Concern about reproducing the profile
  - Could we trust the profiles from the upstream to use them instead of our own? Good question. That’s why is important to commit scripts for reproducing the profile (and still can differ due to time-based things like I had in YDB)
* PGO and proprietary software
  - ArenagraphDB just didn’t answer my request - maybe an email wasn’t enterprise-enough? :)
  - NauEngine: sent an email about PGO (not response yet)
* PGO implementation improvements:
  - Tooling. PGO at scale is simply not possible right now in an easy way with available open-source tooling
* Post-Link Optimizers (LLVM BOLT, Propeller, their states and perspectives)
* Bunch of funny stories from integrating PGO into different projects:
  - Hardcoded timeout issue in YDB and Instrumentation PGO
  - Developers does not believe in their own benchmarks (DuckDB and Broot)
  - Rainer Grimm from Rsyslog thinks that algorithm always beats peephole optimizations (it’s wrong - they work at the same time)
  - Developers are unresponsive (SQLite, MongoDB and others)
  - Bugs with LTO and PGO in Rustc compiler
  - ClickHouse in the Instrumentation mode runs the ClickBench suite for more 30+ hours on my Linux setup
  - Got an additional RAM from my friend since 32 Gib is not enough for LLVM BOLT :)
  - Needed to complete registrations in many bugzillas or even more trickier platforms - please add the possibility to login with Google, GitHub or smth like that
  - Some people integrate PGO even without benchmarks - just because they can :) Usually, it’s not a good idea (since PGO can give you nothing but you will pay for the longer compilation process)
  - Problems with hardware architecture availability in CI (GitHub Actions as an example of such CI service)
  - A comment from Stolyarov about PGO in Thalassa :)
* PGO FAQ:
  - What if I have different workloads for the application? Well, here there are multiple options:
  - Merge profiles (could be less efficient sometimes)
  - Prepare one binary per workload and deploy them separately if you are able to do it.
  - Do I need to use LTO with PGO?
  - Good question. No - they are independent optimizations. However, usually people enables LTO at first (because it’s easier to do) and then start thinking about PGO, not vice versa. But e.g. MSVC compiler does not allow you to enable PGO without LTO (I do not know why - maybe some internal implementation difficulties - how knows)
  - Could I put information from PGO profile directly into the sources?
  - Well, sort of. Like usually you able to write inline attribute here and there based on PGO profiles. But it can be hard to maintain with a little or no benefits
* Piece of advice for the developers:
  - If you are a language/compiler designer - please consider integrating PGO into your language/compiler
  - If you are a project developer - please consider providing better PGO integration into your project if you care about the performance
  - If you are a maintainer - please consider enabling PGO in your packages
  - If you have experience with PGO in production - please share your numbers/pains/experience with us!
* A bunch of possible ideas for further PGO improvements:
  - Write a PGO handbook - maybe would be a good idea
  - Many Go applications are waiting to be PGO-optimized. CNCF - I am looking at you
  - Integrate PGO profiles into your favorite IDE: you can read the code and see - where is hot and cold paths of your program
  - Integrate PGO into Godbolt program - easier to show the PGO difference in some microbenchmarks
* Similar projects:
  - Application-Specific Operating Systems (ASOS)
  - Machine-learning based compilers like Clang with Tensorflow inlining module
* PGO experts (from my experience):
  - Add Niels Bohr quote about experts and mistakes
  - Kobzol from Rust team
  - People from LLVM BOLT dev team
* Why am I doing all this stuff with PGO
  - Better performance overall -> easier to resolve NFR for the systems
  - Popularization using runtime information during the compilation for better optimizations
  - As one more step into the ML-based optimization techniques for the AOT compilation model.
* Write a note about software distributors like Percona (PostgreSQL) and Altinity (ClickHouse)
