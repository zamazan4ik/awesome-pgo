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
* Write an answer to https://blog.cloudflare.com/reclaiming-cpu-for-free-with-pgo
* PGO benchmark for https://github.com/dalance/amber . Blocked by https://github.com/dalance/amber/issues/335
* Add resvg and Bend numbers to the article (also other numbers should be extracted)
* Strange responses about PGO - https://github.com/microsoft/qsharp/issues/752#issuecomment-1773194243
* Compiler statistics of PGO: https://github.com/oxc-project/oxc/issues/812#issuecomment-2134080943
* Search over compiler options: https://github.com/google/heir/discussions/613#discussioncomment-9606880
* PGO benches for https://github.com/sarah-ek/faer-rs
* Write about a stupid idea about waiting for the loooong benchmarks during the training phase (ClickHouse and faer-rs cases). Add a warning for using benchmarks as a PGO training workload. PGO instrumentation phase for faer-rs was finished in 40 hours (!!!)
* An error with BOLTing a project: https://discord.com/channels/636084430946959380/652264942778449934/1247133234719100929
* Difference between Rustc and Clang in reordering: https://users.rust-lang.org/t/differences-in-an-optimization-decision-between-rustc-and-clang/112593
* Add PGO request to https://github.com/davidlattimore/wild
* Add a chapter about starting LTO/PGO/PLO journey
* HALO: https://dl.acm.org/doi/pdf/10.1145/3368826.3377914
* PGO issues with cross-compilation: https://github.com/LadybirdWebBrowser/ladybird/commit/3ff7b76502ff2b7492660475a881befe2cda42f9
* PGO is not enabled for an LLVM fork: https://github.com/Xilinx/llvm-aie
* Once again - closing a PGO issue without interest at all: https://github.com/SerenityOS/serenity/issues/19549
* CERN's Root closed the PGO issue: https://github.com/root-project/root/issues/15778#issuecomment-2154453640
* SerenityOS (and Ladybird) policy, 8th one - "Talk is cheap. Don't waste other people's time by talking about your great ideas if you don't also spend time implementing your ideas. We have no need for "idea guys"". LOL, how do I need to propose an idea?
