# TODO

Here I collect random thoughts and ideas about further PGO investigation.

For now list is clear, and I am happy with that. But the article has near 200 TODOs...

* Think about using Rust's Crater (https://github.com/rust-lang/crater) for evaluating PGO over the Rust ecosystem "at scale"
* Write about my triggering on words like "performance" and similar
* Benefits from PGO to LTO in GCC too - https://gcc.gnu.org/legacy-ml/gcc-help/2019-05/msg00032.html
* BOLT's BAT: https://github.com/llvm/llvm-project/blob/main/bolt/docs/BAT.md + https://github.com/llvm/llvm-project/blob/main/bolt/docs/Heatmaps.md
* What about using PGO during fuzzing? For optimizing the fuzzing efficiency
* Write about BOLT and many missing user guides, hidden options, etc.
* wrappers like https://github.com/KWARC/rust-libxml can struggle with enabling PGO due to its multi-lingua nature
* BOLT atomically updates its counters: https://discord.com/channels/636084430946959380/930647188944613406/1239768541729918976
