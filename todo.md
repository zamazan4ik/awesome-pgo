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
* Found a huge slowdown from BOLT instrumentation compared to PGO instrumentation in the Pylyzer case
* Extract useful information about PGO in LDC and add to the article: https://johanengelen.github.io/ldc/2016/04/13/PGO-in-LDC-virtual-calls.html
* Add to the article info about link errors and passing PGO flags to LD too
* Some people actually read the article! https://github.com/asterinas/asterinas/issues/760#issuecomment-2060277998
* Add to the article information about PGO profile file formats. Discuss at least LLVM-compatible format. Sampling file format is described here - https://clang.llvm.org/docs/UsersManual.html#sample-profile-formats . But not instrumentation formats - is it possible to find it? Is it required since we have documented sampling format?
* Ask Android devs about collection PGO profiles from an emulator. Now it says "simpleperf E event_type.cpp:508] Unknown event_type 'cs-etm', try `simpleperf list` to list all possible event type names" so it doesn't work with an emulator. From my Pixel 8 I also cannot gather profiles due to the "adbd cannot be runned in the production builds" adb error
* Interesting discussions about multiple PGO profiles, memory degradation and performance boost due to better PGO training set: https://issues.chromium.org/issues/41490637
* PGO and experiments (aka Finch) in Chromium: https://issues.chromium.org/issues/326465539 + https://news.ycombinator.com/item?id=12954588 (explains what Finch is)
* ThinLTO, thread contention and possible mitigations for that: https://issues.chromium.org/issues/40942246#comment4
* Local PGO profile generation for Android questions: https://issues.chromium.org/issues/40286433 + https://issues.chromium.org/issues/40287513
* Some improvements around PGO generation in CrOS (ChromeOS): https://issuetracker.google.com/issues/298247335
* ChromeOS uses AFDO for the kernel: https://issuetracker.google.com/issues/245629081 + https://issuetracker.google.com/issues/327058411
* Tweaking LTO constants: https://issues.chromium.org/issues/40219076
