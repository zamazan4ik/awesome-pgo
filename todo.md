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
* Enable LTO for benchmarks too (related mainly for Rust ecosystem)
