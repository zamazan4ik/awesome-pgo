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
* Suggest PGO for https://github.com/phiresky/ripgrep-all/blob/master/Cargo.toml
* Golang Conf 2024, a talk about PGO in Go - https://www.youtube.com/watch?v=FyZJlPMBFm8 , talk to the speaker about their PGO way in Go
* Rework wording in https://github.com/Kobzol/cargo-pgo/issues/60 - add to the article about influence of such "wrong" docs
* CachyOS optimizes more packages with PGO: https://www.phoronix.com/news/CachyOS-September-2024 - mention Phoronix too in the article
* Add about MLGO for regalloc: https://groups.google.com/g/llvm-dev/c/7jxuvu3WPl0 + https://github.com/google/ml-compiler-opt/blob/main/docs/regalloc-demo/demo.md + https://www.phoronix.com/forums/forum/phoronix/latest-phoronix-articles/1494741-cachyos-optimizing-more-packages-with-pgo-for-up-to-~10-better-performance?p=1494782#post1494782 . In the article already there is some information about MLGO - extend it
