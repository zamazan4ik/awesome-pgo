# TODO

Here I collect random thoughts and ideas about further PGO investigation.

For now, the list is clear, and I am happy with that. But the article has nearly 100 TODOs...

* What about using PGO during fuzzing? To optimize the fuzzing efficiency but fuzzing explores all paths...
* <https://llvm.org/docs/CommandGuide/llvm-profdata.html#cmdoption-llvm-profdata-merge-text> - test this functionality and check the generated profile. Add to the article
* PGO benchmark for vers
* Sometimes people aren't interested enough in PGO without benchmarks - <https://github.com/oxc-project/oxc/issues/812#issuecomment-2116285554>
* Compiler statistics of PGO: <https://github.com/oxc-project/oxc/issues/812#issuecomment-2134080943>
* PGO benches for <https://github.com/sarah-ek/faer-rs>
* Add a chapter about starting LTO/PGO/PLO journey
* Make PGO benchmark for <https://github.com/apache/arrow-rs> - benchmarks are done, they are saved on a local machine. Need to be published to the upstream
* Enable LTO for benchmarks too (related mainly for the Rust ecosystem)
* PGO documentation for libraries: https://github.com/Frostie314159/ieee80211-rs?tab=readme-ov-file#optimization
* Perform PGO benchmarks for https://github.com/clarkmcc/cel-rust
* Perform PGO benchmarks for https://github.com/mwlon/pcodec/blob/main/pco_cli/README.md#bench
* Question about PGO was being moderated for months in OVH Cloud
* Propose PGO to https://git.deuxfleurs.fr/Deuxfleurs/garage
* Propose LTO and PGO for https://pnut.sh/
* Perform PGO benchmarks on https://github.com/pydantic/jiter - https://github.com/pydantic/jiter/issues/123
* Of course, I found ICE in Clang 18 during PGOing Redpanda :D - https://github.com/llvm/llvm-project/issues/101902
