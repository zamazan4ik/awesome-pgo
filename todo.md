# TODO

Here I collect random thoughts and ideas about further PGO investigation.

* Add more info about LTO and PGO state for packages in different Linux distros
* Get more information about PGO states in other browsers like Safari, Brave ([GitHub comment](https://github.com/brave/brave-browser/issues/20560#issuecomment-1658782341)), Yandex.Browser and others
* Points of the software supply chain, where PGO can be applied (we can influence any of them with different methods and priorities):
  - Software itself (add PGO to the build scripts)
  - Maintainers (work with different distributions about enabling PGO for the packages)
  - In-house builds (if we build binary somewhere in a closed perimeter)
* Extract actual numbers directly into the document to avoid cases like [this](https://github.com/facebook/mariana-trench/issues/137#issuecomment-1658195725).
* Play locally with Propeller: https://github.com/google/llvm-propeller and discuss integrating it into the cargo-pgo
* Do not forget to add a mention about monitoring actual workload and tracking - does it match your current PGO-profile or not
* Add information about PGO missed optimizations like GCC (https://gcc.gnu.org/bugzilla/buglist.cgi?quicksearch=PGO&list_id=403385), Clang (TODO), Go compiler (TODO), Rustc (TODO)
* Clear Linux PGO packages: https://github.com/search?q=org%3Aclearlinux-pkgs+%22pgo+%3D+true%22&type=code + PGO integration as a USE flag
* Find information about single and multi-threaded PGO counters - in Clang it's https://clang.llvm.org/docs/UsersManual.html#cmdoption-fprofile-update . Related GCC bug about PGO profiles slowness: https://gcc.gnu.org/bugzilla/show_bug.cgi?id=89307 . GCC docs for it: https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html#index-fprofile-update . Problems in practice with such an approach: https://reviews.llvm.org/D34085 . More discussion in https://lists.llvm.org/pipermail/llvm-dev/2014-April/072172.html
* Write about multiple instrumentation and sampling tooling and their compatibility with PGO infrastructure: https://thume.ca/2023/12/02/tracing-methods/
* Fix Android PGO docs: https://source.android.com/docs/core/perf/pgo - https://issuetracker.google.com/issues/315464624
* AOSP switches to AFDO: https://issuetracker.google.com/issues/315464624
* Write a note about reporting problems with PGO to the upstream - how to do it properly? Especially, if we are talking about debuggability of the performance regression
* Add a note to the article about debugging performance regressions with PGO - how you can do it
* Talk with SerpentOS maintainers about enabling PGO for their packages: https://serpentos.com/
* Write about a failed idea of extending PGO support across the ecosystem and GSoC (was discussed at FOSDEM 2024 with Google representatives - it's not so easy to do via GSoC as I thought before)
* Great paper about different sampling FDO approaches: https://hpctest.cs.tsinghua.edu.cn/papers/tc11.pdf
* Check information about LIPO (lightweight IPO): https://static.googleusercontent.com/media/research.google.com/ru//pubs/archive/36355.pdf What is the current state of the LIPO in the compilers? ThinLTO is not the same thing.
* Check https://storage.googleapis.com/gweb-research2023-media/pubtools/pdf/36358.pdf paper for LBR efficiency in Sampling PGO in practice
* Write about how to report performance degradation from PGO and why it could be difficult to do
* Write about llvm-profdata version compatibility - it's important otherwise you will get errors
* Add more benchmark numbers when a PGO training workload and target workload are not completely the same
* Evaluate PGO on https://github.com/guillaume-be/rust-bert and https://github.com/guillaume-be/rust-tokenizers
* Discuss a question about UI applications and PGO optimization during the build phase: https://github.com/slint-ui/slint/issues/3909#issuecomment-1816054164
* Check https://www.phoronix.com/news/Intel-Thin-Layout-Optimizer tool - like LLVM BOLT but from Intel. Highly recommend reading this: https://github.com/intel/thin-layout-optimizer/wiki . Add to the article
* HighTec compilers like https://hightec-rt.com/en/rust - are they PGO-optimized? Started a conversation with a HighTec dev on Reddit
* Found a huge slowdown from BOLT instrumentation compared to PGO instrumentation in the Pylyzer case
* Library documentation about PGO can look like this: https://github.com/lovasoa/serde-sqlite-jsonb/pull/2
* AutoFDO support was added in GCC 5 and LLVM 3.5 - https://hubicka.blogspot.com/2015/04/GCC5-IPA-LTO-news.html
* Extract useful information about PGO in LDC and add to the article: https://johanengelen.github.io/ldc/2016/04/13/PGO-in-LDC-virtual-calls.html
* FOSDEM 2017 talk about PGO in LDC: https://av.tib.eu/media/42272
* Add to the article info about link errors and passing PGO flags to LD too
