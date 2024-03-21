# TODO

Here I collect random thoughts and ideas about further PGO investigation.

* Add more info about LTO and PGO state for packages in different Linux distros
* Get more information about PGO states in other browsers like Safari, Brave ([GitHub comment](https://github.com/brave/brave-browser/issues/20560#issuecomment-1658782341)), Yandex.Browser and others
* Write more about PGO and library development
* Add an issue to LLVM upstream about writing a difference between PGO implementations (Frontend vs IR vs Context-Sensitive), pros and cons of each, which to use by default. Probably add a link to the blog post of a person who described all these stuff.
* Contact with more proprietary vendors regarding PGO support in their compilers and PGO optimization their compilers itself before distribution to the customers. Possible compilers list: https://en.wikipedia.org/wiki/List_of_compilers
* Points of the software supply chain, where PGO can be applied (we can influence any of them with different methods and priorities):
  - Software itself (add PGO to the build scripts)
  - Maintainers (work with different distributions about enabling PGO for the packages)
  - In-house builds (if we build binary somewhere in a closed perimeter)
* Extract actual numbers directly into the document for avoiding the cases like [this](https://github.com/facebook/mariana-trench/issues/137#issuecomment-1658195725).
* Describe different PGO application scenarios: for SaaS, for open-source, for closed-source but delivered to the customers, etc.
* Discussion BOLT and Propeller: https://discourse.llvm.org/t/practical-compiler-optimizations-for-wsc-workshop-us-llvm-dev-meeting-2023/73998/
* Play locally with Propeller: https://github.com/google/llvm-propeller and discuss integrating it into the cargo-pgo
* Do not forget to add a mention about monitoring actual workload and tracking - does it match your current PGO-profile or not
* Example of an application where PGO exists but is not used for prebuilt binaries: https://github.com/ispc/ispc/issues/2687
* Write about Clang (LLVM) vs GCC PGO implementations: GCC has more PGO switches, Clang has more recently-researched PGO-related things inside, etc.
* Add information about PGO missed optimizations like GCC (https://gcc.gnu.org/bugzilla/buglist.cgi?quicksearch=PGO&list_id=403385), Clang (TODO), Go compiler (TODO), Rustc (TODO)
* Add CachyOS project as an example where BOLT is integrated at most to the OS build pipelines
* An interesting case of a try to enable PGO for Foot in Void Linux: https://github.com/void-linux/void-packages/issues/29526
* Create a generic PGO + BOLT issue in Gentoo (links reporting in the bugzilla restricted until 15.11.2023)
* Check https://hpsfoundation.github.io/ and its projects. Probably they will be interested in PGO as well since it's HPC foundation
* Become a Mageia maintainer to push PGO/PLO activity: https://bugs.mageia.org/show_bug.cgi?id=32511
* Clear Linux PGO packages: https://github.com/search?q=org%3Aclearlinux-pkgs+%22pgo+%3D+true%22&type=code + PGO integration as a USE flag
* Write about an observation that if in the upstream there is no PGO support - maintainers almost always do not enable PGO in their packages
* Intel compiler has support for Sampling PGO too even outside `perf` - https://www.intel.com/content/www/us/en/docs/dpcpp-cpp-compiler/developer-guide-reference/2024-0/hardware-profile-guided-optimization.html. Add to the article.
* Check https://www.youtube.com/watch?v=GBtQrYx_Jbc talk about PGO
* Add a link about some issues with the current build infrastructure for PGO in Rust: https://github.com/yamafaktory/jql/discussions/244#discussioncomment-7665597
* Add to the article: https://discourse.llvm.org/t/couple-of-general-questions-about-pgo/72279. Here there are some interesting insights about PGO state in the compiler. https://discourse.llvm.org/t/status-of-ir-vs-frontend-pgo-fprofile-generate-vs-fprofile-instr-generate/58323 thread also has many valuable insights about FE PGO vs IR PGO.
* PGO questions on LLVM forum - https://discourse.llvm.org/t/profile-guided-optimization-pgo-related-questions-and-suggestions/75232
* Find information about single and multi-threaded PGO counters - in Clang its https://clang.llvm.org/docs/UsersManual.html#cmdoption-fprofile-update . Related GCC bug about PGO profiles slowness: https://gcc.gnu.org/bugzilla/show_bug.cgi?id=89307 . GCC docs for it: https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html#index-fprofile-update . Problems in practice with such an approach: https://reviews.llvm.org/D34085 . More discussion in https://lists.llvm.org/pipermail/llvm-dev/2014-April/072172.html
* Write a note about LLVM static counters warning and possible mitigations: https://bugs.fuchsia.dev/p/fuchsia/issues/detail?id=111987
* PGO for Android games: https://developer.android.com/games/agde/pgo-overview and GDC talk (https://www.youtube.com/watch?v=CNbpFTyHOe8)
  - Contact the author and ask about more advanced PGO techniques: CSIR PGO, Sampling PGO, maybe BOLT/Propeller
* Documentation issues about PGO even from Google: https://issuetracker.google.com/issues/315487999
* Write about multiple instrumentation and sampling tooling and their compatibility with PGO infrastructure: https://thume.ca/2023/12/02/tracing-methods/
* Apple question about PGO:
  - Apple StackExchange: https://apple.stackexchange.com/questions/467215/profile-guided-optimization-pgo-in-xcode , https://apple.stackexchange.com/questions/467182/sampling-profile-guided-optimization-pgo-with-xcode-and-clang , https://apple.stackexchange.com/questions/467218/profile-guided-optimization-pgo-differences-between-clang-and-apple-clang
  - Apple developer forum: https://developer.apple.com/forums/thread/742840
* Fix Android PGO docs: https://source.android.com/docs/core/perf/pgo - https://issuetracker.google.com/issues/315464624
* GraalVM's LLVM toolchain and PGO optimization: https://github.com/oracle/graal/discussions/7989
* Ask GraalVM about dumping PGO profile to a memory region and other PGO-related runtime intrinsics: https://github.com/oracle/graal/discussions/7991
* GraalVM PGO profile optimization strategy for non-executed code: https://github.com/oracle/graal/discussions/7999
* Arch feature levels as an optimization idea: https://www.phoronix.com/news/Ubuntu-x86-64-v3-Experiment
* AOSP switches to AFDO: https://issuetracker.google.com/issues/315464624
* Write a note about reporting problems with PGO to the upstream - how to do it properly? Especially, if we are talking about debuggability of the performance regression
* Add a note to the article about debugging performance regressions with PGO - how you can do it
* Talk with SerpentOS maintainers about enabling PGO for their packages: https://serpentos.com/
* Write about a failed idea of extending PGO support across the ecosystem and GSoC (was discussed at FOSDEM 2024 with Google representatives - it's not so easy to do via GSoC as I thought before)
* Discuss the PGO state across operating systems. Discuss about PGO-edge experience in distributions like ClearLinux, CachyOS, Gentoo
* Do I need to prepare more materials about PGO in C#?
* Great paper about different sampling FDO approaches: https://hpctest.cs.tsinghua.edu.cn/papers/tc11.pdf
* Check information about LIPO (lightweight IPO): https://static.googleusercontent.com/media/research.google.com/ru//pubs/archive/36355.pdf What is the current state of the LIPO in the compilers? ThinLTO is not the same thing. However, here I see some that ThinLTO somehow supports profile data: https://llvm.org/devmtg/2015-04/slides/ThinLTO_EuroLLVM2015.pdf . My question is here: https://discourse.llvm.org/t/thinlto-and-profile-based-module-groups/77120
* Check https://storage.googleapis.com/gweb-research2023-media/pubtools/pdf/36358.pdf paper for LBR efficiency in Sampling PGO in practice
* Write about how to report performance degradation from PGO and why it could be difficult to do
* Write about llvm-profdata version compatibility - it's important otherwise you will get errors
* Check ThinLTO with PGO results: https://ieeexplore.ieee.org/document/7863733?reload=true
* Write some statistics about how projected reacted to PGO reports to them
* Write about "missing profile" errors
