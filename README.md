# awesome-pgo
Various materials about Profile Guided Optimization (PGO) and other similar stuff like AutoFDO, Bolt, etc.

## Showcases

* [Chromium](https://www.chromium.org/Home/): 
  - https://blog.chromium.org/2016/10/making-chrome-on-windows-faster-with-pgo.html
  - https://blog.chromium.org/2020/08/chrome-just-got-faster-with-profile.html
* [Firefox](https://www.mozilla.org/en-US/firefox/new/): https://groups.google.com/g/mozilla.dev.platform/c/wwO48xXFx0A/m/ztg4i0DYAAAJ
* [Rust](https://www.rust-lang.org/) (`rustc` compiler):
  - https://blog.rust-lang.org/inside-rust/2020/11/11/exploring-pgo-for-the-rust-compiler.html
  - https://kobzol.github.io/rust/rustc/2022/10/27/speeding-rustc-without-changing-its-code.html
* [Python](https://www.python.org/): https://www.activestate.com/blog/python-performance-boost-using-profile-guided-optimization/
* [ScyllaDB](https://www.scylladb.com/): https://github.com/scylladb/scylladb/pull/10808
* [Linux kernel](https://kernel.org/):
  - [Paper](https://web.eecs.umich.edu/~takh/papers/ugur-one-profile-fits-all-osr-2022.pdf)
  - [Microsoft presentation](https://lpc.events/event/7/contributions/771/attachments/630/1193/Exploring_Profile_Guided_Optimization_of_the_Linux_Kernel.pdf)
  - [Phoronix post](https://www.phoronix.com/news/Clang-PGO-For-Linux-Next)
  - Yet another attemp to PGO Linux kernel: http://coolypf.com/kpgo.htm
  - [Gentoo Wiki](https://wiki.gentoo.org/wiki/Kernel/Optimization#Performance)
* [Vector](https://vector.dev/): https://github.com/vectordotdev/vector/issues/15631
* [YDB](https://ydb.tech/): https://github.com/ydb-platform/ydb/issues/140#issuecomment-1483943715
* [MariaDB](https://mariadb.org/): https://mariadb.com/files/MariaDBEnteprise-Profile-GuidedOptimization-20150401_0.pdf
* [PostgreSQL](https://www.postgresql.org/): see "postgresql_results.md" file in the repo
* [YugabyteDB](https://www.yugabyte.com/): https://github.com/yugabyte/yugabyte-db/commit/34cb791ed9d3d5f8ae9a9b9e9181a46485e1981d
* [GreptimeDB](https://greptime.com/product/db): https://github.com/GreptimeTeam/greptimedb/issues/1218
* [Bevy](https://bevyengine.org/): PGO-run (first) vs non-PGO (second) - [Pastebin](https://gist.github.com/zamazan4ik/bbffbdf9b10e2a281f5d5373347f48ef)
* [Wordpress](https://wordpress.com/): https://blog.bitnami.com/2016/08/intel-pgo-optimizations-lead-to-20.html
* [Databend](https://databend.rs/): https://github.com/datafuselabs/databend/issues/9387#issuecomment-1566210063

## Projects with already integrated PGO into their builds

* Rustc
* GCC
* Clang
* Python (`cpython`)
* V8 - [Bazel flag](https://github.com/v8/v8/blob/main/BUILD.gn#L184)
* Chromium
* Firefox
* PHP - [Makefile command](https://github.com/php/php-src/blob/master/build/Makefile.global#L138)

## PGO support in programming languages

* C and C++:
  - [GCC](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html#Instrumentation-Options)
  - [Clang](https://clang.llvm.org/docs/UsersManual.html#profile-guided-optimization)
  - [MSVC](https://learn.microsoft.com/en-us/cpp/build/profile-guided-optimizations)
  - [ICC](https://www.intel.com/content/www/us/en/docs/cpp-compiler/developer-guide-reference/2021-8/profile-guided-optimization-pgo.html)
  - [AOCC](https://www.amd.com/en/developer/aocc.html#documentation) (supports, but the documentation right now exists only as PDF files)
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

Possibly other compilers support PGO too. If you know any, please let me know.

## Beyond PGO

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

## Traps

* PGO
  - Requires multiple builds
  - Instrumented binaries work too slowly, so rarely could be used in production -> you need to prepare "sample" workload
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

* [Some notes about PGO](https://rigtorp.se/notes/pgo/)
* A rejected idea to integrate BOLT into `cpython` build: [link](https://github.com/faster-cpython/ideas/issues/224#issuecomment-1022371595)
* [cperl notes on LTO, PGO, BOLT](https://perl11.github.io/blog/bolt.html)

## Related projects

* [Awesome Machine learning in compilers](https://github.com/zwang4/awesome-machine-learning-in-compilers)

## TODO

* Add information about caveats of each method: PGO, AutoFDO, Bolt, Propeller, more advanced techniques
* Add more info about LTO and PGO state for packages in different Linux distros
* Add links to the existing projects how PGO is integrated into them
