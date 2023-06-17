## MongoDB PGO

Current results:
* `mongod` from the `master` branch and `r6.3.1` segfaults on the start if built in instrumentation mode (with `-fprofile-generate` compiler flag)
* `mongod` does not compile with Clang 12 and corresponding `libc++` in Fedora 38 due to multiple compilation errors on the current `master` branch
* `mongod` does not compile with Clang 16 and corresponding `libc++` in Fedora 38 due to multiple compilation errors on the current `master` branch
* `mongod` does not compile with Clang 12 and 16 and `libstdc++` from Fedora 38 due to multiple compilation errors on the current `master` branch
* `mongod` does not compile with Clang 16 and `libstdc++` from Fedora 38 due to multiple compilation errors on the current `master` branch
* MongoDB on the `master` branch does not correctly detect GCC 13 supports `-flto` flag with enabled `--lto` build option

Seems like `r6.3.1` MongoDB tag builds fine with Clang 16. The tests will continue...
