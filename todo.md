# TODO

Here I collect random thoughts and ideas about further PGO investigation.

* Add more info about LTO and PGO state for packages in different Linux distros
* Get more information about PGO states in other browsers like Safari, Brave ([GitHub comment](https://github.com/brave/brave-browser/issues/20560#issuecomment-1658782341)), Yandex.Browser and others
* Write more about PGO and library development.
* Try to use PGO on some modules like Nginx VTS (https://github.com/vozlt/nginx-module-vts)
* Add an issue to LLVM upstream about writing a difference between PGO implementations (Frontend vs IR vs Context-Sensitive), pros and cons of each, which to use by default. Probably add a link to the blog post of a person who described all these stuff.
* Contact with more proprietary vendors regarding PGO support in their compilers and PGO optimization their compilers itself before distribution to the customers. Possible compilers list: https://en.wikipedia.org/wiki/List_of_compilers
* Points of the software supply chain, where PGO can be applied (we can influence any of them with different methods and priorities):
  - Software itself (add PGO to the build scripts)
  - Maintainers (work with different distributions about enabling PGO for the packages)
  - In-house builds (if we build binary somewhere in a closed perimeter)
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
* How to quickly check PGO support in the project: search over issues for "PGO", "Profile guided", "FDO". Also works grepping over the project for the same words or for PGO-related compiler flags like `-fprofile-generate`/`-fprofile-use`, etc.
* Extract actual numbers directly into the document for avoiding the cases like [this](https://github.com/facebook/mariana-trench/issues/137#issuecomment-1658195725).
* Describe different PGO applications scenarios: for SaaS, for open-source, for closed-source but delivered to the customers, etc.
* Check Altinity builds for ClickHouse and suggest PGO for their CH build. Their sources are available here: https://github.com/Altinity/ClickHouse
* Using built-in benchmarks can be not so good idea: benchmark code coverage issues, built-in benchmarks can be too slow in the instrumentation modes
* What to do with performance regressions after PGO? Rustc question: https://users.rust-lang.org/t/how-to-report-performance-regressions-with-profile-guided-optimization-pgo/98225
* Check `simdjson` with PGO: https://github.com/simdjson/simdjson/blob/master/benchmark/CMakeLists.txt
* Check new Go PGO when Go 1.21 compiler update will arrive in the distrubutions. Some CPU-heavy scenarios like log parsing should be a good candidate to test. Right now it has a lot of places to improve: https://github.com/golang/go/issues?q=is%3Aissue+is%3Aopen+PGO
* Add PGO information to Rsyslog: https://github.com/rsyslog/rsyslog/issues/5048#issuecomment-1694533339
* Not everywhere PGO is available (e.g. WASM): https://github.com/rust-lang/rust/issues/81684
* Committing PGO profiles into a project - transparency problem if you do not describe how this profile is collected
* "Awesome-X" repositories are good places to start PGOify some domains
* Envoy build fails with FDO: https://pastebin.com/8HtsEC26
* Probably a good list to optimize command-line tooling: https://gist.github.com/sts10/daadbc2f403bdffad1b6d33aff016c0a
* Check https://trofi.github.io/posts/243-gcc-profiler-internals.html article about GCC profiler internals
* Check PGO support state across different build systems like CMake, Meson, Bazel, Buck, Cargo, etc.
* OCOLOS: https://people.ucsc.edu/~hlitz/papers/ocolos.pdf
* Discussion BOLT and Propeller: https://discourse.llvm.org/t/practical-compiler-optimizations-for-wsc-workshop-us-llvm-dev-meeting-2023/73998/
* What is https://prodfiler.com/ ? Check their work and probably write about them in the article
* https://github.com/aaupov/school_topics/blob/main/perfd.md - highlight this project as future BOLT improvement idea
* Play locally with Propeller: https://github.com/google/llvm-propeller and discuss integrating it into the cargo-pgo
* Make a research about PGO support in package managers like Conan, Vcpkg, etc.
* Do not forget to mention about monitoring actual workload and tracking - does it match your current PGO-profile or not
* Example of an application where PGO exists but not used for prebuilt binaries: https://github.com/ispc/ispc/issues/2687
* Add a note about unavailable LBR in virtual machines (that can limit AutoFDO usage in virtualized envs)
* Why GraalVM (and consequently PGO in Java world) have so low adoption? I need to investigate it. GraalVM metadata repo - https://github.com/oracle/graalvm-reachability-metadata
* Write about Clang (LLVM) vs GCC PGO implementations: GCC has more PGO switches, Clang has more recently-researched PGO-related things inside, etc.
* Prepare an answer in Angie (Nginx fork) repository about PGO
* https://github.com/stevinz/awesome-game-engine-dev
* Possible LTO issues in compilers: https://github.com/rust-lang/rust/pull/113433
* Good tracking issue for the Rustc optimizations: https://github.com/rust-lang/rust/issues/103595
* PGO-related issue in Rustc: https://github.com/tensorchord/pgvecto.rs/issues/43#issuecomment-1788324707
* Check Android (and Android-dependent) project for PGO/BOLT usage
* Add information about PGO missed optimizations like GCC (https://gcc.gnu.org/bugzilla/buglist.cgi?quicksearch=PGO&list_id=403385), Clang (TODO), Go compiler (TODO), Rustc (TODO)
* Add CachyOS project as an example where BOLT is integrated at most to the OS build pipelines
* Add "llvm-bolt" request for building in OS distributions
* An interesting case of a try to enable PGO for Foot in Void Linux: https://github.com/void-linux/void-packages/issues/29526
* Create a generic PGO + BOLT issue in Gentoo (links reporting in the bugzilla restricted until 15.11.2023)
* Add to OpenWRT request about building its GCC with PGO
* Check https://hpsfoundation.github.io/ and its projects. Probably they will be interested in PGO as well since it's HPC foundation
* Check Windows-based package managers like Chocolatey and others for PGO enablement
* A comment about build resources in Void Linux (why they do not use PGO): https://github.com/void-linux/void-packages/issues/39652#issuecomment-1265308710
* Become a Mageia maintainer to push PGO/PLO activity: https://bugs.mageia.org/show_bug.cgi?id=32511
* Check Firefox and Chromium derivatives like Thunderbird and Tor browser for PGO
* Cobol is a compiled language. What about enabling PGO for that? Check Cobol ecosystem
* Clear Linux PGO packages: https://github.com/search?q=org%3Aclearlinux-pkgs+%22pgo+%3D+true%22&type=code + PGO integration as a USE flag
* Check CRAN, *BSD (FreeBSD, OpenBSD, DragonflyBSD, etc) repositories for enabled PGO over projects
* Add to the article this idea: https://github.com/llvm/llvm-project/issues/58422
* Check for PGO compilers at https://cgit.freebsd.org/ports/tree/lang - can be a valuable source for the PGO experiments
