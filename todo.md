# TODO

Here I collect random thoughts and ideas about further PGO investigation.

For now, the list is clear, and I am happy with that. But the article has nearly 100 TODOs...

* What about using PGO during fuzzing? To optimize the fuzzing efficiency but fuzzing explores all paths...
* <https://llvm.org/docs/CommandGuide/llvm-profdata.html#cmdoption-llvm-profdata-merge-text> - test this functionality and check the generated profile. Add to the article
* Sometimes people aren't interested enough in PGO without benchmarks - <https://github.com/oxc-project/oxc/issues/812#issuecomment-2116285554>
* Compiler statistics of PGO: <https://github.com/oxc-project/oxc/issues/812#issuecomment-2134080943>
* Add a chapter about starting LTO/PGO/PLO journey
* Propose PGO to https://git.deuxfleurs.fr/Deuxfleurs/garage - write why proposing to such places is a bit more difficult
* Perform PGO benchmarks on https://github.com/pydantic/jiter - https://github.com/pydantic/jiter/issues/123 + https://github.com/pydantic/jiter/issues/123#issuecomment-2255594560
* Check Rust no_std programs - how do they work with PGO? https://github.com/Amanieu/minicov - here can be some useful info about the topic
* Suggest PGO to https://github.com/phiresky/sqlite-zstd
* Write about next step in PGO advertisment - Go ecosystem and why exactly this PL
* Estimating PGO efficiency without benchmarks - no such tools but some ideas like metrics, ML, questionaries like "Are you Fabris Bellard?"
* https://app.radicle.xyz/nodes/seed.radicle.xyz/rad:z3gqcJUoA1n9HaHKufZs5FCSGazv5 - I don't know how to create an issue for the project
* Check PGO state in https://www.illumos.org/ . Maybe create one more "PGO all the packages" issue at their forum/bug tracker
* https://github.com/ferrumc-rs/ferrumc/issues/15#issuecomment-2348951292 - another comment that don't understand PGO's value for applications performance in general
* Add PGO request to https://sourceforge.net/projects/sdcc/ . Build the compiler with PGO and implement PGO for the compiler
* Add PGO request to https://github.com/FalkorDB/FalkorDB/ + https://github.com/FalkorDB/FalkorDB-core-rs
* Golang Conf 2024, a talk about PGO in Go - https://www.youtube.com/watch?v=FyZJlPMBFm8 , talk to the speaker about their PGO way in Go
* Rework wording in https://github.com/Kobzol/cargo-pgo/issues/60 + https://github.com/Kobzol/cargo-pgo/pull/61 - add to the article about influence of such "wrong" docs
* CachyOS optimizes more packages with PGO: https://www.phoronix.com/news/CachyOS-September-2024 - mention Phoronix too in the article
* Add about MLGO for regalloc: https://groups.google.com/g/llvm-dev/c/7jxuvu3WPl0 + https://github.com/google/ml-compiler-opt/blob/main/docs/regalloc-demo/demo.md + https://www.phoronix.com/forums/forum/phoronix/latest-phoronix-articles/1494741-cachyos-optimizing-more-packages-with-pgo-for-up-to-~10-better-performance?p=1494782#post1494782 . In the article already there is some information about MLGO - extend it
* Add to the article - https://www.reddit.com/r/cpp/comments/1fsj7a1/comment/lplvd07/
* Add https://pythonspeed.com/ to the article as a meta-idea
* Write about an idea of performance-oriented community and meetup initiatives
* Write about different benchmarks types: micro-bench, end-to-end bench, etc. and how they are suitable for PGO
* Update rinja docs: https://www.reddit.com/r/rust/comments/1ftx5iv/comment/lpy5v01/ - https://github.com/rinja-rs/rinja/pull/188
* Added documentation about PGO to a library: https://github.com/FlixCoder/serde-brief/pull/6
* AutoFDO + Propeller for the Linux kernel: https://www.phoronix.com/news/Linux-AutoFDO-Prop-v2
* AutoFDO regressions between LLVM releases: https://github.com/google/autofdo/issues/181#issuecomment-2270557096
* Proprietary tooling and PGO: https://community.intel.com/t5/Intel-oneAPI-DPC-C-Compiler/Boost-Performance-with-Hardware-Counter-Assisted-Profile-Guided/m-p/1635106/highlight/true#M4128
* https://github.com/spiceai/spiceai - suggest PGO
* closed the PGO issue even if they ARE interested in it (but not now): https://github.com/FuelLabs/fuel-core/issues/1369#issuecomment-2391410543
* Write to the author https://www.manning.com/books/latency about LTO, PGO and Pythonspeed resources
* Add LTO and PGO requests for https://github.com/greenbone/openvas-scanner/tree/main/rust (for PGO perform benches)
* Rinja example: PGO optimized e2e bench (https://github.com/mitsuhiko/minijinja/pull/588#issuecomment-2387957123) but failed in microbenchmarks: https://gist.github.com/zamazan4ik/c61ccdb86372bdc1ee7ab305381e2e28
* Another comment about "fixing the code" instead of PGO: https://github.com/optiprism-io/optiprism/discussions/292#discussioncomment-10878781
* Write RocksDB as a pretty common multilanguage example for using PGO (e.g. RocksDB in C++, the layer above in anything)
* After several rounds of optimizations PGO efficiency can become less: https://github.com/WGUNDERWOOD/tex-fmt/issues/22#issuecomment-2401824584
* Performance matters talk - https://www.youtube.com/watch?v=r-TLSBdHe1A (also I need to check the StrangeLoop conference)
* Perform PGO benches for https://github.com/tonbo-io/tonbo/tree/main/benches - blocked by https://github.com/tonbo-io/tonbo/issues/185
* Student's work in the PGO-like domains: http://mcst.ru/sites/default/files/u11/Master_2023_Levchenko.pdf (warn: in Russian)
* Sometimes people doesn't want to keep issues open... https://github.com/espruino/Espruino/issues/2392
* Write that PLO optimizers are not a replacement for PGO - it's an addition (also insert a citation from the BOLT paper from "abstract"). Perform some "PGO vs BOLT" + "PGO + BOLT vs PGO" + "BOLT vs PGO + BOLT" on SQLite - add these benches to the article
* A request example of enabling PGO for a tool with PGO support in the upstream
* 3rd party repos for PGO-optimized stuff for more boring projects like Fedora: https://discussion.fedoraproject.org/t/fedora-llvm-team-llvm-pgo-optimized/84361
* OS maintainers have more desire to enable PGO if an upstream project supports it: https://discord.com/channels/862292009423470592/1060577525929103510/1295008340652326934
* Mention Rust performance book in the article
* Add to the article: https://github.com/llvm/llvm-project/issues/57501
* Think about a talk about "LTO, PGO and PLO for Python native-based libs" for Python conferences. Also, the idea can be applicable for other langs like Ruby, etc.
* Suggest PGO to https://github.com/CrunchyData/pg_parquet
* Suggest PGO to https://github.com/jank-lang/jank/blob/main/compiler%2Bruntime/src/cpp/jank/runtime/core/munge.cpp
* After some time PGO issues can be closed: https://github.com/apache/horaedb/issues/1051
* Write to Ferrocene support - https://github.com/ferrocene/ferrocene/issues/22 . Add to the article as an example that proprietary companies don't care much about public feedback mechanisms. If I won't get an answer before November 10 - write to office@ferrous-systems.com directly - update the article according to the answer from Ferrocene
* Ask Tinygo (https://github.com/tinygo-org/tinygo) about PGO support in the compiler and for the compiler itself
* Add PGO request to https://docs.yscope.com/clp/main/dev-guide/components-core/
* TikTok videos about PGO, huh? :D
* Update article according to the comment: https://github.com/zamazan4ik/awesome-pgo/issues/8
* PGO profile reproducibility question: https://discourse.llvm.org/t/pgo-profile-reproducibility/82861
* Add https://discussion.fedoraproject.org/t/expand-usage-of-profile-guided-optimization-pgo-and-llvm-bolt-across-fedora-packages/133724/6 and https://kwk.github.io/pgo-experiment/ links to the article
* Add to the article that Google drops GCC support: https://www.mail-archive.com/gcc@gcc.gnu.org/msg102804.html
* LLVM BOLT and kernel: https://md.archlinux.org/s/maQL1mmOT
* Add https://cachyos.org/blog/2411-kernel-autofdo/ to the article
* CachyOS optimizations: https://discord.com/channels/862292009423470592/873309651364610118/1305227583696404510
* Lightweight PGO: https://groups.google.com/g/llvm-dev/c/r03Z6JoN7d4 . Was mentioned in https://rocm.docs.amd.com/projects/llvm-project/en/latest/LLVM/llvm/html/InstrProfileFormat.html
* Add my PGO slides to the awesome PGO repo
