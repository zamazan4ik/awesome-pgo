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
* Discuss a question about UI applications and PGO optimization during the build phase: https://github.com/slint-ui/slint/issues/3909#issuecomment-1816054164
* Found a huge slowdown from BOLT instrumentation compared to PGO instrumentation in the Pylyzer case
* Library documentation about PGO can look like this: https://github.com/lovasoa/serde-sqlite-jsonb/pull/2
* Extract useful information about PGO in LDC and add to the article: https://johanengelen.github.io/ldc/2016/04/13/PGO-in-LDC-virtual-calls.html
* Add to the article info about link errors and passing PGO flags to LD too
* Some people read article! https://github.com/asterinas/asterinas/issues/760#issuecomment-2060277998
* Rework article a bit since Intel SEP can be used for Sampling PGO on Linux, Windows, FreeBSD platforms - https://clang.llvm.org/docs/UsersManual.html#using-sampling-profilers . But latest version of this tool doesn't work at least on Linux kernel 6.x - https://community.intel.com/t5/Analyzers/socwatch2-15-module-is-not-loaded-correctly-when-using-kernel-6/td-p/1584234
* Discuss it in the article: https://github.com/zamazan4ik/awesome-pgo/issues/3#issuecomment-2065582161
* Add to the article information about PGO profile file formats. Discuss at least LLVM-compatible format. Sampling file format is described here - https://clang.llvm.org/docs/UsersManual.html#sample-profile-formats . But not instrumentation formats
* Android switches to use AFDO instead of PGO: https://source.android.com/docs/core/perf/autofdo according to https://issuetracker.google.com/issues/315464624#comment3 . Check more on this topic
* Sometimes PGO is disabled: https://github.com/llvm/llvm-project/commit/e6c3289804a67ea0bb6a86fadbe454dd93b8d855
* Ask Android devs about collection PGO profiles from an emulator. Now it says "simpleperf E event_type.cpp:508] Unknown event_type 'cs-etm', try `simpleperf list` to list all possible event type names" so it doesn't work with an emulator. From my Pixel 8 I also cannot gather profiles due to the "adbd cannot be runned in the production builds" adb error
* Temporal PGO: https://issues.chromium.org/issues/326463850 + https://groups.google.com/a/chromium.org/g/chromium-dev/c/Awq_xBXQgFY/m/8N2L2W2vAQAJ + https://discourse.llvm.org/t/rfc-temporal-profiling-extension-for-irpgo/68068
* What about talking about PGO at mobile-development-oriented conferences? Need to be considered
* libvpx performance with PGO: https://issues.chromium.org/issues/325103518#comment8
* Measured PGO degradation due to failed benchmarks: https://issues.chromium.org/issues/41491803#comment17
* Interesting discussions about multiple PGO profiles, memory degradation and performance boost due to better PGO training set: https://issues.chromium.org/issues/41490637
* PGO training workloads can be shared between companies and can be quite private: https://issues.chromium.org/issues/325519354 + https://chromium.googlesource.com/chromium/src.git/+/master/docs/pgo.md
* PGO and experiments (aka Finch) in Chromium: https://issues.chromium.org/issues/326465539 + https://news.ycombinator.com/item?id=12954588 (explains what Finch is)
* Long discussion about enabling PGO with Clang for the Linux kernel: https://groups.google.com/g/clang-built-linux/c/85eKdM9_Jeg/m/cX7-6Kk7AQAJ
* Request for enabling PGO for the Rustc compiler in Fuchsia: https://issues.fuchsia.dev/issues/42083760
* Enable PGO, LTO for Fuchsia: https://issues.fuchsia.dev/issues/335919004 + https://issues.fuchsia.dev/issues/331995728
* LLVM PGO hash mismatch and the fix for it: https://issues.fuchsia.dev/issues/336358027
* Another PGO abbreviation: https://en.m.wikipedia.org/wiki/PGO_waves
