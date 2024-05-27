# TODO

Here I collect random thoughts and ideas about further PGO investigation.

For now, the list is clear, and I am happy with that. But the article has nearly 200 TODOs...

* Think about using Rust's Crater (https://github.com/rust-lang/crater) for evaluating PGO over the Rust ecosystem "at scale"
* Write about my triggering on words like "performance" and similar
* Benefits from PGO to LTO in GCC too - https://gcc.gnu.org/legacy-ml/gcc-help/2019-05/msg00032.html
* BOLT's BAT: https://github.com/llvm/llvm-project/blob/main/bolt/docs/BAT.md + https://github.com/llvm/llvm-project/blob/main/bolt/docs/Heatmaps.md
* What about using PGO during fuzzing? To optimize the fuzzing efficiency but fuzzing explores all paths...
* Write about BOLT and many missing user guides, hidden options, etc.
* wrappers like https://github.com/KWARC/rust-libxml can struggle with enabling PGO due to its multi-lingua nature
* BOLT atomically updates its counters: https://discord.com/channels/636084430946959380/930647188944613406/1239768541729918976
* https://github.com/llvm/llvm-project/issues/63024#issuecomment-1751625579 - partial training mitigation
* https://llvm.org/docs/CommandGuide/llvm-profdata.html#cmdoption-llvm-profdata-merge-text - test this functionality and check the generated profile. Add to the article
* Why does this option exist if it isn't supported? https://llvm.org/docs/CommandGuide/llvm-profdata.html#cmdoption-llvm-profdata-merge-gcc . Asked the question: https://discourse.llvm.org/t/llvm-profdata-merge-and-gcc-emit-format-support/79207
* Write about missing Discussion in many repos on GitHub - that's why I created many Issues for PGO even if they are not the issues. Try to find a user request for enabling the Discussions by default on GitHub (or create such a request)
* PGO for Wuffs: https://github.com/google/wuffs (https://github.com/google/wuffs/blob/main/doc/benchmarks.md + https://github.com/google/wuffs/blob/main/doc/wuffs-the-language.md). Applying PGO for Wuffs' tooling like transpiler, formatters, etc.
* Bench PGO for https://github.com/HigherOrderCO/Bend
* PGO benchmark for vers
* Sometimes people aren't interested enough in PGO without benchmarks - https://github.com/oxc-project/oxc/issues/812#issuecomment-2116285554
* Write an answer to https://blog.cloudflare.com/reclaiming-cpu-for-free-with-pgo
* Add https://github.com/FStarLang/FStar to the missing PGO language list
* PGO benchmark for https://github.com/dalance/amber . Blocked by https://github.com/dalance/amber/issues/335
* Add resvg and Bend numbers to the article (also other numbers should be extracted)
* Strange responses about PGO - https://github.com/microsoft/qsharp/issues/752#issuecomment-1773194243
