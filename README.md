# awesome-pgo
Various materials about Profile Guided Optimization (PGO) and other similar stuff like AutoFDO, Bolt, etc.

## Theory (a little bit)

* What is PGO:
  - [Wiki](https://en.wikipedia.org/wiki/Profile-guided_optimization)
  - [Microsoft docs](https://learn.microsoft.com/en-us/cpp/build/profile-guided-optimizations)

Also, you could find PDO (Profile Directed Optimization), FDO (Feedback Driven Optimization), FBO (Feedback Based Optimization), PDF (Profile Directed Feedback), PBO (Profile Based Optimization) - do not worry, that's just a PGO but with a different name. 

Additionally, I need to mention [Link-Time Optimization (LTO)](https://en.wikipedia.org/wiki/Interprocedural_optimization) since usually PGO is applied after LTO (since usually LTO is easier to enable and it brings significant performance and/or binary size improvements). PGO does not replace LTO but complements it. More information about LTO you could find in `lto.md`.

## PGO Showcases

Here I collect links to the articles/benchmarks/etc. with PGO on multiple projects (with numbers!).

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
  - According to the experiments from a person in a local Telegram chat with optimization GCC in Gentoo: +4% to compilation speed with LTO, +10% to compilation speed with PGO
* [Python](https://www.python.org/): [Blog](https://www.activestate.com/blog/python-performance-boost-using-profile-guided-optimization/)
* [PHP](https://www.php.net/):
  - [Alibaba post](https://www.alibabacloud.com/forum/read-539)
  - [Phoronix benchmarks](https://www.phoronix.com/news/Clear-Linux-PHP7-PGO-Opt)
* Perl ([cperl](https://github.com/perl11/cperl)): [Blog](https://perl11.github.io/blog/bolt.html)
* [Ruby](https://www.ruby-lang.org/): [Ruby Forum (post from 2006 with GCC 4.1)](https://www.ruby-forum.com/t/compiling-ruby-w-profile-guided-optimization/60564)
* [Lua](https://www.lua.org/): [Lua interpeter results - Reddit](https://www.reddit.com/r/lua/comments/151dtyu/profileguided_optimization_pgo_on_lua_interpreters/)
* [tfcompile](https://www.tensorflow.org/xla/tfcompile): [GitHub comment](https://github.com/tensorflow/tensorflow/issues/60944#issuecomment-1637143591)

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
  - [ASOS (Application Specific Operating System)](http://scis.scichina.com/en/2018/092102.pdf)
  - [Phoronix post](https://www.phoronix.com/news/Clang-PGO-For-Linux-Next)
  - Yet another attempt to PGO Linux kernel: http://coolypf.com/kpgo.htm
  - [Gentoo Wiki](https://wiki.gentoo.org/wiki/Kernel/Optimization#Performance)
  - Optimizing Linux kernel with Clang. An [article](https://habr.com/ru/companies/ruvds/articles/696236/)(in Russian) and [results](https://github.com/h0tc0d3/linux_pgo)
  - From my experience and tests, PGO with Linux kernel could be tricky to perform and does not bring huge results for 3rd party applications(tested on Redis and PostgreSQL). Further testing is needed. One possible idea - PGO was not applied right with GCC due to some .gcda find path issues. The test must be repeated with GCC and Clang.
* Windows: 5-20% improvement according to the [presentation](https://lpc.events/event/7/contributions/771/attachments/630/1193/Exploring_Profile_Guided_Optimization_of_the_Linux_Kernel.pdf)

### Virtual machines

* [QEMU](https://www.qemu.org/): [Blog](https://www.talospace.com/2021/03/juicing-qemu-for-fun-and-profit.html)
* [CrosVM](https://crosvm.dev/book/): [Intel blog](https://www.intel.com/content/www/us/en/developer/articles/technical/enhance-vm-workloads-performance-with-pgo.html)

### Databases

* [PostgreSQL](https://www.postgresql.org/):
  - See "postgresql_results.md" file in the repo.
  - [Mailing list thread](https://www.postgresql.org/message-id/flat/CAGjvy28Srze%2B-QGhUEEWsrpZA-8gMn_kf43dt7v_iUs%3Do-y2EQ%40mail.gmail.com)
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
* [RonDB](https://www.rondb.com/): [GitHub comment](https://github.com/logicalclocks/rondb/issues/335#issuecomment-1624188886)
* [ReDB](https://www.redb.org/): [GitHub comment](https://github.com/cberner/redb/issues/638#issuecomment-1641380707)

### Logging

* [Vector](https://vector.dev/): [GitHub issue](https://github.com/vectordotdev/vector/issues/15631)
* [Fluent-Bit](https://fluentbit.io/): [GitHub comment](https://github.com/fluent/fluent-bit/discussions/6638#discussioncomment-6419880)
* [Rsyslog](https://www.rsyslog.com/): [GitHub comment](https://github.com/rsyslog/rsyslog/issues/5048#issuecomment-1631807664)
* Some logs parsing routines: [Blog](https://andre.arko.net/2022/03/13/parsing-logs-faster-with-rust-revisited/)

### Other

* [Suricata](https://suricata.io/): [Slides](https://suricon.net/wp-content/uploads/2019/11/SURICON2019_Tools-and-Techniques-to-Simplify-Suricata-Performance-Testing.pdf)
* [Handbrake](https://handbrake.fr/): [GitHub issue comment](https://github.com/HandBrake/HandBrake/issues/1072#issuecomment-865630524)
* [CP2K](https://www.cp2k.org/): [Docs](https://www.cp2k.org/howto:pgo)
* [Bevy](https://bevyengine.org/): PGO-run (first) vs non-PGO (second) - [Pastebin](https://gist.github.com/zamazan4ik/bbffbdf9b10e2a281f5d5373347f48ef). In these results you need to interpret performance decrease as "Release version is slower than PGOed" and performance increase as "Release version is faster than PGOed".
* [Wordpress](https://wordpress.com/): [Bitnami blog](https://blog.bitnami.com/2016/08/intel-pgo-optimizations-lead-to-20.html)
* Zstd and LZ4: [Blosc blog](https://www.blosc.org/posts/codecs-pgo/)
* Windows terminal: [GitHub PR](https://github.com/microsoft/terminal/pull/10071)
* [Drill](https://github.com/fcsonline/drill): [GitHub issue](https://github.com/fcsonline/drill/issues/185)
* [Goose](https://www.tag1consulting.com/goose-podcasts-blogs-presentations-more): [Article](https://www.tag1consulting.com/blog/golden-goose-egg-compile-time-adventure)
* Chess engines (Stockfish, Cfish, asmFish): [Reddit post](https://www.reddit.com/r/chess/comments/7uw699/speed_benchmark_stockfish_9_vs_cfish_vs_asmfish/)
* Multiple smaller benchmarks by Phoronix: [link](https://www.phoronix.com/review/gcc11-pgo-5950x)
* Benchmarks from OpenSUSE: [Docs](https://documentation.suse.com/sbp/all/html/SBP-GCC-10/index.html)
* Bunch of LLVM test suite algorithms benchmarks: [Blog](https://johnnysswlab.com/tune-your-programs-speed-with-profile-guided-optimizations/)
* [ClamAV](https://www.clamav.net/): [BLog](https://nikkhokkho.sourceforge.io/static.php?page=ClamAVOpt)
* Mesa: [Mailing list](https://lists.freedesktop.org/archives/mesa-dev/2020-February/224096.html) about OpenGL benchmark. Worth reading the whole thread though.
* [hck](https://github.com/sstadick/hck): [README note](https://github.com/sstadick/hck#profile-guided-optimization)
* [Typst](https://typst.app/): [GitHub issue](https://github.com/typst/typst/issues/1733)
* [Cemu](https://cemu.info/): [GitHub comment](https://github.com/cemu-project/Cemu/issues/797#issuecomment-1521169155)

## Projects with already integrated PGO into their build scripts

* Rustc: a CI [script](https://github.com/rust-lang/rust/blob/master/src/ci/stage-build.py) for the multi-stage build
* GCC:
  - Official [docs](https://gcc.gnu.org/install/build.html), section "Building with profile feedback" (even AutoFDO build is supported)
  - A [part](https://github.com/gcc-mirror/gcc/blob/4832767db7897be6fb5cbc44f079482c90cb95a6/configure#L7818) in a "wonderful" `configure` script 
* Clang: [Docs](https://llvm.org/docs/HowToBuildWithPGO.html) 
* Python: 
  - CPython: [README](https://github.com/python/cpython#profile-guided-optimization)
  - Pyston: [README](https://github.com/pyston/pyston#building)
* V8: [Bazel flag](https://github.com/v8/v8/blob/main/BUILD.gn#L184)
* ChakraCore: [Scripts](https://github.com/chakra-core/ChakraCore/tree/master/Build/scripts/pgo)
* Chromium: [Script](https://chromium.googlesource.com/chromium/src/build/config/+/refs/heads/main/compiler/pgo/BUILD.gn)
* Firefox: [Docs](https://firefox-source-docs.mozilla.org/build/buildsystem/pgo.html)
   - Thunderbird has PGO support too
* PHP - [Makefile command](https://github.com/php/php-src/blob/master/build/Makefile.global#L138) and old Centminmod [scripts](https://github.com/centminmod/php_pgo_training_scripts)
* MySQL: [CMake script](https://github.com/mysql/mysql-server/blob/8.0/cmake/fprofile.cmake)
* YugabyteDB: [GitHub commit](https://github.com/yugabyte/yugabyte-db/commit/34cb791ed9d3d5f8ae9a9b9e9181a46485e1981d)
* Zstd: [Makefile](https://github.com/facebook/zstd/blob/dev/programs/Makefile#L232)
* [Foot](https://codeberg.org/dnkl/foot): [Scripts](https://codeberg.org/dnkl/foot/src/branch/master/pgo)
* Windows Terminal: [GitHub PR](https://github.com/microsoft/terminal/pull/10071)

## PGO support in programming languages and compilers

* C and C++:
  - [GCC](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html#Instrumentation-Options)
  - [Clang](https://clang.llvm.org/docs/UsersManual.html#profile-guided-optimization)
  - [MSVC](https://learn.microsoft.com/en-us/cpp/build/profile-guided-optimizations)
  - [ICC](https://www.intel.com/content/www/us/en/docs/cpp-compiler/developer-guide-reference/2021-8/profile-guided-optimization-pgo.html)
  - [AOCC](https://www.amd.com/en/developer/aocc.html#documentation) (supports but the documentation right now exists only as PDF files)
  - [Circle](https://www.circle-lang.org/) (not exactly a C++ compiler): no PGO support
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
* [Nim](https://nim-lang.org/): [Nim forum](https://forum.nim-lang.org/t/6295)
* Ocaml: [almost no](https://github.com/ocaml/ocaml/issues/12200)
* [Zig](https://ziglang.org/): [no](https://github.com/ziglang/zig/issues/237)
* [V](https://vlang.io/): [kind of](https://github.com/vlang/v/issues/7024)
* [Red](https://www.red-lang.org/): [seems like not](https://github.com/red/red/issues/5333)

Possibly other compilers support PGO too. If you know any, please let me know.

## Are we PGO yet?

Check "are_we_pgo_yet.md" file in the repo to check the PGO status in a project.

## BOLT showcases

Here I collect all results with applying LLVM BOLT to the projects (with numbers).

* YDB: [GitHub comment](https://github.com/ydb-platform/ydb/issues/140)
* Clang: [Slides](https://llvm.org/devmtg/2022-11/slides/Lightning15-OptimizingClangWithBOLTUsingCMake.pdf)

## Are we BOLT yet?

Just a list of BOLT-related issues in different projects. So you can estimate the BOLT state in your favourite open-source product.

* Clang in Gentoo: https://bugs.gentoo.org/907931

## LTO, PGO, BOLT, etc and provided by someone binaries

Well, it's hard to say, is your binary already LTO/PGO optimized or not. It depends on the multiple factors like upstream support for LTO/PGO, maintainers willing to enable these optimizations, etc. Usually the most obvious way to check it - just ask the question "Is the binary LTO/PGO optimized?" from the binary author (a person who built the binary). It could be your colleague (if you build programs on your own), build scripts from CI, maintainers from your favourite OS/repository (if you use provided by repos binaries), software developers (if you use downloaded from a site "official" binaries). Do not hesitate to ask!

### PGO adoption across projects

PGO usually is **not** enabled by the upstream developers due to lack of support for sample load or lack of resources for the multi-stage build. So please ask maintainers explicitly about PGO support addition.

### PGO adoption across Linux distros

Even if PGO is supported by a project, it does not mean that your favourite Linux distro builds this project with PGO enbaled. For this there are a lot of reasons: maintainer burden (because we are humans (yet)), build machines burden (in general you need to compile twice), reproducibility issues (like profile is an additional input to the build process and you need to make it reproducible), a maintainer just don't know about PGO, etc.

So here I will try to collect an information about the PGO status across the Linux distros for the projects that supports PGO in the upstream. If you didn't find your distro - don't worry! Just check it somehow (probably in some chats/distros' build systems, etc.) and report it here (e.g. via Issues) - I will add it to the list.

* GCC:
  - Note: PGO for GCC usually is not enabled for all architectures since it requires too much from the build systems
  - Debian: yes
  - Ubuntu: same as Debian
  - RedHat: Yes. And that is the reason why PGO is enabled for GCC in all RedHat-based distros.
  - Fedora: [yes](https://src.fedoraproject.org/rpms/gcc/blob/rawhide/f/gcc.spec#_1182)
  - Rocky Linux: [yes](https://git.rockylinux.org/staging/rpms/gcc-toolset-11-gcc/-/blob/r8/SPECS/gcc.spec#L1130)
  - Alma Linux: [yes](https://git.almalinux.org/rpms/gcc/src/branch/c8/SPECS/gcc.spec#L1164)
  - NixOS: [no](https://github.com/NixOS/nixpkgs/pull/112928)
  - OpenSUSE: [yes](https://build.opensuse.org/package/view_file/openSUSE:Factory/gcc12/gcc12.spec), see line `2414`
  - AltLinux: TODO: check :D
* Clang:
  - Binaries from LLVM are already PGO-optimized (according to the [note](https://apt.llvm.org/) about using "stage2" build - it's PGO-optimized build)
  - RedHat (CentOS Stream): [no](https://bugzilla.redhat.com/show_bug.cgi?id=2224925)
  - Fedora: [no](https://bugzilla.redhat.com/show_bug.cgi?id=2156679)
  - AlmaLinux: [no](https://bugs.almalinux.org/view.php?id=350)
  - Rocky Linux: [no](https://bugs.rockylinux.org/view.php?id=1618)
  - NixOS: [no](https://github.com/NixOS/nixpkgs/issues/208188)
  - Arch Linux: sent an email to the Clang maintainer in Arch Linux - no response yet
* CPython:
  - Fedora: [yes](https://src.fedoraproject.org/rpms/python3.11/blob/f38/f/python3.11.spec#_73). Also check [this](https://lists.fedoraproject.org/archives/list/devel@lists.fedoraproject.org/thread/2H4ZDRR326XAZ2EPCQKTNRMQYG5YZQ2K/) discussion. I guess other RedHat-based distros builds are the same for this package (however I didn't check it but Rocky Linux is the [same](https://git.rockylinux.org/staging/rpms/python3.11/-/blame/r8/SPECS/python3.11.spec#L70)).

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
* Overview of all kinds of PGO in LLVM: [link](https://aaupov.github.io/blog/2023/07/09/pgo)

## Communities

Here are the *incomplete* community list where you can find PGO-related advice with higher probability:

* Gentoo (chats, forums)
* ClearLinux (chats, forums)

## Related projects

* [Awesome Machine learning in compilers](https://github.com/zwang4/awesome-machine-learning-in-compilers)
* [CompilerGym](https://compilergym.com/): https://github.com/facebookresearch/CompilerGym/ (an interesting project about applying ML on compiler optimization flags)
* MLGO: A Machine Learning Framework for Compiler Optimization: [Google blog](https://ai.googleblog.com/2022/07/mlgo-machine-learning-framework-for.html)
* Phoronix Test Suite (PTS) integration with PGO: [GitHub](https://github.com/phoronix-test-suite/phoronix-test-suite/blob/master/pts-core/modules/pgo.php)
* An [article](https://thenewstack.io/how-to-use-bolt-binary-optimization-and-layout-tool/) about BOLT 

## Where PGO did not help (according to my tests)

* [Catboost](https://catboost.ai/) - I think this is due to highly math-oriented nature of this. I did a test on `fit` and `calc` modes (trainig and evaluation, respectively) on `epsilon` dataset. In `calc` mode PGO for some reason made things even worse. Maybe, PGO could help in other modes but I didn't test it (yet).

## Contribute

If you have an example where PGO shines (and where doesn't) - please open an issue and/or PR to the repo. It's important to collect as much as possible showcases about PGO!

## TODO

* Add more information about caveats of each method: PGO, AutoFDO, Bolt, Propeller, more advanced techniques
* Add more info about LTO and PGO state for packages in different Linux distros
* Add more links to the existing projects how PGO is integrated into them
* Get more information about PGO states in other browsers like Safari, Brave, Yandex.Browser and others
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
* Check Rustc PGO state in Linux distributions
* How to quickly check PGO support in the project: search over issues for "PGO", "Profile guided", "FDO". Also works grepping over the project for the same words or for PGO-related compiler flags like `fprofile-generate`/`fprofile-use`, etc.
