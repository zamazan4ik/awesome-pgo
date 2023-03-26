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
* [Linux kernel](https://kernel.org/): https://web.eecs.umich.edu/~takh/papers/ugur-one-profile-fits-all-osr-2022.pdf
* [Vector](https://vector.dev/): https://github.com/vectordotdev/vector/issues/15631
* [YDB](https://ydb.tech/): https://github.com/ydb-platform/ydb/issues/140#issuecomment-1483943715
* [YugabyteDB](https://www.yugabyte.com/): https://github.com/yugabyte/yugabyte-db/commit/34cb791ed9d3d5f8ae9a9b9e9181a46485e1981d
* [GreptimeDB](https://greptime.com/product/db): https://github.com/GreptimeTeam/greptimedb/issues/1218

## Projects with already integrated PGO into their builds

* Rustc
* GCC
* Clang
* Python
* V8
* Chromium
* Firefox

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
* Java:
  - [GraalVM](https://www.graalvm.org/22.0/reference-manual/native-image/PGO/) (only in Enterprise edition)
* Go:
  - [Go compiler](https://go.dev/doc/pgo) since Go 1.20, still in preview
* Ada:
  - GNAT: same as GCC

Possibly other compilers support PGO too. If you know any, please let me know.

## Beyond PGO

* AutoFDO:
  - [Paper](https://research.google/pubs/pub45290/)
  - [GitHub](https://github.com/google/autofdo)
* BOLT:
  - [Original paper](https://research.facebook.com/publications/bolt-a-practical-binary-optimizer-for-data-centers-and-beyond/)
  - [VESPA](https://research.facebook.com/publications/vespa-static-profiling-for-binary-optimization/)
  - [GitHub](https://github.com/llvm/llvm-project/blob/main/bolt/README.md)

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

## Useful links

* https://rigtorp.se/notes/pgo/

## Related projects

* [Awesome Machine learning in compilers](https://github.com/zwang4/awesome-machine-learning-in-compilers)

## TODO

* Add information about caveats of each method: PGO, AutoFDO, Bolt, more advanced techniques
* Add more info about PGO state for packages in different Linux distros