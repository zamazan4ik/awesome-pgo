# TODO

Here I collect random thoughts and ideas about further PGO investigation.

For now, the list is clear, and I am happy with that. But the article has nearly 200 TODOs...

* Write about my triggering on words like "performance" and similar
* BOLT's BAT: https://github.com/llvm/llvm-project/blob/main/bolt/docs/BAT.md + https://github.com/llvm/llvm-project/blob/main/bolt/docs/Heatmaps.md
* What about using PGO during fuzzing? To optimize the fuzzing efficiency but fuzzing explores all paths...
* Write about BOLT and many missing user guides, hidden options, etc.
* wrappers like https://github.com/KWARC/rust-libxml can struggle with enabling PGO due to its multi-lingua nature
* BOLT atomically updates its counters: https://discord.com/channels/636084430946959380/930647188944613406/1239768541729918976
* https://github.com/llvm/llvm-project/issues/63024#issuecomment-1751625579 - partial training mitigation
* https://llvm.org/docs/CommandGuide/llvm-profdata.html#cmdoption-llvm-profdata-merge-text - test this functionality and check the generated profile. Add to the article
* Write about missing Discussion in many repos on GitHub - that's why I created many Issues for PGO even if they are not the issues. Try to find a user request for enabling the Discussions by default on GitHub (or create such a request)
* PGO for Wuffs: https://github.com/google/wuffs (https://github.com/google/wuffs/blob/main/doc/benchmarks.md + https://github.com/google/wuffs/blob/main/doc/wuffs-the-language.md). Applying PGO for Wuffs' tooling like transpiler, formatters, etc.
* Bench PGO for https://github.com/HigherOrderCO/Bend
* PGO benchmark for vers
* Sometimes people aren't interested enough in PGO without benchmarks - https://github.com/oxc-project/oxc/issues/812#issuecomment-2116285554
* PGO benchmark for https://github.com/dalance/amber . Blocked by https://github.com/dalance/amber/issues/335
* Add resvg and Bend numbers to the article (also other numbers should be extracted)
* Compiler statistics of PGO: https://github.com/oxc-project/oxc/issues/812#issuecomment-2134080943
* Search over compiler options: https://github.com/google/heir/discussions/613#discussioncomment-9606880
* PGO benches for https://github.com/sarah-ek/faer-rs
* Write about a stupid idea about waiting for the loooong benchmarks during the training phase (ClickHouse and faer-rs cases). Add a warning for using benchmarks as a PGO training workload. PGO instrumentation phase for faer-rs was finished in 40 hours (!!!)
* An error with BOLTing a project: https://discord.com/channels/636084430946959380/652264942778449934/1247133234719100929
* Difference between Rustc and Clang in reordering: https://users.rust-lang.org/t/differences-in-an-optimization-decision-between-rustc-and-clang/112593
* Add PGO request to https://github.com/davidlattimore/wild
* Add a chapter about starting LTO/PGO/PLO journey
* PGO issues with cross-compilation: https://github.com/LadybirdWebBrowser/ladybird/commit/3ff7b76502ff2b7492660475a881befe2cda42f9
* PGO is not enabled for an LLVM fork: https://github.com/Xilinx/llvm-aie
* Once again - closing a PGO issue without interest at all: https://github.com/SerenityOS/serenity/issues/19549
* Add https://github.com/chroma-core/chroma/issues/2296#issuecomment-2176556475 comment as proof of my work usefulness
* Make PGO benchmark for https://github.com/apache/arrow-rs - benchmarks are done, they are saved on a local machine. Need to be published to the upstream
* Perform PGO benchmarks for https://github.com/unicode-org/icu4x . Blocked by https://github.com/unicode-org/icu4x/issues/5105
* Suggest LTO + PGO for https://github.com/verus-lang/verus
* Perform PGO benchmarks for Sequoia (also check for LTO - they are not against it, open an issue for that). It has link errors during the instrumentation phase (as I met for Tach project). Need to fugure out the reason
* Make PGO benchmarks (or at least a request) for https://github.com/prisma/prisma-engines
* Make PGO benches for https://github.com/udoprog/musli
* Make PGO request for https://github.com/OpenRadioss/OpenRadioss
* Lady Dreidra's documentation about PGO - https://github.com/Eliah-Lakhin/lady-deirdre/commit/63b54ec04b2bac26812f066737f807f6a5b21ebf - is not visible for users of the library. It should be placed in a more visible place like the official documentation (the place which users read) - https://lady-deirdre.lakhin.com/
* Add to the article a comment from RustFest about PGO profile collection time, possible strategies for reduction it and hot/cold ratio for different PGO traing phase durations
* Suggest using LTO + PGO for https://github.com/pendulum-project/ntpd-rs/blob/main/Cargo.toml
* https://github.com/ojdkbuild/tools_toolchain_vs2017bt_1416/blob/master/VC/Tools/MSVC/14.16.27023/crt/src/vcruntime/exe_common.inl#L204 - what the hell is going on with PGO instrumentation on Windows
