# TODO

Here I collect random thoughts and ideas about further PGO investigation.

For now, the list is clear, and I am happy with that. But the article has nearly 100 TODOs...

* What about using PGO during fuzzing? To optimize the fuzzing efficiency but fuzzing explores all paths...
* <https://llvm.org/docs/CommandGuide/llvm-profdata.html#cmdoption-llvm-profdata-merge-text> - test this functionality and check the generated profile. Add to the article
* PGO benchmark for vers
* Sometimes people aren't interested enough in PGO without benchmarks - <https://github.com/oxc-project/oxc/issues/812#issuecomment-2116285554>
* Compiler statistics of PGO: <https://github.com/oxc-project/oxc/issues/812#issuecomment-2134080943>
* PGO benches for https://github.com/sarah-ek/faer-rs
* Add a chapter about starting LTO/PGO/PLO journey
* Make PGO benchmark for <https://github.com/apache/arrow-rs> - benchmarks are done, they are saved on a local machine. Need to be published to the upstream
* Enable LTO for benchmarks too (related mainly for the Rust ecosystem) - discuss it in the article
* PGO documentation for libraries: https://github.com/Frostie314159/ieee80211-rs?tab=readme-ov-file#optimization
* Perform PGO benchmarks for https://github.com/clarkmcc/cel-rust
* Perform PGO benchmarks for https://github.com/mwlon/pcodec/blob/main/pco_cli/README.md#bench
* Question about PGO was being moderated for months in OVH Cloud
* Propose PGO to https://git.deuxfleurs.fr/Deuxfleurs/garage
* Propose LTO and PGO for https://pnut.sh/
* Perform PGO benchmarks on https://github.com/pydantic/jiter - https://github.com/pydantic/jiter/issues/123
* Of course, I found ICE in Clang 18 during PGOing Redpanda :D - https://github.com/llvm/llvm-project/issues/101902
* Check Rust no_std programs - how do they work with PGO? https://github.com/Amanieu/minicov - here can be some useful info about the topic
* Suggest PGO to https://github.com/phiresky/sqlite-zstd
* Another example of software not benefitting a lot from PGO - https://www.reddit.com/r/cpp/comments/17zn0e3/comment/ka0cl2x/
* SlateDB build time (db_bench): 25s without LTO vs 1 min with LTO (lto = true)
* Write about next step in PGO advertisment - Go ecosystem and why exactly this PL
* Estimating PGO efficiency without benchmarks - no such tools but some ideas like metrics, ML, questionaries like "Are you Fabris Bellard?" )
* https://app.radicle.xyz/nodes/seed.radicle.xyz/rad:z3gqcJUoA1n9HaHKufZs5FCSGazv5 - I don't know how to create an issue for the project
* Check LTO and PGO for https://github.com/spiraldb/vortex
* Check PGO state in https://www.illumos.org/ . Maybe create one more "PGO all the packages" issue at their forum/bug tracker
* https://github.com/ferrumc-rs/ferrumc/issues/15#issuecomment-2348951292 - another comment that don't understand PGO's value for applications performance in general
* Add PGO integration example - SVT-AV1: https://gitlab.com/AOMediaCodec/SVT-AV1/-/blob/master/CMakeLists.txt?ref_type=heads#L262
* Check PGO for https://github.com/adam-mcdaniel/dune/commit/c056b4c08782050262abc5c34e4391f2d387507e + suggest LTO
* Write a dedicated "Call for action" about using LTO for binaries: share thoughts, performance numbers, plans, suggestions, etc. Because of I'm tired with creating issues like "Enable LTO" on GitHub
* Some Cargo commands already enabled LTO: https://github.com/kbknapp/cargo-outdated/blob/master/Cargo.toml#L48
* Cargo extensions don't usually enable LTO for some reason. One of them - increased "installation" time for users since they compile extensions locally on their machines
* LTO increased build time for Buck: https://github.com/facebook/buck2/blob/main/Cargo.toml#L416 - 50s to 84s with ThinLTO
* Add PGO request to https://sourceforge.net/projects/sdcc/ . Build the compiler with PGO and implement PGO for the compiler
