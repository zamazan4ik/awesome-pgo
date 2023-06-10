# awesome-pgo
Various materials about Profile Guided Optimization (PGO) and other similar stuff like AutoFDO, Bolt, etc.

## Theory (a little bit)

* What is PGO:
  - [Wiki](https://en.wikipedia.org/wiki/Profile-guided_optimization)
  - [Microsoft docs](https://learn.microsoft.com/en-us/cpp/build/profile-guided-optimizations)

Also, you could find PDO (Profile Directed Optimization), FDO (Feedback Driven Optimization), PDF (Profile Directed Feedback) - do not worry, that's just a PGO but with a different name. 

Additionally, I need to mention [Link-Time Optimization (LTO)](https://en.wikipedia.org/wiki/Interprocedural_optimization) since usually PGO is applied after LTO (since usually LTO is easier to enable and it brings significant performance and/or binary size improvements). PGO does not replace LTO but complements it.

## Showcases

* [Chromium](https://www.chromium.org/Home/): 
  - https://blog.chromium.org/2016/10/making-chrome-on-windows-faster-with-pgo.html
  - https://blog.chromium.org/2020/08/chrome-just-got-faster-with-profile.html
* [Firefox](https://www.mozilla.org/en-US/firefox/new/): https://groups.google.com/g/mozilla.dev.platform/c/wwO48xXFx0A/m/ztg4i0DYAAAJ
* [Rust](https://www.rust-lang.org/) (`rustc` compiler):
  - [Rust Lang blog](https://blog.rust-lang.org/inside-rust/2020/11/11/exploring-pgo-for-the-rust-compiler.html)
  - [Kobzol blog](https://kobzol.github.io/rust/rustc/2022/10/27/speeding-rustc-without-changing-its-code.html)
* [Python](https://www.python.org/): [Blog](https://www.activestate.com/blog/python-performance-boost-using-profile-guided-optimization/)
* [Clang](https://clang.llvm.org/): [Docs](https://llvm.org/docs/HowToBuildWithPGO.html#introduction)
  - Libclang on Windows: [Article](https://cristianadam.eu/20160104/speeding-up-libclang-on-windows/)
* [GCC](https://gcc.gnu.org/): [ArchLinux bugtracker](https://bugs.archlinux.org/task/56856). Numbers for Gcc 3.3 ... Sorry, I have no numbers for a newer version yet.
* [PHP](https://www.php.net/): [Alibaba post](https://www.alibabacloud.com/forum/read-539)
* [ScyllaDB](https://www.scylladb.com/): [GitHub PR](https://github.com/scylladb/scylladb/pull/10808)
* [Linux kernel](https://kernel.org/):
  - [Paper](https://web.eecs.umich.edu/~takh/papers/ugur-one-profile-fits-all-osr-2022.pdf)
  - [Microsoft presentation](https://lpc.events/event/7/contributions/771/attachments/630/1193/Exploring_Profile_Guided_Optimization_of_the_Linux_Kernel.pdf)
  - [Phoronix post](https://www.phoronix.com/news/Clang-PGO-For-Linux-Next)
  - Yet another attempt to PGO Linux kernel: http://coolypf.com/kpgo.htm
  - [Gentoo Wiki](https://wiki.gentoo.org/wiki/Kernel/Optimization#Performance)
  - From my experience and tests, PGO with Linux kernel could be tricky to perform and does not bring huge results (tested on Redis and PostgreSQL)
* [Vector](https://vector.dev/): [GitHub issue](https://github.com/vectordotdev/vector/issues/15631)
* [YDB](https://ydb.tech/): [GitHub issue](https://github.com/ydb-platform/ydb/issues/140#issuecomment-1483943715)
* [MariaDB](https://mariadb.org/):
  - [Official MariaDB article](https://mariadb.com/files/MariaDBEnteprise-Profile-GuidedOptimization-20150401_0.pdf)
  - [ClearLinux benchmarks](https://clearlinux.org/news-blogs/profile-guided-optimization-mariadb-benchmarks)
* [MySQL](https://www.mysql.com/): 
  - [oneAPI report](https://www.oneapi.io/blog/tencent-gains-up-to-85-performance-boost-for-mysql-using-intel-oneapi-tools/)
  - [A user report](https://bugs.mysql.com/bug.php?id=99781)
* [PostgreSQL](https://www.postgresql.org/): see "postgresql_results.md" file in the repo
* [YugabyteDB](https://www.yugabyte.com/): [GitHub commit](https://github.com/yugabyte/yugabyte-db/commit/34cb791ed9d3d5f8ae9a9b9e9181a46485e1981d)
* [GreptimeDB](https://greptime.com/product/db): [GitHub issue](https://github.com/GreptimeTeam/greptimedb/issues/1218)
* [Bevy](https://bevyengine.org/): PGO-run (first) vs non-PGO (second) - [Pastebin](https://gist.github.com/zamazan4ik/bbffbdf9b10e2a281f5d5373347f48ef)
* [Wordpress](https://wordpress.com/): [Bitnami blog](https://blog.bitnami.com/2016/08/intel-pgo-optimizations-lead-to-20.html)
* [Databend](https://databend.rs/): [GitHub issue](https://github.com/datafuselabs/databend/issues/9387#issuecomment-1566210063)
* [Skytable](https://octave.skytable.io/): [GitHub issue](https://github.com/skytable/skytable/issues/300)
* [Tarantool](https://www.tarantool.io/): [GitHub issue](https://github.com/tarantool/tarantool/issues/8089#issuecomment-1580628168)
* Zstd and LZ4: [Blosc blog](https://www.blosc.org/posts/codecs-pgo/)
* Chess engines (Stockfish, Cfish, asmFish): [Reddit post](https://www.reddit.com/r/chess/comments/7uw699/speed_benchmark_stockfish_9_vs_cfish_vs_asmfish/)
* Multiple smaller benchmarks by Phoronix: [link](https://www.phoronix.com/review/gcc11-pgo-5950x)
* Benchmarks from OpenSUSE: [Docs](https://documentation.suse.com/sbp/all/html/SBP-GCC-10/index.html)

## Projects with already integrated PGO into their builds

* Rustc: a CI [script](https://github.com/rust-lang/rust/blob/master/src/ci/stage-build.py) for the multi-stage build
* GCC: a [part](https://github.com/gcc-mirror/gcc/blob/4832767db7897be6fb5cbc44f079482c90cb95a6/configure#L7818) in a "wonderful" `configure` script 
* Clang: [Docs](https://llvm.org/docs/HowToBuildWithPGO.html) 
* Python (`cpython`): [README](https://github.com/python/cpython#profile-guided-optimization)
* V8: [Bazel flag](https://github.com/v8/v8/blob/main/BUILD.gn#L184)
* Chromium: [Script](https://chromium.googlesource.com/chromium/src/build/config/+/refs/heads/main/compiler/pgo/BUILD.gn)
* Firefox: [Docs](https://firefox-source-docs.mozilla.org/build/buildsystem/pgo.html)
* PHP - [Makefile command](https://github.com/php/php-src/blob/master/build/Makefile.global#L138) and old Centminmod [scripts](https://github.com/centminmod/php_pgo_training_scripts)
* MySQL: [CMake script](https://github.com/mysql/mysql-server/blob/8.0/cmake/fprofile.cmake)
* YugabyteDB: [GitHub commit](https://github.com/yugabyte/yugabyte-db/commit/34cb791ed9d3d5f8ae9a9b9e9181a46485e1981d)
* Zstd: [Makefile](https://github.com/facebook/zstd/blob/dev/programs/Makefile#L232)
* [Foot](https://codeberg.org/dnkl/foot): [Scripts](https://codeberg.org/dnkl/foot/src/branch/master/pgo)

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
  - [GraalVM](https://www.graalvm.org/22.0/reference-manual/native-image/PGO/) (only in Enterprise edition)
* Go:
  - [Go compiler](https://go.dev/doc/pgo) since Go 1.20, still in preview
  - [GoLLVM](https://go.googlesource.com/gollvm) - [not yet](https://go.googlesource.com/gollvm/#thinltofdo)
  - GCCGO - unknown, but it should be possible to try
* Ada:
  - GNAT: should be possible, same as GCC
* Nim: [Nim forum](https://forum.nim-lang.org/t/6295)
* Ocaml: [almost no](https://github.com/ocaml/ocaml/issues/12200)

Possibly other compilers support PGO too. If you know any, please let me know.

## Are we PGO yet?

* ScyllaDB: https://github.com/scylladb/scylladb/pull/10808
* Tarantool: https://github.com/tarantool/tarantool/issues/8089
* ArangoDB: https://github.com/arangodb/arangodb/issues/17861
* DragonflyDB: https://github.com/dragonflydb/dragonfly/issues/592
* YDB: https://github.com/ydb-platform/ydb/issues/140
* Quickwit: https://github.com/quickwit-oss/quickwit/issues/794
* YugabyteDB: https://github.com/yugabyte/yugabyte-db/issues/12763
* comdb2: https://github.com/bloomberg/comdb2/issues/3597
* ClickHouse: https://github.com/ClickHouse/ClickHouse/issues/44567
* SurrealDB: https://github.com/surrealdb/surrealdb/issues/1547
* Skytable: https://github.com/skytable/skytable/issues/300
* TiKV: https://github.com/tikv/tikv/issues/13990
* mongoDB: https://feedback.mongodb.com/forums/924280-database/suggestions/46127320-build-mongodb-with-pgo
* Greptimedb: https://github.com/GreptimeTeam/greptimedb/issues/1218
* Vector: https://github.com/vectordotdev/vector/issues/15631
* Fluent-bit: https://github.com/fluent/fluent-bit/issues/6577
* rsyslog: https://github.com/rsyslog/rsyslog/issues/5048
* Redpanda: https://github.com/redpanda-data/redpanda/issues/7945
* parseable: https://github.com/parseablehq/parseable/issues/224
* Ruby: https://bugs.ruby-lang.org/issues/19256
* Clippy: https://github.com/rust-lang/rust-clippy/issues/10121
* mold: https://github.com/rui314/mold/issues/250
* databend: https://github.com/datafuselabs/databend/issues/9387
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
* OpenObserve: https://github.com/openobserve/openobserve/issues/911

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

* PGO
  - Requires multiple builds
  - Instrumented binaries work too slowly, so rarely could be used in production -> you need to prepare a "sample" workload
  - For services sometimes PGO reports are not flushed to the disk properly, so you need to do it manually like [here](https://github.com/scylladb/scylladb/pull/10808/files#diff-bf1eacd22947b4daf9f4c2639427b8593d489f093eb1acfbba3e4cc1c9b0288bR427)
* AutoFDO
  - Huge memory consumption during profile conversion: https://github.com/google/autofdo/issues/162
  - Supports only `perf`, so cannot be used with other profilers from different like Windows/macOS (support for other profilers could be implemented manually)
  - "Support" from Google is at least questionable: no regular releases, compilation [issues](https://github.com/google/autofdo/issues/157)
* Bolt
  - Huge memory usage during build: https://github.com/llvm/llvm-project/issues/61711
  - For better results you need hardware/software with LBR/BRS support
  - There are a lot of bugs - be careful
* Propeller:
  - Too Google-oriented - could be hard to use outside of Google
  - Relies on the latest compiler developments, also could be unstable

## Useful links

* Implementation details of different PGO approaches in Clang: [Youtube](https://www.youtube.com/watch?v=GBtQrYx_Jbc), [slides](https://llvm.org/devmtg/2020-09/slides/PGO_Instrumentation.pdf)
* [Some notes about PGO](https://rigtorp.se/notes/pgo/)
* A rejected idea to integrate BOLT into `cpython` build: [link](https://github.com/faster-cpython/ideas/issues/224#issuecomment-1022371595)
* [cperl notes on LTO, PGO, BOLT](https://perl11.github.io/blog/bolt.html)
* Great bag of bugs in different software with LTO: [GitHub](https://github.com/InBetweenNames/gentooLTO/issues)

## Communities

Here are the *incomplete* community list where you can find PGO-related advice with higher probability:

* Gentoo (chats, forums)
* ClearLinux (chats, forums)

## Related projects

* [Awesome Machine learning in compilers](https://github.com/zwang4/awesome-machine-learning-in-compilers)

## TODO

* Add more information about caveats of each method: PGO, AutoFDO, Bolt, Propeller, more advanced techniques
* Add more info about LTO and PGO state for packages in different Linux distros
* Add links to the existing projects how PGO is integrated into them
* For some reason, ClickHouse does not show performance improvements from PGO. Continue investigating the issue
