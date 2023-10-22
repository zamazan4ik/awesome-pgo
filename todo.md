# TODO

Here I collect random thoughts and ideas about further PGO investigation.

* Add more information about caveats of each method: PGO, AutoFDO, Bolt, Propeller, more advanced techniques
* Add more info about LTO and PGO state for packages in different Linux distros
* Add more links to the existing projects how PGO is integrated into them
* Get more information about PGO states in other browsers like Safari, Brave ([GitHub comment](https://github.com/brave/brave-browser/issues/20560#issuecomment-1658782341)), Yandex.Browser and others
* Write more about PGO and library development.
* Try to use PGO on some modules like Nginx VTS (https://github.com/vozlt/nginx-module-vts)
* There is another PGO - PostGreSQL Operator from CrunchyDate ([GitHub](https://github.com/CrunchyData/postgres-operator)). It makes a bit harder to find information about Profile-Guided Optimization :)
* PGO helps with optimizing binary size since we can inline less for actually cold paths of our programs (and it can help with performance as well since our program will be smaller and more friendly for CPU I-cache)
* Add an issue to LLVM upstream about writing a difference between PGO implementations (Frontend vs IR vs Context-Sensitive), pros and cons of each, which to use by default. Probably add a link to the blog post of a person who described all these stuff.
* Contact with more proprietary vendors regarding PGO support in their compilers and PGO optimization their compilers itself before distribution to the customers. Possible compilers list: https://en.wikipedia.org/wiki/List_of_compilers
* Write to Mike Pall (LuaJIT author) about "How to benchmark LuaJIT performance itself - without benchmarking the load"
* Points of the software supply chain, where PGO can be applied (we can influence any of them with different methods and priorities):
  - Software itself (add PGO to the build scripts)
  - Maintainers (work with different distributions about enabling PGO for the packages)
  - In-house builds (if we build binary somewhere in a closed perimeter)
* From time to time I hear from different people something like "Algorithms always beat compilers". That's could be true but that's not a question actually because you can (and should) always use compiler optimizations WITH algorithms. Just let compiler (in this case - with PGO) do their job. And free time spend on that humans do best (yet) - write algorithmic optimizations for your favourite software.
* Add a chapter about PGO tips:
  - PGO profiles are not written to a disk due to signal handlers. Overwrite them or customize. I highly recommend to tune software to write a profile to a disk with a signal (call `__llvm_dump_profile()` or [similar functions](https://github.com/llvm/llvm-project/blob/main/compiler-rt/lib/profile/InstrProfiling.h) if you use Clang)
  - Partial profile sets are useful too since they usually covers a lot of hot paths (LLD and ClickHouse as en example).
  - Merging multiple profiles - works well.
  - Running on a test suite - generally is not a good idea. Real-life workload usually would differ and a profile from the tests would not be so useful. However, if you have "real-life" tests - it's fine to use them as a profile workload.
  - PGO works well with libraries. However, will be a question - how to collect and distribute a profile for them? Especially if we are talking about general-purpose builds like in OS distros. Also, it's not easy to collect a profile from instrumented but non-instrumented binary (e.g. instrumented .so library which is used from Python software). Needs to be clarified but writing an instrumented wrapper right now is a recommended option.
  - PGO profiles does not depend on time! However, if your code has time-dependent paths, PGO profiles could differ due to "time stuff" like time issues on a build machine, different speed of different build machines, etc - be careful with that
  - PGO works fine without LTO (in normal compilers. MSVC does not belong to this family - you cannot use PGO without LTO there). So if LTO is too expensive to your build resources (especially if we are talking about FatLTO that consumes A LOT of RAM) - just use PGO without LTO, that's completely fine
  - It's hard to estimate how slow would be your application in the Instrumentation mode without actual testing. It hugely depends on the hot paths of your application, number of branches, etc. It's possible to predict based on some metrics of the application. But much easier just to test it :) Be careful, some application could be too slow in Instrumentation mode (like ClickHouse, Clangd or LLD)
  - When recompile software, would be useful to recompile 3rd parties with PGO as well. Here could be a challenge since often provided by OS packages dependencies are used. And it could be challenging to recompile them as well from the scratch (especially if we are talking about C and C++). Also, not for every software is obvious to understand - which 3rd parties are rebuilt from the scratch and which are taken from another place (like OS-provided or some package manager like Conan).
  - For reproducibility purposes you can just collect profile and commit it to the repo
* Add notes about system-wide PGO:
  - Google approach with AutoFDO: run `perf` on a subset of nodes, collect profiles for all running applications and use these profiles on CI
  - Ad-hoc approach: use AutoFDO approach from Google or "just" rebuild all packages with Instrumentation PGO, run on a real-life workload, then collect the profiles and use them during the optimized build
* Check PGO state in Linux distributions for different packages:
  - Browsers (Firefox, Chromium, etc.)
  - Compilers (GCC, Clang, Rustc, Swift)
  - Other software (like `zstd` command-line tools)
* How to quickly check PGO support in the project: search over issues for "PGO", "Profile guided", "FDO". Also works grepping over the project for the same words or for PGO-related compiler flags like `fprofile-generate`/`fprofile-use`, etc.
* Extract actual numbers directly into the document for avoiding the cases like [this](https://github.com/facebook/mariana-trench/issues/137#issuecomment-1658195725).
* Describe different PGO applications scenarios: for SaaS, for open-source, for closed-source but delivered to the customers, etc.
* Check Altinity builds for ClickHouse and suggest PGO for their CH build. Their sources are available here: https://github.com/Altinity/ClickHouse
* Using built-in benchmarks can be not so good idea: benchmark code coverage issues, built-in benchmarks can be too slow in the instrumentation modes
* Think about collecting "general" PGO profiles via crowd sourcing
* Think about a PGO talk for https://packaging-con.org/ ?
* What to do with performance regressions after PGO? Rustc question: https://users.rust-lang.org/t/how-to-report-performance-regressions-with-profile-guided-optimization-pgo/98225
* Check `simdjson` with PGO: https://github.com/simdjson/simdjson/blob/master/benchmark/CMakeLists.txt
* Check new Go PGO when Go 1.21 compiler update will arrive in the distrubutions. Some CPU-heavy scenarios like log parsing should be a good candidate to test. Right now it has a lot of places to improve: https://github.com/golang/go/issues?q=is%3Aissue+is%3Aopen+PGO
* Write an article about PGO and its caveats
* Try to optimize LLVM LLC (https://llvm.org/docs/CommandGuide/llc.html)
* Add PGO information to Rsyslog: https://github.com/rsyslog/rsyslog/issues/5048#issuecomment-1694533339
* Rustc and LTO+PGO bug: https://github.com/rust-lang/rust/issues/115344
* Ask in `cargo-pgo` repo about profiling C-like dependencies (e.g. RocksDB deps) in Rust projects: https://github.com/Kobzol/cargo-pgo/issues/38
* Not everywhere PGO is available (e.g. WASM): https://github.com/rust-lang/rust/issues/81684
* Committing PGO profiles into a project - transparency problem if you do not describe how this profile is collected
* "Awesome-X" repositories are good places to start PGOify some domains
* Envoy build fails with FDO: https://pastebin.com/8HtsEC26
* Probably a good list to optimize command-line tooling: https://gist.github.com/sts10/daadbc2f403bdffad1b6d33aff016c0a
* Write to Expensify about optimizing SQLite performance further with PGO: https://use.expensify.com/blog/scaling-sqlite-to-4m-qps-on-a-single-server
* Check https://trofi.github.io/posts/243-gcc-profiler-internals.html article about GCC profiler internals
