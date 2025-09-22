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
* Write about next step in PGO advertisment - Go ecosystem and why exactly this PL
* Estimating PGO efficiency without benchmarks - no such tools but some ideas like metrics, ML, questionaries like "Are you Fabris Bellard?"
* Check PGO state in https://www.illumos.org/ . Maybe create one more "PGO all the packages" issue at their forum/bug tracker
* https://github.com/ferrumc-rs/ferrumc/issues/15#issuecomment-2348951292 - another comment that don't understand PGO's value for applications performance in general
* Add PGO request to https://sourceforge.net/projects/sdcc/ . Build the compiler with PGO and implement PGO for the compiler
* Add PGO request to https://github.com/FalkorDB/FalkorDB/ + https://github.com/FalkorDB/FalkorDB-core-rs
* Golang Conf 2024, a talk about PGO in Go - https://www.youtube.com/watch?v=FyZJlPMBFm8 , talk to the speaker about their PGO way in Go
* Add about MLGO for regalloc: https://groups.google.com/g/llvm-dev/c/7jxuvu3WPl0 + https://github.com/google/ml-compiler-opt/blob/main/docs/regalloc-demo/demo.md + https://www.phoronix.com/forums/forum/phoronix/latest-phoronix-articles/1494741-cachyos-optimizing-more-packages-with-pgo-for-up-to-~10-better-performance?p=1494782#post1494782 . In the article already there is some information about MLGO - extend it
* Add https://pythonspeed.com/ to the article as a meta-idea
* Write about different benchmarks types: micro-bench, end-to-end bench, etc. and how they are suitable for PGO
* Update rinja docs: https://www.reddit.com/r/rust/comments/1ftx5iv/comment/lpy5v01/ - https://github.com/rinja-rs/rinja/pull/188
* Added documentation about PGO to a library: https://github.com/FlixCoder/serde-brief/pull/6
* AutoFDO + Propeller for the Linux kernel: https://www.phoronix.com/news/Linux-AutoFDO-Prop-v2
* Proprietary tooling and PGO: https://community.intel.com/t5/Intel-oneAPI-DPC-C-Compiler/Boost-Performance-with-Hardware-Counter-Assisted-Profile-Guided/m-p/1635106/highlight/true#M4128
* Write to the author https://www.manning.com/books/latency about LTO, PGO and Pythonspeed resources
* Add PGO request for https://github.com/greenbone/openvas-scanner/tree/main/rust (for PGO perform benches)
* Rinja example: PGO optimized e2e bench (https://github.com/mitsuhiko/minijinja/pull/588#issuecomment-2387957123) but failed in microbenchmarks: https://gist.github.com/zamazan4ik/c61ccdb86372bdc1ee7ab305381e2e28
* Another comment about "fixing the code" instead of PGO: https://github.com/optiprism-io/optiprism/discussions/292#discussioncomment-10878781
* Write RocksDB as a pretty common multilanguage example for using PGO (e.g. RocksDB in C++, the layer above in anything)
* After several rounds of optimizations PGO efficiency can become less: https://github.com/WGUNDERWOOD/tex-fmt/issues/22#issuecomment-2401824584
* Performance matters talk - https://www.youtube.com/watch?v=r-TLSBdHe1A (also I need to check the StrangeLoop conference)
* Perform PGO benches for https://github.com/tonbo-io/tonbo/tree/main/benches - blocked by https://github.com/tonbo-io/tonbo/issues/185
* Student's work in the PGO-like domains: http://mcst.ru/sites/default/files/u11/Master_2023_Levchenko.pdf (warn: in Russian)
* Write that PLO optimizers are not a replacement for PGO - it's an addition (also insert a citation from the BOLT paper from "abstract"). Perform some "PGO vs BOLT" + "PGO + BOLT vs PGO" + "BOLT vs PGO + BOLT" on SQLite - add these benches to the article
* A request example of enabling PGO for a tool with PGO support in the upstream
* 3rd party repos for PGO-optimized stuff for more boring projects like Fedora: https://discussion.fedoraproject.org/t/fedora-llvm-team-llvm-pgo-optimized/84361
* OS maintainers have more desire to enable PGO if an upstream project supports it: https://discord.com/channels/862292009423470592/1060577525929103510/1295008340652326934
* Mention Rust performance book in the article
* Add to the article: https://github.com/llvm/llvm-project/issues/57501
* Think about a talk about "LTO, PGO and PLO for Python native-based libs" for Python conferences. Also, the idea can be applicable for other langs like Ruby, etc.
* Write to Ferrocene support - https://github.com/ferrocene/ferrocene/issues/22 . Add to the article as an example that proprietary companies don't care much about public feedback mechanisms. If I won't get an answer before November 10 - write to office@ferrous-systems.com directly - update the article according to the answer from Ferrocene. I've a reply from them - talk about it in the article.
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
* Add PGO request to https://github.com/heroseh/hcc
* LLVM-based projects don't enable PGO even for already PGO-enabled things: https://github.com/pizlonator/llvm-project-deluge
* https://documentation.ubuntu.com/server/explanation/performance/perf-pgo/
* Run PGO benches here: https://github.com/nurmohammed840/nio/tree/main/benchmarks
* data-driven schedulers for preventing problems like this: https://github.com/sched-ext/scx/issues/376
* Manual inlining stuff here and there: https://github.com/FyroxEngine/Fyrox/issues/192#issuecomment-933926278
* Perform PGO benchmarks for Mistral: https://github.com/EricLBuehler/mistral.rs/tree/master/mistralrs-bench
* https://lwn.net/Articles/993828/ - Linux kernel with BOLT
* AutoFDO removes GCC support: https://www.mail-archive.com/gcc@gcc.gnu.org/msg102858.html
* https://github.com/orgs/tinygo-org/discussions/4643 - tinygo and PGO
* Link all PGO slides to awesome-pgo (people are asking for that at conferences)
* Sharing PGO profiles between distributions: https://www.phoronix.com/forums/forum/phoronix/latest-phoronix-articles/1509316-clang-autofdo-propeller-optimization-support-merged-for-linux-6-13?p=1509355#post1509355
* Add PGO request to https://github.com/orioledb/orioledb
* An example of optimizing software manually (https://github.com/rust-lang/rust-analyzer/issues/17491) instead of doing PGO: https://github.com/rust-lang/rust-analyzer/issues/9412
* Tell people more what PGO does in practice: https://github.com/tursodatabase/limbo/issues/78#issuecomment-2212339234
* Good optimize/overhead decision: https://github.com/DioxusLabs/dioxus/blob/main/Cargo.toml#L252
* Improvements from LTO were not huge: https://github.com/ricott1/rebels-in-the-sky/issues/24#issuecomment-2537146655
* Nikita Popov's talk about "Rust loves LLVM": https://www.youtube.com/watch?v=Kqz-umsAnk8
* https://github.com/floooh/fips/blob/3d05e74bc2f07a0b31138eed795e0a7d0368f753/CHANGELOG.md?plain=1#L194 - LTO enabled by default for VS projects in some build systems too
* People forget to enable a dedicated Release profile: https://github.com/Asurar0/mikomikagi/issues/2#issuecomment-2538506225
* An additional source of uncertainity possibly due to PGO: https://github.com/rust-lang/rustc-perf/issues/1592 that limits local reproduction (that's why PGO profiles should be public as well - write an idea about this)
* PGO for data structures could prevent this in theory but we not there yet: https://github.com/rust-lang/rustc_codegen_cranelift/commit/5d516f9e118d6527947ca5deb3d76bbc4fa0f8a1
* https://developers.redhat.com/articles/2023/11/07/how-i-experimented-pgo-enabled-llvm-fedora
* Propeller integration in CachyOS for Linux kernel: https://github.com/CachyOS/linux-cachyos/pull/359
* People are asking for PGO/PLO optimized binaries: https://www.reddit.com/r/cpp/comments/1hjhcwr/are_there_any_prebuilt_pgoboltoptimized_version/
* A person never used PGO (it's written in the article) - I am here to change it: https://deterministic.space/high-performance-rust.html
* Add another Ruby compiler: https://blog.llvm.org/posts/2024-12-03-minimalistic-ruby-compiler/ - AoT way is still important nowadays!
* PGO write profile and timeout: https://github.com/chimera-linux/cports/commit/81dd2a368e39993c2cb12d5be8ec06f1b7663908
* PGO network access can be a problem: https://github.com/chimera-linux/cports/blob/d8e3510901aca1c3843edaf7b3f33d62ed598a55/main/thunderbird/template.py#L171
* An example of how PGO could be enabled in OS package scripts: https://github.com/chimera-linux/cports/commit/3fdfa1a46ee61ca6fcb6d7a9573e49302309e247
* An example of less popular OS: https://aros.sourceforge.io/ where PGO won't be enabled soon (since PGO has pretty high entrance level right now)
* If documentation contains instructions about PGO usage it doesn't mean that PGO improves the project: https://github.com/vozlt/nginx-module-vts/pull/288#issuecomment-2567264150
* Learned (data-driven!) performance models for compilers: https://proceedings.mlsys.org/paper_files/paper/2021/file/6bcfac823d40046dca25ef6d6d59cc3f-Paper.pdf
* Add https://research.facebook.com/publications/vespa-static-profiling-for-binary-optimization/ to the article
* Add https://storage.googleapis.com/pub-tools-public-publication-data/pdf/578a590c3d797cd5d3fcd98f39657819997d9932.pdf to the article
* Add rustc: supports, but marked unstable: [commit](https://github.com/rust-lang/rust/commit/a17193dbb931ea0c8b66d82f640385bce8b4929a), [unstable book](https://doc.rust-lang.org/beta/unstable-book/compiler-flags/profile_sample_use.html) to the article
* Write about  -fdebug-info-for-profiling and -funique-internal-linkage-names: https://reviews.llvm.org/D25435 + https://reviews.llvm.org/D73307
* "Note the use of the -b flag. This tells Perf to use the Last Branch Record (LBR) to record call chains. While this is not strictly required, it provides better call information, which improves the accuracy of the profile data." - write that it's not clear how important the `-b` flag is in practice
* Add https://llvm.org/devmtg/2024-04/slides/TechnicalTalks/Xiao-EnablingHW-BasedPGO.pdf
* LLVM Profi: https://reviews.llvm.org/D109860?id=
* https://israelo.io/blog/pgo/ - add to the article
* https://theyahya.com/posts/go-pgo/ - add to the article
* Consumes too much RAM? Just add more swap :))) https://github.com/google/autofdo/issues/162#issuecomment-2627337220
* For some people almost 2x performance improvement is not worth it: https://github.com/crate-ci/typos/issues/827#issuecomment-2668988450
* Tooling can prevent enabling PGO: https://github.com/rust-cross/cargo-zigbuild/issues/315 + https://github.com/rust-lang/rust-analyzer/issues/9412#issuecomment-2802294024
* Not significant 20% perf but later the same improvement becomes significant: https://github.com/rust-lang/rust-analyzer/issues/9412#issuecomment-1298277389 + https://github.com/rust-lang/rust-analyzer/pull/19595#issuecomment-2807206942
* BOLT vs PGO example: https://github.com/rust-lang/rust/pull/139648
* https://github.com/WGUNDERWOOD/tex-fmt/issues/22#issuecomment-2401824584 - PGO stopped to help after some performance improvements in a project
* PGO won't be added to langs like MUMPS, heh: https://gitlab.com/Reference-Standard-M/rsm/-/blob/main/runtime/run.c?ref_type=heads
* Add PGO request to https://github.com/trynova/nova (and suggest enabling CG1 in an addition to LTO)
* Add PGO request to https://github.com/veryl-lang/veryl
* PGO slowly comes to compilers: https://github.com/llvm/llvm-project/pull/136098
* LLVM MinGW results for PGO: https://github.com/mstorsjo/llvm-mingw/pull/503#issue-3140215955
* Continuous instrumentation PGO in Clang: https://clang.llvm.org/docs/UsersManual.html#cmdoption-fprofile-continuous
* Use custom compiler versus a system one to speedup the builds: https://github.com/mstorsjo/llvm-mingw/pull/503#issue-3140215955
* PGO feature parity between LLVM-based compilers - start this discussion with Rustc devs
* Write about Deepwiki: https://deepwiki.com/zamazan4ik/awesome-pgo
* TursoDB another performance report: https://github.com/tursodatabase/turso/issues/78#issuecomment-3015109684
* Check PGO results on Wasmi: https://github.com/wasmi-labs/wasmi/pull/1449#issuecomment-2771943238
* PGO for Rustc on macOS - start the conversation again about enabling it
* https://github.com/zamazan4ik/awesome-pgo/blob/main/are_we_pgo_yet.md - this page often cannot be opened on GitHub since it's too large, lol
* Even giants and optimization nerds like Google have missing oppotunities for optimization with LTO/PGO for many years: https://issues.fuchsia.dev/issues/42120900?pli=1
* Bake - https://github.com/SanderMertens/bake?tab=readme-ov-file#installation - build system by default installs Debug version of the application...
* Report PGO results for Flecs later on GitHub to make them publicly-available (copy them from Discord)
* Flecs dev has an interesting idea about "domain-specific PGO" for collecting some runtime statistics for its ECS to improve performance - https://discord.com/channels/633826290415435777/715447777605320724/1389637330641485996 I was confused since he also used PGO abbreviation for it :)
* https://github.com/SanderMertens/bake by default is installed in a debug mode since it gives nicer stack traces
* GCC has its own Temporal PGO way before LLVM: https://gcc.gnu.org/onlinedocs/gcc-15.1.0/gcc/Optimize-Options.html#index-fprofile-reorder-functions + https://gcc.gnu.org/bugzilla/show_bug.cgi?id=120928#c2
* People don't need aging tickets to track PGO for some reason: https://github.com/neondatabase/neon/issues/4649#issuecomment-3083685375
* Memory-oriented PGO for TCMalloc: https://github.com/valexey/tcmalloc_hot_cold . Can we propose something similar for other allocators like mimalloc? Mimalloc: https://github.com/microsoft/mimalloc/issues/1129 What about jemalloc?
