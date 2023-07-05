# awesome-pgo
Various materials about Profile Guided Optimization (PGO) and other similar stuff like AutoFDO, Bolt, etc.

## Theory (a little bit)

* What is PGO:
  - [Wiki](https://en.wikipedia.org/wiki/Profile-guided_optimization)
  - [Microsoft docs](https://learn.microsoft.com/en-us/cpp/build/profile-guided-optimizations)

Also, you could find PDO (Profile Directed Optimization), FDO (Feedback Driven Optimization), FBO (Feedback Based Optimization), PDF (Profile Directed Feedback), PBO (Profile Based Optimization) - do not worry, that's just a PGO but with a different name. 

Additionally, I need to mention [Link-Time Optimization (LTO)](https://en.wikipedia.org/wiki/Interprocedural_optimization) since usually PGO is applied after LTO (since usually LTO is easier to enable and it brings significant performance and/or binary size improvements). PGO does not replace LTO but complements it. More information about LTO you could find in `lto.md`.

## PGO Showcases

### Browsers

* [Chromium](https://www.chromium.org/Home/): 
  - [Chromium blog 1](https://blog.chromium.org/2016/10/making-chrome-on-windows-faster-with-pgo.html)
  - [Chromium blog 2](https://blog.chromium.org/2020/08/chrome-just-got-faster-with-profile.html)
* [Firefox](https://www.mozilla.org/en-US/firefox/new/): [Google Groups thread](https://groups.google.com/g/mozilla.dev.platform/c/wwO48xXFx0A/m/ztg4i0DYAAAJ)
* [Edge](https://www.microsoft.com/en-us/edge): [MS Blog](https://blogs.windows.com/msedgedev/2020/09/23/faster-leaner-more-efficient-microsoft-edge/)
* [Opera](https://www.opera.com/): [Blog about PGO on Windows](https://blogs.opera.com/desktop/2016/12/even-faster-opera-for-windows-with-pgo/)

### Compilers and interpreters

* [Rust](https://www.rust-lang.org/) (`rustc` compiler):
  - [Rust Lang blog](https://blog.rust-lang.org/inside-rust/2020/11/11/exploring-pgo-for-the-rust-compiler.html)
  - [Kobzol blog](https://kobzol.github.io/rust/rustc/2022/10/27/speeding-rustc-without-changing-its-code.html)
* [Clang](https://clang.llvm.org/): [Docs](https://llvm.org/docs/HowToBuildWithPGO.html#introduction)
  - [KDE blog](https://planet.kde.org/lubos-lunak-2021-04-18-the-effect-of-cpu-link-time-lto-and-profile-guided-pgo-optimizations-on-the-compiler-itself/)
  - Libclang on Windows: [Article](https://cristianadam.eu/20160104/speeding-up-libclang-on-windows/)
  - BOLT effects on Clang speed: [Slides](https://llvm.org/devmtg/2022-11/slides/Lightning15-OptimizingClangWithBOLTUsingCMake.pdf) or [Android](https://android-review.linaro.org/plugins/gitiles/toolchain/llvm_android/+/f36c64eeddf531b7b1a144c40f61d6c9a78eee7a) experience
* [GCC](https://gcc.gnu.org/): 
  - [ArchLinux bugtracker](https://bugs.archlinux.org/task/56856). Numbers for Gcc 3.3 - be careful.
  - [NixOS experiments](https://github.com/NixOS/nixpkgs/pull/112928#issuecomment-778508138)
  - According to the experiments from a person in a local Telegra, chat with optimization GCC in Gentoo: +4% to compilation speed with LTO, +10% to compilation speed with PGO
* [Python](https://www.python.org/): [Blog](https://www.activestate.com/blog/python-performance-boost-using-profile-guided-optimization/)
* [PHP](https://www.php.net/): [Alibaba post](https://www.alibabacloud.com/forum/read-539)
* Perl ([cperl](https://github.com/perl11/cperl)): [Blog](https://perl11.github.io/blog/bolt.html)

### Developer tooling

* [Clangd](https://clangd.llvm.org/):
  - [JetBrains blog](https://blog.jetbrains.com/clion/2022/05/testing-3-approaches-performance-cpp_apps/)
  - [GitHub issue](https://github.com/llvm/llvm-project/issues/63486)
* [Clang-Tidy](https://clang.llvm.org/extra/clang-tidy/): [GitHub issue](https://github.com/llvm/llvm-project/issues/63486#issuecomment-1606147035)
* [lld](https://lld.llvm.org/): [GitHub issue](https://github.com/llvm/llvm-project/issues/63486#issuecomment-1607953028)
* [clang-format](https://clang.llvm.org/docs/ClangFormat.html): [GitHub comment](https://github.com/llvm/llvm-project/issues/63486#issuecomment-1617008106)
* [Uncrustify](https://uncrustify.sourceforge.net/): [GitHub issue](https://github.com/uncrustify/uncrustify/issues/4045)

### Operating systems

* [Linux kernel](https://kernel.org/):
  - [Paper](https://web.eecs.umich.edu/~takh/papers/ugur-one-profile-fits-all-osr-2022.pdf)
  - [Microsoft presentation](https://lpc.events/event/7/contributions/771/attachments/630/1193/Exploring_Profile_Guided_Optimization_of_the_Linux_Kernel.pdf)
  - [Phoronix post](https://www.phoronix.com/news/Clang-PGO-For-Linux-Next)
  - Yet another attempt to PGO Linux kernel: http://coolypf.com/kpgo.htm
  - [Gentoo Wiki](https://wiki.gentoo.org/wiki/Kernel/Optimization#Performance)
  - Optimizing Linux kernel with Clang. An [article](https://habr.com/ru/companies/ruvds/articles/696236/)(in Russian) and [results](https://github.com/h0tc0d3/linux_pgo)
  - From my experience and tests, PGO with Linux kernel could be tricky to perform and does not bring huge results for 3rd party applications(tested on Redis and PostgreSQL). Further testing is needed.
* Windows: 5-20% improvement according to the [presentation](https://lpc.events/event/7/contributions/771/attachments/630/1193/Exploring_Profile_Guided_Optimization_of_the_Linux_Kernel.pdf)

### Virtual machines

* [QEMU](https://www.qemu.org/): [Blog](https://www.talospace.com/2021/03/juicing-qemu-for-fun-and-profit.html)
* [CrosVM](https://crosvm.dev/book/): [Intel blog](https://www.intel.com/content/www/us/en/developer/articles/technical/enhance-vm-workloads-performance-with-pgo.html)

### Databases

* [PostgreSQL](https://www.postgresql.org/): see "postgresql_results.md" file in the repo
* [MariaDB](https://mariadb.org/):
  - [Official MariaDB article](https://mariadb.com/files/MariaDBEnteprise-Profile-GuidedOptimization-20150401_0.pdf)
  - [ClearLinux benchmarks](https://clearlinux.org/news-blogs/profile-guided-optimization-mariadb-benchmarks)
* [MySQL](https://www.mysql.com/): 
  - [oneAPI report](https://www.oneapi.io/blog/tencent-gains-up-to-85-performance-boost-for-mysql-using-intel-oneapi-tools/)
  - [A user report](https://bugs.mysql.com/bug.php?id=99781)
* [ClickHouse](https://clickhouse.com/): [GitHub issue](https://github.com/ClickHouse/ClickHouse/issues/44567#issuecomment-1589541199)
* [MongoDB](https://www.mongodb.com/): see "mongodb.md" file in the repo
* [Redis](https://redis.io/): see "redis.md" file in the repo
* [SQLite](https://www.sqlite.org/index.html):
  - See "sqlite.md" file in the repo for the detailed report
  - [SQLite forum discussion](https://sqlite.org/forum/forumpost/d26f4eba26)
* [YDB](https://ydb.tech/): [GitHub issue](https://github.com/ydb-platform/ydb/issues/140#issuecomment-1483943715)
* [FoundationDB](https://www.foundationdb.org/): [GitHub issue](https://github.com/apple/foundationdb/issues/1334)
* [DuckDB](https://duckdb.org/): [GitHub comment](https://github.com/duckdb/duckdb/discussions/7721#discussioncomment-6254284)
* [Memcached](https://memcached.org/): [GitHub issue](https://github.com/memcached/memcached/issues/1054)
* [DragonflyDB](https://www.dragonflydb.io/): [GitHub comment](https://github.com/dragonflydb/dragonfly/issues/592#issuecomment-1616777005)
* [YugabyteDB](https://www.yugabyte.com/): [GitHub commit](https://github.com/yugabyte/yugabyte-db/commit/34cb791ed9d3d5f8ae9a9b9e9181a46485e1981d)
* [ScyllaDB](https://www.scylladb.com/): [GitHub PR](https://github.com/scylladb/scylladb/pull/10808)
* [GreptimeDB](https://greptime.com/product/db): [GitHub issue](https://github.com/GreptimeTeam/greptimedb/issues/1218)
* [Databend](https://databend.rs/): [GitHub issue](https://github.com/datafuselabs/databend/issues/9387#issuecomment-1566210063)
* [Skytable](https://octave.skytable.io/): [GitHub issue](https://github.com/skytable/skytable/issues/300)
* [Tarantool](https://www.tarantool.io/): [GitHub issue](https://github.com/tarantool/tarantool/issues/8089#issuecomment-1580628168)

### Other

* [Vector](https://vector.dev/): [GitHub issue](https://github.com/vectordotdev/vector/issues/15631)
* [Handbrake](https://handbrake.fr/): [GitHub issue comment](https://github.com/HandBrake/HandBrake/issues/1072#issuecomment-865630524)
* [CP2K](https://www.cp2k.org/): [Docs](https://www.cp2k.org/howto:pgo)
* [Bevy](https://bevyengine.org/): PGO-run (first) vs non-PGO (second) - [Pastebin](https://gist.github.com/zamazan4ik/bbffbdf9b10e2a281f5d5373347f48ef). In these results you need to interpret performance decrease as "Release version is slower than PGOed" and performance increase as "Release version is faster than PGOed".
* [Wordpress](https://wordpress.com/): [Bitnami blog](https://blog.bitnami.com/2016/08/intel-pgo-optimizations-lead-to-20.html)
* Zstd and LZ4: [Blosc blog](https://www.blosc.org/posts/codecs-pgo/)
* Windows terminal: [GitHub PR](https://github.com/microsoft/terminal/pull/10071)
* Chess engines (Stockfish, Cfish, asmFish): [Reddit post](https://www.reddit.com/r/chess/comments/7uw699/speed_benchmark_stockfish_9_vs_cfish_vs_asmfish/)
* Multiple smaller benchmarks by Phoronix: [link](https://www.phoronix.com/review/gcc11-pgo-5950x)
* Benchmarks from OpenSUSE: [Docs](https://documentation.suse.com/sbp/all/html/SBP-GCC-10/index.html)
* Some logs parsing routines: [Blog](https://andre.arko.net/2022/03/13/parsing-logs-faster-with-rust-revisited/)

## Projects with already integrated PGO into their build scripts

* Rustc: a CI [script](https://github.com/rust-lang/rust/blob/master/src/ci/stage-build.py) for the multi-stage build
* GCC: a [part](https://github.com/gcc-mirror/gcc/blob/4832767db7897be6fb5cbc44f079482c90cb95a6/configure#L7818) in a "wonderful" `configure` script 
* Clang: [Docs](https://llvm.org/docs/HowToBuildWithPGO.html) 
* Python: 
  - CPython: [README](https://github.com/python/cpython#profile-guided-optimization)
  - Pyston: [README](https://github.com/pyston/pyston#building)
* V8: [Bazel flag](https://github.com/v8/v8/blob/main/BUILD.gn#L184)
* Chromium: [Script](https://chromium.googlesource.com/chromium/src/build/config/+/refs/heads/main/compiler/pgo/BUILD.gn)
* Firefox: [Docs](https://firefox-source-docs.mozilla.org/build/buildsystem/pgo.html)
* PHP - [Makefile command](https://github.com/php/php-src/blob/master/build/Makefile.global#L138) and old Centminmod [scripts](https://github.com/centminmod/php_pgo_training_scripts)
* MySQL: [CMake script](https://github.com/mysql/mysql-server/blob/8.0/cmake/fprofile.cmake)
* YugabyteDB: [GitHub commit](https://github.com/yugabyte/yugabyte-db/commit/34cb791ed9d3d5f8ae9a9b9e9181a46485e1981d)
* Zstd: [Makefile](https://github.com/facebook/zstd/blob/dev/programs/Makefile#L232)
* [Foot](https://codeberg.org/dnkl/foot): [Scripts](https://codeberg.org/dnkl/foot/src/branch/master/pgo)
* Windows Terminal: [GitHub PR](https://github.com/microsoft/terminal/pull/10071)

## PGO support in programming languages

* C and C++:
  - [GCC](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html#Instrumentation-Options)
  - [Clang](https://clang.llvm.org/docs/UsersManual.html#profile-guided-optimization)
  - [MSVC](https://learn.microsoft.com/en-us/cpp/build/profile-guided-optimizations)
  - [ICC](https://www.intel.com/content/www/us/en/docs/cpp-compiler/developer-guide-reference/2021-8/profile-guided-optimization-pgo.html)
  - [AOCC](https://www.amd.com/en/developer/aocc.html#documentation) (supports but the documentation right now exists only as PDF files)
* Rust:
  - [rustc](https://doc.rust-lang.org/rustc/profile-guided-optimization.html)
  - [cargo-pgo](https://github.com/Kobzol/cargo-pgo)
* Fortran:
  - [GCC](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html#Instrumentation-Options)
  - [Flang](https://clang.llvm.org/docs/UsersManual.html#profile-guided-optimization)
  - [IFC](https://www.intel.com/content/www/us/en/docs/fortran-compiler/developer-guide-reference/2023-0/profile-guided-optimization.html)
  - [IBM](https://www.ibm.com/docs/en/openxl-fortran-aix/17.1.0?topic=compatibility-profile-guided-optimization-pgo)
  - [AOCC](https://www.amd.com/en/developer/aocc.html#documentation) (supports, but the documentation right now exists only as PDF files)
* C#:
  - [Gist](https://gist.github.com/EgorBo/dc181796683da3d905a5295bfd3dd95b) from [EgorBo](https://gist.github.com/EgorBo)
  - [MS Blog](https://devblogs.microsoft.com/dotnet/conversation-about-pgo/)
  - [.Net 7](https://petabridge.com/blog/dotnet7-pgo-performance-improvements/)
* Java:
  - [GraalVM](https://www.graalvm.org/22.0/reference-manual/native-image/PGO/) (already [free to use](https://medium.com/graalvm/a-new-graalvm-release-and-new-free-license-4aab483692f5))
* Go:
  - [Go compiler](https://go.dev/doc/pgo) in Preview since Go 1.20, GA in [1.21](https://go.dev/blog/go1.21rc)
  - [GoLLVM](https://go.googlesource.com/gollvm) - [not yet](https://go.googlesource.com/gollvm/#thinltofdo)
  - GCCGO - unknown, but it should be possible to try
* Ada:
  - GNAT: should be possible, same as GCC
* [D](https://dlang.org/): [LDC docs](https://wiki.dlang.org/LDC_LLVM_profiling_instrumentation)
* Nim: [Nim forum](https://forum.nim-lang.org/t/6295)
* Ocaml: [almost no](https://github.com/ocaml/ocaml/issues/12200)
* [Zig](https://ziglang.org/): [no](https://github.com/ziglang/zig/issues/237)
* [V](https://vlang.io/): [kind of](https://github.com/vlang/v/issues/7024)
* [Red](https://www.red-lang.org/): [seems like not](https://github.com/red/red/issues/5333)

Possibly other compilers support PGO too. If you know any, please let me know.

## Are we PGO yet?

* ScyllaDB: https://github.com/scylladb/scylladb/pull/10808
* Tarantool: https://github.com/tarantool/tarantool/issues/8089
* ArangoDB: https://github.com/arangodb/arangodb/issues/17861
* DragonflyDB: https://github.com/dragonflydb/dragonfly/issues/592
* YDB: https://github.com/ydb-platform/ydb/issues/140
* Quickwit: https://github.com/quickwit-oss/quickwit/issues/3548
* YugabyteDB: https://github.com/yugabyte/yugabyte-db/issues/12763
* comdb2: https://github.com/bloomberg/comdb2/issues/3597
* ClickHouse: https://github.com/ClickHouse/ClickHouse/issues/44567
* SurrealDB: https://github.com/surrealdb/surrealdb/issues/1547
* Skytable: https://github.com/skytable/skytable/issues/300
* TiKV: https://github.com/tikv/tikv/issues/13990
* MongoDB: https://feedback.mongodb.com/forums/924280-database/suggestions/46127320-build-mongodb-with-pgo or https://jira.mongodb.org/browse/SERVER-3733
* Greptimedb: https://github.com/GreptimeTeam/greptimedb/issues/1218
* Vector: https://github.com/vectordotdev/vector/issues/15631
* Fluent-bit: https://github.com/fluent/fluent-bit/issues/6577
* rsyslog: https://github.com/rsyslog/rsyslog/issues/5048
* syslog-ng: https://github.com/syslog-ng/syslog-ng/issues/4511
* Redpanda: https://github.com/redpanda-data/redpanda/issues/7945
* parseable: https://github.com/parseablehq/parseable/issues/224
* Ruby: https://bugs.ruby-lang.org/issues/19256
* Clippy: https://github.com/rust-lang/rust-clippy/issues/10121
* mold: https://github.com/rui314/mold/issues/250
* Databend: https://github.com/datafuselabs/databend/issues/9387
* Alacritty: https://github.com/alacritty/alacritty/issues/6598
* Envoy: https://github.com/envoyproxy/envoy/issues/25500
* linkerd-proxy: https://github.com/linkerd/linkerd2/issues/10308
* Haproxy: https://github.com/haproxy/haproxy/issues/2047
* Tesseract: https://github.com/tesseract-ocr/tesseract/issues/3744
* Deno: https://github.com/denoland/rusty_v8/pull/1063
* Firecracker: https://github.com/firecracker-microvm/firecracker/issues/3456
* Codon: https://github.com/exaloop/codon/issues/137
* YTSaurus: https://github.com/ytsaurus/ytsaurus/issues/40
* DuckDB: https://github.com/duckdb/duckdb/discussions/7721
* Firebird: https://github.com/FirebirdSQL/firebird/issues/7631
* OpenObserve: https://github.com/openobserve/openobserve/issues/911
* RustPython: https://github.com/RustPython/RustPython/issues/5000
* Red: https://github.com/red/red/issues/5333
* [Elena-lang](https://elena-lang.github.io/): https://github.com/ELENA-LANG/elena-lang/issues/578
* BOLT: https://github.com/llvm/llvm-project/issues/63245
* Redscript: https://github.com/jac3km4/redscript/issues/94
* Nushell: https://github.com/nushell/nushell/issues/6943
* mrustc: https://github.com/thepowersgang/mrustc/issues/304
* PgCat: https://github.com/postgresml/pgcat/issues/479
* Cemu: https://github.com/cemu-project/Cemu/issues/797
* Odyssey: https://github.com/yandex/odyssey/issues/507
* AutoFDO: https://github.com/google/autofdo/issues/170
* Flang: https://github.com/llvm/llvm-project/issues/63376
* Sccache: https://github.com/mozilla/sccache/issues/1811
* Emscripten: https://github.com/emscripten-core/emscripten/issues/19671
* Tensorflow: https://github.com/tensorflow/tensorflow/issues/60944
* SerenityOS: https://github.com/SerenityOS/serenity/issues/19549
* pgvector: https://github.com/pgvector/pgvector/issues/168
* Tauri: https://github.com/tauri-apps/tauri/issues/7284
* LLVM projects (like Clangd, Clang-Tidy, lld): https://github.com/llvm/llvm-project/issues/63486
* cppcheck: https://trac.cppcheck.net/ticket/11672
* Rust Analyzer: https://github.com/rust-lang/rust-analyzer/issues/9412
* ccls: https://github.com/MaskRay/ccls/issues/942
* Gleam: https://github.com/gleam-lang/gleam/issues/2237
* Gluon: https://github.com/gluon-lang/gluon/issues/954
* Millet: https://github.com/azdavis/millet/issues/37
* Sorbet: https://github.com/sorbet/sorbet/issues/7102
* loxcraft: https://github.com/ajeetdsouza/loxcraft/issues/18
* Godot: https://github.com/godotengine/godot-proposals/issues/2610
* Uncrustify: https://github.com/uncrustify/uncrustify/issues/4045
* Redis: https://github.com/redis/redis/issues/12371
* KeyDB: https://github.com/Snapchat/KeyDB/issues/683
* Memcached: https://github.com/memcached/memcached/issues/1054
* SPIRV-Tools: https://github.com/KhronosGroup/SPIRV-Tools/issues/5303
* kphp: https://github.com/VKCOM/kphp/issues/862
* RocksDB: https://groups.google.com/g/rocksdb/c/j9iMFskUnpA
* FoundationDB: https://github.com/apple/foundationdb/issues/1334 or https://forums.foundationdb.org/t/profile-guided-optimization/4043

## BOLT showcases

* YDB: [GitHub comment](https://github.com/ydb-platform/ydb/issues/140)

## Are we BOLT yet?

* Clang in Gentoo: https://bugs.gentoo.org/907931

## LTO, PGO, BOLT, etc and provided by someone binaries

Well, it's hard to say, is your binary already LTO/PGO optimized or not. It depends on the multiple factors like upstream support for LTO/PGO, maintainers willing to enable these optimizations, etc. Usually the most obvious way to check it - just ask the question "Is the binary LTO/PGO optimized?" from the binary author (a person who built the binary). It could be your colleague (if you build programs on your own), build scripts from CI, maintainers from your favourite OS/repository (if you use provided by repos binaries), software developers (if you use downloaded from a site "official" binaries). Do not hesitate to ask!

### PGO adoption across projects

PGO usually is **not** enabled by the upstream developers due to lack of support for sample load or lack of resources for the multi-stage build. So please ask maintainers explicitly about PGO support addition.

### Other optimization techniques like BOLT

BOLT and others certainly are not enabled by default anywhere right now. So if you see a performance improvement from it - contact the upstream.

## Beyond PGO (could be covered here later as well)

* AutoFDO:
  - [Paper](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/45290.pdf)
  - [GitHub](https://github.com/google/autofdo)
* BOLT:
  - [Original paper](https://research.facebook.com/publications/bolt-a-practical-binary-optimizer-for-data-centers-and-beyond/)
  - [VESPA](https://research.facebook.com/publications/vespa-static-profiling-for-binary-optimization/)
  - [GitHub](https://github.com/llvm/llvm-project/blob/main/bolt/README.md)
* Propeller
  - [Propeller: A Profile Guided, Relinking Optimizer for
Warehouse-Scale Applications](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/578a590c3d797cd5d3fcd98f39657819997d9932.pdf)

## Traps

The biggest problem is "How to collect a good profile?". There are multiple ways for doing this:
- Prepare a reference workload. It could be quite difficult to create and maintain (since during the time it could become more and more different from your actual workload). However, for some loads like compilers load is usually predictable (compiling programs) so this way is good enough in this case. For other cases like databases the workload could hugely depend on the actual input from your users and users can change their queries for some reason. So be careful.
- Collect profile from your actual production. It could be difficult to do with a usual PGO since it's requires an instrumentation, and instrumentation binaries could work too slowly. If it's your case - you could try to use AutoFDO since it has a low overhead due to the underlying `perf` nature. But it also has its own limitations (usually Linux-only, less efficient than usual PGO, could be more buggy). E.g. Google uses AutoFDO for profiling all their services and has a lot of automation around sampling profiles at their scale, storing them, integration into CI pipelines, etc. But all these tooling is closed-source so you need to implement it from the scratch.

In my opinion, usually you should start with simple PGO via Instrumentation mode, especially if you upgrade your binaries seldomly. And only if Instrumentation starts to hurt you - start thinking about AutoFDO.

Another issue could be reproducibility. Since you are injecting some information from runtime (some execution counters based on your sample workload) you get more variables that could influence your binary. In this case you need to store somewhere in VCS your sample workload, probably collected profiles based on this workload, etc.

Other pitfalls include the following things:

* PGO
  - Requires multiple builds (at least two stages, in Context-Sensitive LLVM PGO ([CSPGO](https://reviews.llvm.org/D54175)) - three stages)
  - Instrumented binaries work too slowly, so rarely could be used in production -> you need to prepare a "sample" workload
  - For services sometimes PGO reports are not flushed to the disk properly, so you need to do it manually like [here](https://github.com/scylladb/scylladb/pull/10808/files#diff-bf1eacd22947b4daf9f4c2639427b8593d489f093eb1acfbba3e4cc1c9b0288bR427)
  - Reproducubility issues - could be important for some use-cases even more than performance
* AutoFDO
  - Huge memory consumption during profile conversion: [GitHub issue](https://github.com/google/autofdo/issues/162)
  - Supports only `perf`, so cannot be used with other profilers from different like Windows/macOS (support for other profilers could be implemented manually)
  - "Support" from Google is at least questionable: no regular releases, compilation [issues](https://github.com/google/autofdo/issues/157)
* Bolt
  - Huge memory usage during build: [GitHub issue](https://github.com/llvm/llvm-project/issues/61711)
  - For better results you need hardware/software with [LBR](https://lwn.net/Articles/680985/)/[BRS](https://lwn.net/Articles/877245/) support
  - There are a lot of bugs - be careful
* Propeller:
  - Too Google-oriented - could be hard to use outside of Google
  - Relies on the latest compiler developments, also could be unstable

## Useful links

* Implementation details of different PGO approaches in Clang: [Youtube](https://www.youtube.com/watch?v=GBtQrYx_Jbc), [slides](https://llvm.org/devmtg/2020-09/slides/PGO_Instrumentation.pdf)
* [Some notes about PGO](https://rigtorp.se/notes/pgo/)
* A rejected idea to integrate BOLT into `cpython` build: [link](https://github.com/faster-cpython/ideas/issues/224#issuecomment-1022371595)
* [cperl notes on LTO, PGO, BOLT](https://perl11.github.io/blog/bolt.html)
* `.profraw` internal details: [blog](https://leodido.dev/demystifying-profraw/)
* Slides about PGO: [link](https://assets.ctfassets.net/oxjq45e8ilak/41KrMkvfzUDEu6MDOEQL7q/dd4575d45a19aef27eebae411faa7952/PGO-___________________________________________________________.pdf) (in Russian)

## Communities

Here are the *incomplete* community list where you can find PGO-related advice with higher probability:

* Gentoo (chats, forums)
* ClearLinux (chats, forums)

## Related projects

* [Awesome Machine learning in compilers](https://github.com/zwang4/awesome-machine-learning-in-compilers)

## Where PGO did not help (according to my tests)

* [Catboost](https://catboost.ai/) - I think this is due to highly math-oriented nature of this. I did a test on `fit` and `calc` modes (trainig and evaluation, respectively) on `epsilon` dataset. In `calc` mode PGO for some reason made things even worse. Maybe, PGO could help in other modes but I didn't test it (yet).

## TODO

* Add more information about caveats of each method: PGO, AutoFDO, Bolt, Propeller, more advanced techniques
* Add more info about LTO and PGO state for packages in different Linux distros
* Add more links to the existing projects how PGO is integrated into them
* Write about PGO and library development. Maybe RocksDB or LevelDB as good examples of popular libraries?
* Try to use PGO on some modules like Nginx VTS (https://github.com/vozlt/nginx-module-vts)
* There is another PGO - PostGreSQL Operator from CrunchyDate ([GitHub](https://github.com/CrunchyData/postgres-operator)). It makes a bit harder to find information about Profile-Guided Optimization :)
* Add a chapter about PGO tips:
  - PGO profiles are not written to a disk due to signal handlers. Overwrite them or customize. I highly recommend to tune software to write a profile to a disk with a signal (call `__llvm_dump_profile` or [similar functions](https://github.com/llvm/llvm-project/blob/main/compiler-rt/lib/profile/InstrProfiling.h) if you use Clang)
  - Partial profile sets are useful too since they usually covers a lot of hot paths (LLD and ClickHouse as en example).
  - Merging multiple profiles - works well.
  - Running on a test suite - generally is not a good idea. Real-life workload usually would differ and a profile from the tests would not be so useful. However, if you have "real-life" tests - it's fine to use them as a profile workload.
  - PGO works well with libraries. However, will be a question - how to collect and distribute a profile for them? Especially if we are talking about general-purpose builds like in OS distros. Also, it's not easy to collect a profile from instrumented but non-instrumented binary (e.g. instrumented .so library which is used from Python software). Needs to be clarified but writing an instrumented wrapper right now is a recommended option.
  - PGO profiles does not depend on time! However, if your code has time-dependent paths, PGO profiles could differ due to "time stuff" like time issues on a build machine, different speed of different build machines, etc - be careful with that
  - PGO works fine without LTO (in normal compilers. MSVC does not belong to this family - you cannot use PGO without LTO there). So if LTO is too expensive to your build resources (especially if we are talking about FatLTO that consumes A LOT of RAM) - just use PGO without LTO, that's completely fine
  - It's hard to estimate how slow would be your application in the Instrumentation mode without actual testing. It hugely depends on the hot paths of your application, number of branches, etc. It's possible to predict based on some metrics of the application. But much easier just to test it :) Be careful, some application could be too slow in Instrumentation mode (like ClickHouse, Clangd or LLD)
  - When recompile software, would be useful to recompile 3rd parties with PGO as well. Here could be a challenge since often provided by OS packages dependencies are used. And it could be challenging to recompile them as well from the scratch (especially if we are talking about C and C++). Also, not for every software is obvious to understand - which 3rd parties are rebuilt from the scratch and which are taken from another place (like OS-provided or some package manager like Conan).
  - For reproducibility purposes you can just collect profile and commit it to the repo
