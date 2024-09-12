# awesome-pgo
Various materials about Profile Guided Optimization (PGO) and other similar stuff like AutoFDO, Bolt, etc.

## !!!ARTICLE!!!

There is an (unfinished) article about all the details about PGO, PLO, etc. - [link](https://github.com/zamazan4ik/awesome-pgo/blob/main/article/article.md). With high chance, it will answer (almost) all your questions about PGO and PLO.

## How to fail with PGO?

![The PGO implementer](/the_pgo_implementer.png)

## Theory (a little bit)

* What is PGO:
  - [Wiki](https://en.wikipedia.org/wiki/Profile-guided_optimization)
  - [Microsoft docs](https://learn.microsoft.com/en-us/cpp/build/profile-guided-optimizations)

Also, you could find PDO (Profile Directed Optimization), FDO (Feedback Driven Optimization), FBO (Feedback Based Optimization), PDF (Profile Directed Feedback), PBO (Profile Based Optimization) - do not worry, that's just a PGO but with a different name.

Additionally, I need to mention [Link-Time Optimization (LTO)](https://en.wikipedia.org/wiki/Interprocedural_optimization) since usually PGO is applied after LTO (since usually LTO is easier to enable and it brings significant performance and/or binary size improvements). PGO does not replace LTO but complements it. More information about LTO can be found in `lto.md`.

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

* [Rust](https://www.rust-lang.org/) (the `rustc` compiler):
  - [Rust Lang blog](https://blog.rust-lang.org/inside-rust/2020/11/11/exploring-pgo-for-the-rust-compiler.html)
  - [Kobzol blog](https://kobzol.github.io/rust/rustc/2022/10/27/speeding-rustc-without-changing-its-code.html)
  - [Android blog](https://android-developers.googleblog.com/2023/12/faster-rust-toolchains-for-android.html)
* [Clang](https://clang.llvm.org/):
  - [Official documentation](https://llvm.org/docs/HowToBuildWithPGO.html#introduction)
  - [KDE blog](https://planet.kde.org/lubos-lunak-2021-04-18-the-effect-of-cpu-link-time-lto-and-profile-guided-pgo-optimizations-on-the-compiler-itself/)
  - Libclang on Windows: [Article](https://cristianadam.eu/20160104/speeding-up-libclang-on-windows/)
  - Homebrew benchmarks: [one](https://github.com/Homebrew/homebrew-core/issues/77975#issuecomment-859190783), [two](https://github.com/Homebrew/homebrew-core/pull/79454#issuecomment-869055079)
  - ScyllaDB benchmarks: [GitHub issue](https://github.com/scylladb/scylladb/issues/10985)
  - Clang on Windows: [Phoronix post](https://www.phoronix.com/news/LLVM-PGO-Windows-Build)
  - Fedora experiments: [GitHub repo](https://github.com/kwk/pgo-experiment//?tab=readme-ov-file#step6)
* [GCC](https://gcc.gnu.org/):
  - [ArchLinux bugtracker](https://bugs.archlinux.org/task/56856). Numbers for GCC 3.3 - could be outdated.
  - [NixOS experiments](https://github.com/NixOS/nixpkgs/pull/112928#issuecomment-778508138)
  - [PGO effects on devirtualization in C++](https://hubicka.blogspot.com/2014/04/devirtualization-in-c-part-5-feedback.html)
  - According to the experiments from a person in a local Telegram chat with optimization GCC in Gentoo: +4% to compilation speed with LTO, +10% to compilation speed with PGO
* [Python](https://www.python.org/):
  - [Blog](https://www.activestate.com/blog/python-performance-boost-using-profile-guided-optimization/)
  - [GitHub PR](https://github.com/void-linux/void-packages/pull/43791#issue-1699145188)
* Go (`go` compiler):
  - [Official blog](https://go.dev/doc/go1.21)
  - [Go compiler performance numbers](https://go-review.googlesource.com/c/go/+/495596)
* D:
  - DMD: [GitHub issue](https://github.com/dlang/dmd/pull/13791#issue-1164476335)
  - LDC: [GitHub comment](https://github.com/ldc-developers/ldc/discussions/4524#discussioncomment-7537608), [this](https://johanengelen.github.io/ldc/2016/07/15/Profile-Guided-Optimization-with-LDC.html) and [this](https://johanengelen.github.io/ldc/2016/04/13/PGO-in-LDC-virtual-calls.html) articles
* Julia: [GitHub PR](https://github.com/JuliaLang/julia/pull/45641#issue-1268010204)
* [PHP](https://www.php.net/):
  - [Alibaba post](https://www.alibabacloud.com/forum/read-539)
  - [Phoronix benchmarks](https://www.phoronix.com/news/Clear-Linux-PHP7-PGO-Opt)
* Perl ([cperl](https://github.com/perl11/cperl)): [Blog](https://perl11.github.io/blog/bolt.html)
* [Ruby](https://www.ruby-lang.org/): [Ruby Forum (post from 2006 with GCC 4.1)](https://www.ruby-forum.com/t/compiling-ruby-w-profile-guided-optimization/60564)
* [Lua](https://www.lua.org/): [Lua interpeter results - Reddit](https://www.reddit.com/r/lua/comments/151dtyu/profileguided_optimization_pgo_on_lua_interpreters/)
* [tfcompile](https://www.tensorflow.org/xla/tfcompile): [GitHub comment](https://github.com/tensorflow/tensorflow/issues/60944#issuecomment-1637143591)
* [SWI-Prolog](https://www.swi-prolog.org): [GitHub comment](https://github.com/macports/macports-ports/pull/20918#issuecomment-1767875432)
* Sage: [GitHub issue](https://github.com/adam-mcdaniel/sage/issues/70)

### Developer tooling

* [Clangd](https://clangd.llvm.org/):
  - [JetBrains blog](https://blog.jetbrains.com/clion/2022/05/testing-3-approaches-performance-cpp_apps/)
  - [GitHub issue](https://github.com/llvm/llvm-project/issues/63486)
* [Clang-Tidy](https://clang.llvm.org/extra/clang-tidy/): [GitHub issue](https://github.com/llvm/llvm-project/issues/63486#issuecomment-1606147035)
* [lld](https://lld.llvm.org/): [GitHub issue](https://github.com/llvm/llvm-project/issues/63486#issuecomment-1607953028)
* [clang-format](https://clang.llvm.org/docs/ClangFormat.html): [GitHub comment](https://github.com/llvm/llvm-project/issues/63486#issuecomment-1617008106)
* [Uncrustify](https://uncrustify.sourceforge.net/): [GitHub issue](https://github.com/uncrustify/uncrustify/issues/4045)
* Android tooling like `dex2oat`: [Medium](https://medium.com/androiddevelopers/pgo-for-native-android-applications-1a48a99e95d0)
* [typos](https://github.com/crate-ci/typos): [GitHub issue](https://github.com/crate-ci/typos/issues/827#issue-1888263250)
* [Rust Analyzer](https://rust-analyzer.github.io/): [GitHub comment](https://github.com/rust-lang/rust-analyzer/issues/9412#issuecomment-1298188709)
* pylyzer: [GitHub discussion](https://github.com/mtshiba/pylyzer/discussions/80#discussion-6500001)
* ctags: [GitHub comment](https://github.com/universal-ctags/ctags/issues/3849#issuecomment-2295323143)

### Operating systems

* [Linux kernel](https://kernel.org/):
  - [Paper](https://web.eecs.umich.edu/~takh/papers/ugur-one-profile-fits-all-osr-2022.pdf)
  - [Microsoft presentation](https://lpc.events/event/7/contributions/771/attachments/630/1193/Exploring_Profile_Guided_Optimization_of_the_Linux_Kernel.pdf)
  - [ASOS (Application Specific Operating System)](http://scis.scichina.com/en/2018/092102.pdf)
  - [TCP Stream perf](https://lpc.events/event/7/contributions/798/attachments/661/1214/LTO_PGO_and_AutoFDO_-_Plumbers_2020_-_Tolvanen_Wendling_Desaulniers.pdf)
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
  - [Blog](https://kristiannielsen.livejournal.com/17676.html)
* [MySQL](https://www.mysql.com/):
  - [oneAPI report](https://www.oneapi.io/blog/tencent-gains-up-to-85-performance-boost-for-mysql-using-intel-oneapi-tools/)
  - [A user report](https://bugs.mysql.com/bug.php?id=99781)
* [ClickHouse](https://clickhouse.com/): [GitHub issue](https://github.com/ClickHouse/ClickHouse/issues/44567#issuecomment-1589541199)
* [MongoDB](https://www.mongodb.com/): See "mongodb.md" file in the repo
* [Redis](https://redis.io/): See "redis.md" file in the repo
* [SQLite](https://www.sqlite.org/index.html):
  - See "sqlite.md" file in the repo for the detailed report
  - [SQLite forum discussion](https://sqlite.org/forum/forumpost/19870fae957d8c1a)
  - [sqlite-parquet-vtable PGO results](https://github.com/cldellow/sqlite-parquet-vtable/blob/master/README.md#building-release)
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
* [ReDB](https://www.redb.org/):
  - [GitHub comment in the main repo](https://github.com/cberner/redb/issues/638#issuecomment-1641380707)
  - [GitHub comment in NativeDB repo](https://github.com/vincent-herlemont/native_db/discussions/92#discussion-6053050) (has PGO results for ReDB too)
* [Nebula](https://www.nebula-graph.io/): [Docs](https://docs.nebula-graph.io/3.5.0/8.service-tuning/enable_autofdo_for_nebulagraph/)
* [Qdrant](https://qdrant.tech/):
  - [Microbenchmarks](https://github.com/qdrant/qdrant/issues/2354#issuecomment-1701936793)
* [OceanBase](https://en.oceanbase.com/): [GitHub comment](https://github.com/oceanbase/oceanbase/issues/1580#issuecomment-1732874282)
* [NativeDB](https://github.com/vincent-herlemont/native_db): [GitHub issue](https://github.com/vincent-herlemont/native_db/discussions/92#discussion-6053050)
* Dolt: [Blog](https://www.dolthub.com/blog/2024-02-02-profile-guided-optimization/)
* bbolt-rs: [GitHub comment](https://github.com/ambaxter/bbolt-rs/issues/2#issue-2291476885)
* libmdbx: [GitFlic issue](https://gitflic.ru/project/erthink/libmdbx/issue/14) (in Russian)
* candystore: [GitHub comment](https://github.com/sweet-security/candystore/issues/7#issue-2509793489)

### Logging

* [Vector](https://vector.dev/): [GitHub issue](https://github.com/vectordotdev/vector/issues/15631)
* [Fluent-Bit](https://fluentbit.io/): [GitHub comment](https://github.com/fluent/fluent-bit/discussions/6638#discussioncomment-6419880)
* [Rsyslog](https://www.rsyslog.com/): [GitHub comment](https://github.com/rsyslog/rsyslog/issues/5048#issuecomment-1631807664)
* Some logs parsing routines: [Blog](https://andre.arko.net/2022/03/13/parsing-logs-faster-with-rust-revisited/)

## Proxy

* Envoy: [GitHub comment](https://github.com/envoyproxy/envoy/issues/25500#issuecomment-1724584679)
* HAProxy:
  - [Clang results](https://github.com/haproxy/haproxy/issues/2047#issuecomment-1728265165)
  - [GCC results](https://github.com/haproxy/haproxy/issues/2047#issuecomment-1729606775)
  - [More Clang and GCC results](https://github.com/haproxy/haproxy/issues/2047#issuecomment-1729971279)
* Nginx: see "nginx.md" file in the repo
* Rathole: [GitHub discussion](https://github.com/rapiz1/rathole/discussions/287)
* httpd: see "httpd.md" file in the repo

### Other

* Unreal Engine:
  - [Release notes](https://docs.unrealengine.com/4.27/en-US/WhatsNew/Builds/ReleaseNotes/4_27/) (search for "PGO" on the page)
  - [Some notes on GitHub](https://github.com/bevyengine/bevy/issues/4586#issuecomment-1674097867)
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
* Multiple smaller benchmarks by Phoronix:
  - [GCC 8](https://www.phoronix.com/review/gcc-82-pgo)
  - [GCC 9](https://www.phoronix.com/news/Threadripper-GCC9-Optimized-Run)
  - [GCC 10](https://www.phoronix.com/news/GCC-10-PGO-3960X-Xmas-Eve)
  - [More GCC 10](https://www.phoronix.com/news/GCC-10.1-PGO-Optimizations)
  - [GCC 11](https://www.phoronix.com/review/gcc11-pgo-5950x)
  - [GCC 12](https://www.phoronix.com/news/GCC-12-PGO-TR-3990X-AMD)
* Benchmarks from OpenSUSE: [Docs](https://documentation.suse.com/sbp/all/html/SBP-GCC-10/index.html)
* Bunch of LLVM test suite algorithms benchmarks: [Blog](https://johnnysswlab.com/tune-your-programs-speed-with-profile-guided-optimizations/)
* [ClamAV](https://www.clamav.net/): [Blog](https://nikkhokkho.sourceforge.io/static.php?page=ClamAVOpt)
* Mesa: [Mailing list](https://lists.freedesktop.org/archives/mesa-dev/2020-February/224096.html) about OpenGL benchmark. Worth reading the whole thread though.
* [hck](https://github.com/sstadick/hck): [README note](https://github.com/sstadick/hck#profile-guided-optimization)
* [Typst](https://typst.app/): [GitHub issue](https://github.com/typst/typst/issues/1733)
* [Cemu](https://cemu.info/): [GitHub comment](https://github.com/cemu-project/Cemu/issues/797#issuecomment-1521169155)
* Pydantic-core: [GitHub comment](https://github.com/pydantic/pydantic-core/pull/678#issuecomment-1600427788)
* xz: [OpenMandriva forum](https://forum.openmandriva.org/t/pgo-xz/2543)
* libspng: [Docs](https://libspng.org/docs/build/#profile-guided-optimization)
* matchit: [GitHub issue](https://github.com/ibraheemdev/matchit/issues/38)
* QOAudio (Rust version): [GitHub issue](https://github.com/rafaelcaricio/qoaudio/issues/5)
* JSON libraries (`serde_json`, `rustc_serialize`, `simd-json`): [GitHub issue](https://github.com/serde-rs/json-benchmark/issues/23)
* XML libraries:
  - `xml-rs`: [GitHub issue](https://github.com/netvl/xml-rs/issues/228)
  - `quick-xml`: [GitHub issue](https://github.com/tafia/quick-xml/issues/632)
  - `roxmltree`: [GitHub issue](https://github.com/RazrFalcon/roxmltree/issues/118#issue-2295151269)
* `tonic`: [GitHub issue](https://github.com/hyperium/tonic/issues/1486)
* `tantivy`: [GitHub issue](https://github.com/quickwit-oss/tantivy/issues/2163)
* Lychee: [GitHub issue](https://github.com/lycheeverse/lychee/issues/1247)
* nushell: [GitHub comment](https://github.com/nushell/nushell/issues/6943#issuecomment-1592327662)
* delta: [GitHub comment](https://github.com/dandavison/delta/issues/1540)
* hurl: [GitHub comment](https://github.com/Orange-OpenSource/hurl/issues/1918)
* fd: [GitHub comment](https://github.com/sharkdp/fd/discussions/1384)
* [MRCC](https://mrcc.hu/): up to 40% performance boost with PGO according to the private benchmarks
* Broot: [GitHub issue](https://github.com/Canop/broot/issues/741)
* Geant4 (a CERN project):
  - "Testing AutoFDO for Geant4" ([slides](https://indico.cern.ch/event/394788/contributions/2357347/attachments/1368686/2074705/slides.pdf))
  - "Speeding up CMS simulations, reconstruction and HLT code using advanced compiler options" ([link](https://indico.cern.ch/event/1106990/contributions/4991214/))
* Youki: [GitHub issue](https://github.com/containers/youki/issues/2386)
* sd: [GitHub issue](https://github.com/chmln/sd/issues/237)
* frawk: [GitHub comment](https://github.com/ezrosent/frawk/issues/103#issuecomment-1751781163)
* bat: [GitHub issue](https://github.com/sharkdp/bat/issues/2701)
* jql: [GitHub issue](https://github.com/yamafaktory/jql/issues/236)
* htmlq: [GitHub issue](https://github.com/mgdm/htmlq/issues/70)
* ouch: [GitHub issue](https://github.com/ouch-org/ouch/issues/537)
* czkawka: [GitHub issue](https://github.com/qarmin/czkawka/issues/1099)
* quilkin: [GitHub comment](https://github.com/googleforgames/quilkin/issues/834#issuecomment-1772719309)
* grcov: [GitHub issue](https://github.com/mozilla/grcov/issues/1128)
* difftastic: [GitHub issue](https://github.com/Wilfred/difftastic/issues/588)
* Perspective: [GitHub discussion](https://github.com/finos/perspective/discussions/2406#discussioncomment-7419407)
* tquic: [GitHub issue](https://github.com/Tencent/tquic/issues/19)
* legba: [GitHub issue](https://github.com/evilsocket/legba/issues/10)
* Slint: [GitHub issue](https://github.com/slint-ui/slint/issues/3909)
* tsv-utils: [Study report](https://github.com/eBay/tsv-utils/blob/master/docs/lto-pgo-study.md)
* wgpu: [GitHub discussion](https://github.com/gfx-rs/wgpu/discussions/4692)
* Mesa: [Phoronix post](https://www.phoronix.com/news/Mesa-2020-PGO-LTO-Builds)
* lingua-rs: [GitHub discussion](https://github.com/pemistahl/lingua-rs/discussions/273)
* libtre: [FreeBSD Bugzilla comment](https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=133302#c0)
* ion:
  - Instrumented to Release: https://gist.github.com/zamazan4ik/200b179278bcad05528eb65340977781
  - PGO-optimized to Release: https://gist.github.com/zamazan4ik/a4c5b603c16ec9fe3427c9d26a50e3e5
  - Platform: Linux
  - ion version: `master` branch on `60bfb73351f0412c95b8ba2afe75e988514470a6` commit
* tokei: [GitHub issue](https://github.com/XAMPPRocky/tokei/issues/1039)
* qsv: [GitHub discussion](https://github.com/jqnatividad/qsv/discussions/1433)
* vtracer: [GitHub discussion](https://github.com/visioncortex/vtracer/discussions/67)
* ripgrep: [GitHub comment](https://github.com/BurntSushi/ripgrep/issues/1225#issuecomment-1827082307)
* lol-html: [GitHub issue](https://github.com/cloudflare/lol-html/issues/202#issue-2022322548)
* tokenizers: [GitHub issue](https://github.com/huggingface/tokenizers/issues/1426)
* Zen: [GitHub discussion](https://github.com/gorules/zen/discussions/109#discussion-6053432)
* native_model: [GitHub issue](https://github.com/vincent-herlemont/native_model/issues/50#issue-2073664067)
* pathfinding: [GitHub issue](https://github.com/evenfurther/pathfinding/issues/510#issue-2073786817)
* [HiGHS](https://highs.dev/): near 2-2.5% in `highs ../check/instances/greenbea.mps` workload
* lace: [GitHub issue](https://github.com/promised-ai/lace/issues/172#issue-2104154605)
* minitrace-rust: [GitHub issue](https://github.com/tikv/minitrace-rust/issues/195#issue-2104485855)
* needletail: [GitHub issue](https://github.com/onecodex/needletail/issues/72#issue-2127271081)
* logos: [GitHub issue](https://github.com/maciejhirsz/logos/issues/374#issue-2127942609)
* llrt: [GitHub issue](https://github.com/awslabs/llrt/issues/117#issue-2128010022)
* varpro: [GitHub issue](https://github.com/geo-ant/varpro/issues/27#issue-2129147949)
* awk: [LWN article](https://lwn.net/Articles/680985/)
* gawk: [GitHub commit](https://github.com/serpent-os/recipes/commit/b62808b8659f2a2747711550d5d72c72f623fc07)
* candy: [GitHub discussion](https://github.com/candy-lang/candy/discussions/953#discussion-6270541)
* axum: [GitHub dicussion](https://github.com/tokio-rs/axum/discussions/2625#discussion-6289057)
* rustls: [GitHub issue](https://github.com/rustls/rustls/issues/1823#issue-2161780638)
* python-libipld: [GitHub PR](https://github.com/MarshalX/python-libipld/pull/21)
* sqlparser-rs: [GitHub discussion](https://github.com/sqlparser-rs/sqlparser-rs/discussions/1163#discussion-6310515)
* arrow-datafusion: [GitHub discussion](https://github.com/apache/arrow-datafusion/discussions/9507#discussion-6341224)
* actson-rs: [GitHub issue](https://github.com/michel-kraemer/actson-rs/issues/33#issue-2187587484)
* oha: [GitHub PR](https://github.com/hatoo/oha/pull/268#issue-1805965954)
* rust_serialization_benchmark: [GitHub issue](https://github.com/djkoloski/rust_serialization_benchmark/issues/18#issuecomment-2002184386)
* ada-url: [GitHub issue](https://github.com/ada-url/rust/issues/61#issue-2190406874)
* struson: [GitHub discussion](https://github.com/Marcono1234/struson/discussions/59#discussion-6392876)
* ast-grep: [GitHub discussion](https://github.com/ast-grep/ast-grep/discussions/738)
* Symbolicator: [GitHub issue](https://github.com/getsentry/symbolicator/issues/1334)
* libjxl: [GitHub issue](https://github.com/libjxl/libjxl/issues/3456#issue-2225475925)
* nucleo: [GitHub discussion](https://github.com/helix-editor/nucleo/discussions/44#discussion-6483115)
* martin: [GitHub discussion](https://github.com/maplibre/martin/discussions/1294#discussion-6483582)
* serde-sqlite-jsonb: [GitHub discussion](https://github.com/lovasoa/serde-sqlite-jsonb/discussions/1#discussion-6500343)
* LibreOffice: [Blog](https://hubicka.blogspot.com/2014/09/linktime-optimization-in-gcc-part-3.html). The article is from 2014 - keep it in mind.
* A lot of insights, history, and great benchmarks for LTO and PGO efficiency in LLVM and GCC in various software (including Firefox and LibreOffice) from Honza Hubička: [GCC 4.8](https://hubicka.blogspot.com/2014/04/linktime-optimization-in-gcc-2-firefox.html), [GCC 5](https://hubicka.blogspot.com/2015/04/GCC5-IPA-LTO-news.html), [GCC 6 and Clang 3.9](https://hubicka.blogspot.com/2016/03/building-libreoffice-with-gcc-6-and-lto.html), [GCC8 and Clang 6](https://hubicka.blogspot.com/2018/12/even-more-fun-with-building-and.html), [GCC9](https://hubicka.blogspot.com/2019/05/gcc-9-link-time-and-inter-procedural.html)
* koto: [GitHub discussion](https://github.com/koto-lang/koto/discussions/312#discussion-6534310)
* prost: [GitHub discussion](https://github.com/tokio-rs/prost/discussions/1041#discussion-6549416)
* angle-grinder: [GitHub discussion](https://github.com/rcoh/angle-grinder/discussions/203#discussion-6583612)
* zune-image: [GitHub discussion](https://github.com/etemesi254/zune-image/discussions/193#discussion-6605993)
* graphql-lint: [GitHub issue](https://github.com/grafbase/grafbase/issues/1689#issue-2279168725)
* nom: [GitHub comment](https://github.com/rust-bakery/nom/issues/1763#issue-2292116078)
* prettyplease: [GitHub comment](https://github.com/dtolnay/prettyplease/issues/74#issue-2292685589)
* genson-rs: [GitHub comment](https://github.com/junyu-w/genson-rs/issues/1#issue-2312519250)
* resvg: [GitHub comment](https://github.com/RazrFalcon/resvg/issues/765#issue-2314946307)
* Cloudflare (internal services): [Blog](https://blog.cloudflare.com/reclaiming-cpu-for-free-with-pgo)
* rustwire: [GitHub comment](https://github.com/Basis-Health/rustwire/issues/2#issue-2317852317)
* Bend: [GitHub comment](https://github.com/HigherOrderCO/Bend/issues/517#issue-2319505187)
* Amber: [GitHub discussion](https://github.com/Ph0enixKM/Amber/discussions/147#discussion-6770832)
* Iggy-rs: [GitHub comment](https://github.com/orgs/iggy-rs/discussions/990#discussioncomment-9687418)
* html5ever: [GitHub comment](https://github.com/servo/html5ever/issues/545#issue-2365897370)
* Symbolica: [Zulip message](https://reform.zulipchat.com/#narrow/stream/345757-general/topic/Profile-Guided.20Optimization.20.28PGO.29.20for.20optimizing.20Symbolica/near/438962380)
* oxc: [GitHub comment](https://github.com/oxc-project/oxc/issues/812#issuecomment-2116233196)
* libvpx: [Chromium issue tracker](https://issues.chromium.org/issues/325103518#comment10)
* lady-deirdre: [GitHub comment](https://github.com/Eliah-Lakhin/lady-deirdre/discussions/7#discussion-6849845)
* musli: [GitHub discussion](https://github.com/udoprog/musli/discussions/166#discussion-6903244)
* limbo: [GitHub comment](https://github.com/penberg/limbo/issues/78#issue-2393613784)
* amber: [GitHub comment](https://github.com/dalance/amber/issues/343#issue-2393623648)
* OpenRadioss: [GitHub comment](https://github.com/orgs/OpenRadioss/discussions/393#discussioncomment-9976967)
* ieee80211-rs: [GitHub comment](https://github.com/Frostie314159/ieee80211-rs/issues/3#issue-2396647036)
* jiff, chrono, time: [GitHub discussion](https://github.com/BurntSushi/jiff/discussions/27#discussion-6965143)
* pulldown-latex: [GitHub comment](https://github.com/carloskiki/pulldown-latex/issues/6#issue-2447849419)
* vrl: [GitHub discussion](https://github.com/vectordotdev/vrl/discussions/977#discussion-7014828)
* Picodrive: [Habr comment](https://habr.com/ru/articles/138132/comments/#comment_4607641) (in Russian. 2x performance improvement)
* harper: [GitHub discussion](https://github.com/elijah-potter/harper/discussions/141#discussion-7142973)
* wildcard: [GitHub comment](https://github.com/cloudflare/wildcard/issues/7#issue-2510199025)
* GQL: [GitHub discussion](https://github.com/AmrDeveloper/GQL/discussions/116#discussion-7165051)
* trie-hard, radix-trie: [GitHub comment](https://github.com/michaelsproul/rust_radix_trie/issues/75#issue-2523191327)
* pingora: [GitHub discussion](https://github.com/cloudflare/pingora/discussions/379#discussion-7174584)
* tex-fmt: [GitHub comment](https://github.com/WGUNDERWOOD/tex-fmt/issues/22#issue-2523375103)

## Projects with already integrated PGO into their build scripts

Below you can find some examples of where and how PGO is integrated into different projects.

* Rustc: a CI [tool](https://github.com/rust-lang/rust/tree/master/src/tools/opt-dist) for the multi-stage build
* GCC:
  - Official [docs](https://gcc.gnu.org/install/build.html), section "Building with profile feedback" (even AutoFDO build is supported)
  - A [part](https://github.com/gcc-mirror/gcc/blob/master/configure#L7896) in a "wonderful" `configure` script.
* Clang:
  - [Docs](https://llvm.org/docs/HowToBuildWithPGO.html)
  - [MinGW build script](https://github.com/msys2/MINGW-packages/commit/4dd91d1d4dfef17f1f451c3a8f59303be855e4b5)
* Python:
  - CPython: [README](https://github.com/python/cpython#profile-guided-optimization)
  - Pyston: [README](https://github.com/pyston/pyston#building)
* Go: [Bash script](https://github.com/golang/go/blob/master/src/cmd/compile/profile.sh)
* Swift: [CMake script](https://github.com/apple/swift/blob/main/CMakeLists.txt#L364)
* V8: [Bazel flag](https://github.com/v8/v8/blob/main/BUILD.gn#L184)
* ChakraCore: [Scripts](https://github.com/chakra-core/ChakraCore/tree/master/Build/scripts/pgo)
* Chromium: [Script](https://chromium.googlesource.com/chromium/src/build/config/+/refs/heads/main/compiler/pgo/BUILD.gn)
* Firefox: [Docs](https://firefox-source-docs.mozilla.org/build/buildsystem/pgo.html)
   - Thunderbird has PGO support too
* PHP - [Makefile command](https://github.com/php/php-src/blob/master/build/Makefile.global#L138) and old Centminmod [scripts](https://github.com/centminmod/php_pgo_training_scripts)
* MySQL: [CMake script](https://github.com/mysql/mysql-server/blob/8.0/cmake/fprofile.cmake)
* YugabyteDB: [GitHub commit](https://github.com/yugabyte/yugabyte-db/commit/34cb791ed9d3d5f8ae9a9b9e9181a46485e1981d)
* FoundationDB: [Script](https://github.com/apple/foundationdb/blob/1a6114a66f3de508c0cf0a45f72f3687ba05750c/contrib/generate_profile.sh)
* Zstd: [Makefile](https://github.com/facebook/zstd/blob/dev/programs/Makefile#L232)
* [Foot](https://codeberg.org/dnkl/foot): [Scripts](https://codeberg.org/dnkl/foot/src/branch/master/pgo)
* Windows Terminal: [GitHub PR](https://github.com/microsoft/terminal/pull/10071)
* Pydantic-core: [GitHub PR](https://github.com/pydantic/pydantic-core/pull/741)
* file.d: [GitHub PR](https://github.com/ozontech/file.d/pull/469)
* OceanBase: [CMake flag](https://github.com/oceanbase/oceanbase/blob/master/cmake/Env.cmake#L55)
* ISPC: [CMake scipts](https://github.com/ispc/ispc/tree/main/superbuild)
* NodeJS: [Configure script](https://github.com/nodejs/node/commit/9be15559cc0bfe506d9cdfba4ad0f4beacf5ce17)
* Android Open Source Project (AOSP):
  - [Official documentation](https://source.android.com/docs/core/perf/pgo)
  - Committed PGO profiles: [repository](https://android.googlesource.com/toolchain/pgo-profiles/+/refs/heads/main)
* DMD: [Custom build rule](https://github.com/dlang/dmd/blob/master/compiler/src/build.d#L553)
* LDC: [GitHub action](https://github.com/ldc-developers/ldc/blob/master/.github/actions/2a-build-pgo/action.yml)
* tsv-utils: [Makefile](https://github.com/eBay/tsv-utils/blob/master/makefile#L56)
* Erlang OTP: [Makefile](https://github.com/erlang/otp/blob/master/Makefile.in#L484)
* Clingo (PGO enabled only in Spack): [Package recipe](https://github.com/spack/spack/blob/develop/var/spack/repos/builtin/packages/clingo-bootstrap/package.py#L96)
* SWI-Prolog:
  - [Script](https://github.com/SWI-Prolog/swipl-devel/blob/master/scripts/pgo-compile.sh)
  - [CMake module](https://github.com/SWI-Prolog/swipl-devel/blob/master/cmake/PGO.cmake)
* hck: [Justfile](https://github.com/sstadick/hck/blob/master/justfile#L27)
* oha: [GitHub PR](https://github.com/hatoo/oha/pull/268)
* Dolt: [Blog](https://www.dolthub.com/blog/2024-04-19-golang-pgo-builds-using-github-actions/)

## Project-specific documentation about PGO

Here we collect projects where PGO is described as an optimization option in the documentation:

* ClickHouse: https://clickhouse.com/docs/en/operations/optimizing-performance/profile-guided-optimization
* Databend: https://databend.rs/doc/contributing/pgo
* Vector: https://vector.dev/docs/administration/tuning/pgo/
* Nebula: https://docs.nebula-graph.io/3.5.0/8.service-tuning/enable_autofdo_for_nebulagraph/
* GCC: Official [docs](https://gcc.gnu.org/install/build.html), section "Building with profile feedback" (even AutoFDO build is supported)
* Clang:
  - https://llvm.org/docs/HowToBuildWithPGO.html
  - https://llvm.org/docs/AdvancedBuilds.html
* Rustc: https://rustc-dev-guide.rust-lang.org/building/optimized-build.html#profile-guided-optimization
* tsv-utils: https://github.com/eBay/tsv-utils/blob/master/docs/BuildingWithLTO.md

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
* Swift: [Seems like supports but I am not sure](https://github.com/apple/swift/blob/main/include/swift/Option/Options.td#L1322)
* Kotlin: [Seems like no](https://youtrack.jetbrains.com/issue/KT-63357/Kotlin-Native-Profile-Guided-Optimization-PGO-support)
* Ada:
  - GNAT: should be possible, same as GCC
* [D](https://dlang.org/): [LDC docs](https://wiki.dlang.org/LDC_LLVM_profiling_instrumentation)
* [Nim](https://nim-lang.org/): [Nim forum](https://forum.nim-lang.org/t/6295)
* Ocaml: [almost no](https://github.com/ocaml/ocaml/issues/12200)
* [Zig](https://ziglang.org/): [no](https://github.com/ziglang/zig/issues/237)
* [V](https://vlang.io/): [kind of](https://github.com/vlang/v/issues/7024)
* [Red](https://www.red-lang.org/): [seems like not](https://github.com/red/red/issues/5333)
* Pascal: [No](https://forum.lazarus.freepascal.org/index.php/topic,65162.0.html)
* Haskell:
  - GHC: [no](https://gitlab.haskell.org/ghc/ghc/-/issues/18393)

Possibly other compilers support PGO too. If you know any, please let me know.

### PGO support in build systems

Here we collect and track PGO integrations into build systems:

* Cargo: No built-in support but there is awesome [cargo-pgo](https://github.com/Kobzol/cargo-pgo)
* Bazel: Supports ([command-line reference](https://bazel.build/reference/command-line-reference#flag--fdo_instrument))
* CMake: No support yet ([GitLab issue](https://gitlab.kitware.com/cmake/cmake/-/issues/19273))
* Meson: Supports (`b_pgo` in the [docs](https://mesonbuild.com/Builtin-options.html)).
* SCons: No support yet ([GitHub discussion](https://github.com/SCons/scons/discussions/4437))

### Sampling PGO (AutoFDO) support

Here we collect information about supporting PGO via sampling across different compilers.

* C and C++:
  - GCC: supports
  - Clang: supports
* Rust:
  - rustc: supports, but marked unstable: [commit](https://github.com/rust-lang/rust/commit/a17193dbb931ea0c8b66d82f640385bce8b4929a), [unstable book](https://doc.rust-lang.org/beta/unstable-book/compiler-flags/profile_sample_use.html)

## Are we PGO yet?

Check "are_we_pgo_yet.md" file in the repo to check the PGO status in a project.

## BOLT showcases

Here I collect all results by applying LLVM BOLT to the projects (with numbers).

* Linux kernel:
  - [Phoronix article](https://www.phoronix.com/news/Linux-BOLT-5p-Performance)
  - [GitHub docs](https://github.com/llvm/llvm-project/blob/main/bolt/docs/OptimizingLinux.md)
* Rustc:
  - [Rustc itself (GitHub PR)](https://github.com/rust-lang/rust/pull/116352)
  - [LLVM in Rustc (Reddit)](https://www.reddit.com/r/rust/comments/y4w2kr/llvm_used_by_rustc_is_now_optimized_with_bolt_on/)
  - [Android blog](https://android-developers.googleblog.com/2023/12/faster-rust-toolchains-for-android.html)
* CPython: [GitHub PR](https://github.com/python/cpython/pull/95908)
* YDB: [GitHub comment](https://github.com/ydb-platform/ydb/issues/140)
* Clang:
  - [Slides](https://llvm.org/devmtg/2022-11/slides/Lightning15-OptimizingClangWithBOLTUsingCMake.pdf)
  - [Results on building Clang](https://github.com/ptr1337/llvm-bolt-scripts/blob/master/results.md)
  - [Linaro results](https://android-review.linaro.org/plugins/gitiles/toolchain/llvm_android/+/f36c64eeddf531b7b1a144c40f61d6c9a78eee7a)
  - [on AMD 7950X3D](https://github.com/llvm/llvm-project/issues/65010#issuecomment-1701255347)
  - [GitHub comment from LLVM](https://github.com/llvm/llvm-project/pull/71067#issuecomment-1791317245)
* LDC: [GitHub comment](https://github.com/ldc-developers/ldc/issues/4228#issuecomment-1334499428)
* HHVM, Proxygen and others: [Facebook paper](https://scontent-waw1-1.xx.fbcdn.net/v/t39.8562-6/240895848_219658560107211_6043870470092412798_n.pdf?_nc_cat=104&ccb=1-7&_nc_sid=ad8a9d&_nc_ohc=93feyEEEdC0AX8gvlSt&_nc_ht=scontent-waw1-1.xx&oh=00_AfAJh5n1HhZg32R-kRPqnpKJxHUSaFZZ2udLMFcT9MRQPw&oe=64CD8CE0)
* NodeJS: [Blog](https://aaupov.github.io/blog/2020/10/08/bolt-nodejs)
* Chromium: [Blog](https://aaupov.github.io/blog/2022/11/12/bolt-chromium)
* MySQL, MongoDB, memcached, Verilator: [Paper](https://people.ucsc.edu/~hlitz/papers/ocolos.pdf)
* ast-grep: [GitHub issue](https://github.com/ast-grep/ast-grep/discussions/738)
* Symbolicator: [GitHub issue](https://github.com/getsentry/symbolicator/issues/1334)
* Pango: [Gnome blog](https://blogs.gnome.org/chergert/2024/03/21/bolting-libraries/)
* pylyzer: [GitHub discussion](https://github.com/mtshiba/pylyzer/discussions/80#discussion-6500001)
* prettyplease: [GitHub comment](https://github.com/dtolnay/prettyplease/issues/74#issue-2292685589)
* bbolt-rs: [GitHub comment](https://github.com/ambaxter/bbolt-rs/issues/2#issue-2291476885)
* resvg: [GitHub comment](https://github.com/RazrFalcon/resvg/issues/765#issue-2314946307)

## Projects with already integrated BOLT into their build scripts

* Rustc: [GitHub PR](https://github.com/rust-lang/rust/pull/116352)
* CPython: [GitHub PR](https://github.com/python/cpython/pull/95908)
* Pyston:
  - [README](https://github.com/pyston/pyston#building)
  - [Makefile](https://github.com/pyston/pyston/blob/pyston_main/Makefile#L200)
* Clang: [CMake script](https://github.com/llvm/llvm-project/blob/main/clang/cmake/caches/BOLT.cmake)
* Linux kernel:
  - [LLVM branch for BOLTing the kernel](https://github.com/maksfb/llvm-project/commits/bolt-linux-kernel/)

## Are we BOLT yet?

Just a list of BOLT-related issues in different projects. So you can estimate the BOLT state in your favorite open-source product.

* Chromium: [Chromium bugtracker](https://bugs.chromium.org/p/chromium/issues/detail?id=1163978)
* Firefox: [Mozilla bugtracker](https://bugzilla.mozilla.org/show_bug.cgi?id=1789087)
  - The same for Propeller: [Mozilla bugtracker](https://bugzilla.mozilla.org/show_bug.cgi?id=1509314)
* NodeJS: [GitHub issue](https://github.com/nodejs/node/issues/50379)
* LDC: [GitHub issue](https://github.com/ldc-developers/ldc/issues/4228)
* GCC: [Bugzilla](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=112492)

## LTO, PGO, BOLT, etc and provided by someone binaries

Well, it's hard to say, is your binary already LTO/PGO optimized or not. It depends on multiple factors like upstream support for LTO/PGO, maintainers willing to enable these optimizations, etc. Usually, the most obvious way to check it - just ask the question "Is the binary LTO/PGO optimized?" from the binary author (a person who built the binary). It could be your colleague (if you build programs on your own), build scripts from CI, maintainers from your favorite OS/repository (if you use provided by repos binaries), software developers (if you use downloaded from a site "official" binaries). Do not hesitate to ask!

### PGO adoption across projects

PGO usually is **not** enabled by the upstream developers due to a lack of support for sample load or a lack of resources for the multi-stage build. So please ask maintainers explicitly about PGO support addition.

### PGO adoption across Linux distros

Even if PGO is supported by a project, it does not mean that your favorite Linux distro builds this project with PGO enabled. For this there are a lot of reasons: maintainer burden (because we are humans (yet)), build machines burden (in general you need to compile twice), reproducibility issues (like profile is an additional input to the build process and you need to make it reproducible), a maintainer just don't know about PGO, etc.

So here I will try to collect information about the PGO status across the Linux distros for the projects that support PGO in the upstream. If you didn't find your distro - don't worry! Just check it somehow (probably in some chats/distros' build systems, etc.) and report it here (e.g. via Issues) - I will add it to the list.

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
* Clang:
  - Binaries from LLVM are already PGO-optimized (according to the [note](https://apt.llvm.org/) about using "stage2" build - it's PGO optimized build)
  - RedHat (CentOS Stream): [no](https://bugzilla.redhat.com/show_bug.cgi?id=2224925)
  - Fedora: [no](https://bugzilla.redhat.com/show_bug.cgi?id=2156679)
  - AlmaLinux: [no](https://bugs.almalinux.org/view.php?id=350)
  - Rocky Linux: [no](https://bugs.rockylinux.org/view.php?id=1618)
  - NixOS: [no](https://github.com/NixOS/nixpkgs/issues/208188)
  - Arch Linux: sent an email to the Clang maintainer in Arch Linux - no response yet
* Rustc:
  - Fedora: [yes](https://src.fedoraproject.org/rpms/rust/blob/rawhide/f/rust.spec#_773)
* CPython:
  - Fedora: [yes](https://src.fedoraproject.org/rpms/python3.11/blob/f38/f/python3.11.spec#_73). Also, check [this](https://lists.fedoraproject.org/archives/list/devel@lists.fedoraproject.org/thread/2H4ZDRR326XAZ2EPCQKTNRMQYG5YZQ2K/) discussion. I guess other RedHat-based distro builds are the same for this package (however I didn't check it but Rocky Linux is the [same](https://git.rockylinux.org/staging/rpms/python3.11/-/blame/r8/SPECS/python3.11.spec#L70)).

## BOLT adoption across Linux distros

Here we track LLVM BOLT enablement across various projects in various OS-specific build scripts:

* Clang:
  - [Gentoo bugtracker](https://bugs.gentoo.org/907931)
* GCC: TODO
* Rustc:
  - Fedora: no
  - RedHat: no
* CPython: TODO
* Pyston: TODO  

Meta-issues about PGO and LLVM BOLT usage in different OSs and package managers:

* Fedora: [Bugzilla](https://bugzilla.redhat.com/show_bug.cgi?id=2249353)
* RedHat: [JIRA](https://issues.redhat.com/browse/RHEL-16308)
* ClearLinux: [GitHub issue](https://github.com/clearlinux/distribution/issues/2996)
* CachyOS ([Website](https://cachyos.org/)): according to the search over its GitHub repositories - they are trying to integrate BOLT as much as possible
* OpenSUSE: Cannot create an account to create a corresponding issue
* Ubuntu: [Ubuntu forums](https://ubuntuforums.org/showthread.php?t=2492480&p=14165131#post14165131)
* Alpine Linux: [Gitlab issue](https://gitlab.alpinelinux.org/alpine/aports/-/issues/15465)
* Mageia: [Bugzilla](https://bugs.mageia.org/show_bug.cgi?id=32511)
* Void Linux: [GitHub issue](https://github.com/void-linux/void-packages/issues/47215)
* Chromebrew: [GitHub discussion](https://github.com/chromebrew/chromebrew/discussions/8941)
* Homebrew: [GitHub discussion](https://github.com/orgs/Homebrew/discussions/4902)
* Spack: [GitHub discussion](https://github.com/spack/spack/discussions/41090)
* Vcpkg: [GitHub discussion](https://github.com/microsoft/vcpkg/discussions/35190)
* FreeBSD: [FreeBSD forum](https://forums.freebsd.org/threads/expand-profile-guided-optimization-pgo-usage-across-freebsd-packages.91034/)
* Conan: [GitHub issue](https://github.com/conan-io/conan-center-index/issues/21245)
* MacPorts: [Ticket](https://trac.macports.org/ticket/68746#ticket)
  - They said this question should be discussed in mailing lists
* LLVM-mingw: [GitHub issue](https://github.com/mstorsjo/llvm-mingw/issues/383)
* MinGW repo: [GitHub issue](https://github.com/msys2/MINGW-packages/issues/19273)
* CBL-Mariner: [GitHub discussion](https://github.com/microsoft/CBL-Mariner/discussions/8072)

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

The biggest problem is "How to collect a good profile?". There are multiple ways to do this:
- Prepare a reference workload. It could be quite difficult to create and maintain (since during the time it could become more and more different from your actual workload). However, for some loads like compilers load is usually predictable (compiling programs) so this way is good enough in this case. For other cases like databases the workload could hugely depend on the actual input from your users and users can change their queries for some reason. So be careful.
- Collect profile from your actual production. It could be difficult to do with a usual PGO since it requires an instrumentation, and instrumentation binaries could work too slowly. If it's your case - you could try to use AutoFDO since it has a low overhead due to the underlying `perf` nature. But it also has its own limitations (usually Linux-only, less efficient than usual PGO, could be more buggy). E.g. Google uses AutoFDO for profiling all their services and has a lot of automation around sampling profiles at their scale, storing them, integration into CI pipelines, etc. But all this tooling is closed-source so you need to implement it from the scratch.

In my opinion, usually you should start with simple PGO via Instrumentation mode, especially if you upgrade your binaries seldomly. And only if Instrumentation starts to hurt you - start thinking about AutoFDO.

Another issue could be reproducibility. Since you are injecting some information from runtime (some execution counters based on your sample workload) you get more variables that could influence your binary. In this case, you need to store somewhere in VCS your sample workload, probably collected profiles based on this workload, etc.

Other pitfalls include the following things:

* PGO
  - Requires multiple builds (at least two stages, in Context-Sensitive LLVM PGO ([CSPGO](https://reviews.llvm.org/D54175)) - three stages)
  - Instrumented binaries work too slowly, so rarely could be used in production -> you need to prepare a "sample" workload
  - For services sometimes PGO reports are not flushed to the disk properly, so you need to do it manually like [here](https://github.com/scylladb/scylladb/pull/10808/files#diff-bf1eacd22947b4daf9f4c2639427b8593d489f093eb1acfbba3e4cc1c9b0288bR427)
  - Reproducibility issues - could be important for some use cases even more than performance
  - Bugs. E.g. LLVM issues when PGO is combined with LTO - [GitHub issue](https://github.com/llvm/llvm-project/issues/57501)
* AutoFDO
  - Huge memory consumption during profile conversion: [GitHub issue](https://github.com/google/autofdo/issues/162)
  - Supports only `perf`, so cannot be used with other profilers from different like Windows/macOS (support for other profilers could be implemented manually)
  - "Support" from Google is at least questionable: no regular releases, compilation [issues](https://github.com/google/autofdo/issues/157)
* Bolt
  - Huge memory usage during build: [GitHub issue](https://github.com/llvm/llvm-project/issues/61711)
  - For better results, you need hardware/software with [LBR](https://lwn.net/Articles/680985/)/[BRS](https://lwn.net/Articles/877245/) support
  - There are a lot of bugs - be careful
* Propeller:
  - Too Google-oriented - could be hard to use outside of Google
  - Relies on the latest compiler developments, also unstable

## Useful links

* Implementation details of different PGO approaches in Clang: [Youtube](https://www.youtube.com/watch?v=GBtQrYx_Jbc), [slides](https://llvm.org/devmtg/2020-09/slides/PGO_Instrumentation.pdf)
* [Some notes about PGO](https://rigtorp.se/notes/pgo/)
* A rejected idea to integrate BOLT into `cpython` build: [link](https://github.com/faster-cpython/ideas/issues/224#issuecomment-1022371595)
* [cperl notes on LTO, PGO, BOLT](https://perl11.github.io/blog/bolt.html)
* `.profraw` internal details: [blog](https://leodido.dev/demystifying-profraw/)
* Slides about PGO from C++ Russia 2021 (Pavel Kosov): [slides](https://github.com/kpdev/rnd_code_blog_utils/blob/main/util/PGO_CPP_RUS_15-11-2021.pdf) (in Russian), [video](https://www.youtube.com/watch?v=CvbIPGSnes8)
* Overview of all kinds of PGO in LLVM: [link](https://aaupov.github.io/blog/2023/07/09/pgo)
* MSVC insights about PGO (a video from 2012): [Microsoft learn](https://learn.microsoft.com/en-us/shows/c9-goingnative/c9goingnative-12-c-build-2012-inside-profile-guided-optimization)

## Communities

Here is the *incomplete* community list where you can find PGO-related advice with higher probability:

* Gentoo (chats, forums)
* ClearLinux (chats, forums)

## Related projects

* [Awesome Machine learning in compilers](https://github.com/zwang4/awesome-machine-learning-in-compilers)
* [CompilerGym](https://compilergym.com/): https://github.com/facebookresearch/CompilerGym/ (an interesting project about applying ML on compiler optimization flags)
* MLGO: A Machine Learning Framework for Compiler Optimization: [Google blog](https://ai.googleblog.com/2022/07/mlgo-machine-learning-framework-for.html)
* Phoronix Test Suite (PTS) integration with PGO: [GitHub](https://github.com/phoronix-test-suite/phoronix-test-suite/blob/master/pts-core/modules/pgo.php)
* An [article](https://thenewstack.io/how-to-use-bolt-binary-optimization-and-layout-tool/) about BOLT
* Nvidia paper about PGO in gamedev: [Publication](https://research.nvidia.com/publication/2021-07_cooperative-profile-guided-optimization)

## Where PGO did not help (according to my tests)

* [Catboost](https://catboost.ai/) - I think this is due to the highly math-oriented nature of this. I did a test on `fit` and `calc` modes (training and evaluation, respectively) on `epsilon` dataset. In the `calc` mode PGO for some reason made things even worse. Maybe, PGO could help in other modes but I didn't test it (yet).

## Contribute

If you have an example where PGO shines (and where doesn't) - please open an issue and/or PR to the repo. It's important to collect as many as possible showcases about PGO!
