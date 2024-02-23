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
* How to quickly check PGO support in the project: search over issues for "PGO", "Profile guided", "FDO". Also works grepping over the project for the same words or for PGO-related compiler flags like `-fprofile-generate`/`-fprofile-use`, etc.
* Extract actual numbers directly into the document for avoiding the cases like [this](https://github.com/facebook/mariana-trench/issues/137#issuecomment-1658195725).
* Describe different PGO application scenarios: for SaaS, for open-source, for closed-source but delivered to the customers, etc.
* Not everywhere PGO is available (e.g. WASM): https://github.com/rust-lang/rust/issues/81684
* Committing PGO profiles into a project - transparency problem if you do not describe how this profile is collected
* Check https://trofi.github.io/posts/243-gcc-profiler-internals.html article about GCC profiler internals
* Discussion BOLT and Propeller: https://discourse.llvm.org/t/practical-compiler-optimizations-for-wsc-workshop-us-llvm-dev-meeting-2023/73998/
* What is https://prodfiler.com/ ? Check their work and probably write about them in the article
* Play locally with Propeller: https://github.com/google/llvm-propeller and discuss integrating it into the cargo-pgo
* Do not forget to mention about monitoring actual workload and tracking - does it match your current PGO-profile or not
* Example of an application where PGO exists but not used for prebuilt binaries: https://github.com/ispc/ispc/issues/2687
* Write about Clang (LLVM) vs GCC PGO implementations: GCC has more PGO switches, Clang has more recently-researched PGO-related things inside, etc.
* Prepare an answer in Angie (Nginx fork) repository about PGO
* Possible LTO issues in compilers: https://github.com/rust-lang/rust/pull/113433
* Good tracking issue for the Rustc optimizations: https://github.com/rust-lang/rust/issues/103595
* PGO-related issue in Rustc: https://github.com/tensorchord/pgvecto.rs/issues/43#issuecomment-1788324707
* Check Android (and Android-dependent) project for PGO/BOLT usage
* Add information about PGO missed optimizations like GCC (https://gcc.gnu.org/bugzilla/buglist.cgi?quicksearch=PGO&list_id=403385), Clang (TODO), Go compiler (TODO), Rustc (TODO)
* Add CachyOS project as an example where BOLT is integrated at most to the OS build pipelines
* An interesting case of a try to enable PGO for Foot in Void Linux: https://github.com/void-linux/void-packages/issues/29526
* Create a generic PGO + BOLT issue in Gentoo (links reporting in the bugzilla restricted until 15.11.2023)
* Check https://hpsfoundation.github.io/ and its projects. Probably they will be interested in PGO as well since it's HPC foundation
* Become a Mageia maintainer to push PGO/PLO activity: https://bugs.mageia.org/show_bug.cgi?id=32511
* Check Firefox and Chromium derivatives like Thunderbird and Tor browser for PGO
* Clear Linux PGO packages: https://github.com/search?q=org%3Aclearlinux-pkgs+%22pgo+%3D+true%22&type=code + PGO integration as a USE flag
* Check *BSD (FreeBSD, OpenBSD, NetBSD, DragonflyBSD, etc) repositories for enabled PGO over projects
* Write about an observation that if in the upstream there is no PGO support - maintainers almost always do not enable PGO in their packages
* Intel compiler has support for Sampling PGO too even outside `perf` - https://www.intel.com/content/www/us/en/docs/dpcpp-cpp-compiler/developer-guide-reference/2024-0/hardware-profile-guided-optimization.html. Add to the article.
* Talk with Pavel Kosov about PGO in Telegram: https://www.youtube.com/watch?v=CvbIPGSnes8 (awesome talk btw)
* Test profiles PGO profiles overlap between OS with `llvm-profdata overlap` - they are not compatible at all for some reason. Write about it.
* Write more about `llvm-profdata` tool usage. Check the documentation for the tool and find missing parts in it
* Check https://www.youtube.com/watch?v=GBtQrYx_Jbc talk about PGO
* Experiments regarding compiler options and PGO profile stability:
  - Enable/Disable LTO: ZERO OVERLAP
  - Change opt-level from 3 to 2: ZERO OVERLAP
* Add a link about some issues with the current build infrastructure for PGO in Rust: https://github.com/yamafaktory/jql/discussions/244#discussioncomment-7665597
* Write a disclaimer in the article about why most tested projects in Rust (because it's easier to apply PGO there - tooling matters)
* Add to the article: https://discourse.llvm.org/t/couple-of-general-questions-about-pgo/72279. Here there are some interesting insights about PGO state in the compiler. https://discourse.llvm.org/t/status-of-ir-vs-frontend-pgo-fprofile-generate-vs-fprofile-instr-generate/58323 thread also has many valuable insights about FE PGO vs IR PGO.
* PGO questions on LLVM forum - https://discourse.llvm.org/t/profile-guided-optimization-pgo-related-questions-and-suggestions/75232
* Find information about single and multi-threaded PGO counters - in Clang its https://clang.llvm.org/docs/UsersManual.html#cmdoption-fprofile-update . Related GCC bug about PGO profiles slowness: https://gcc.gnu.org/bugzilla/show_bug.cgi?id=89307 . GCC docs for it: https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html#index-fprofile-update . Problems in practice with such an approach: https://reviews.llvm.org/D34085 . More discussion in https://lists.llvm.org/pipermail/llvm-dev/2014-April/072172.html
* Extract PGO recommendations from Pavel Kosov's talk about PGO from C++ Russia to the article
* Reproducibility issues from PGO - GCC example: https://gcc.gnu.org/bugzilla/show_bug.cgi?id=93398
* ripgrep does not want to maintain PGO. it's a good indicator that maintaining PGO costs something - https://github.com/BurntSushi/ripgrep/issues/1225#issuecomment-1828965231
* Write more about the CompilerGym in the article: https://github.com/facebookresearch/CompilerGym
* PGO and user hints: https://github.com/llvm/llvm-project/issues/58189
  - Go: https://github.com/golang/go/issues/64460
  - GCC: https://gcc.gnu.org/bugzilla/show_bug.cgi?id=112806
  - GraalVM: https://github.com/oracle/graal/discussions/7990
* Write in the article about manual profile dump via the compiler runtime profile infra
* Suggest an idea about AutoFDO at scale to Grafana Pyroscope (https://github.com/grafana/pyroscope) - https://github.com/grafana/pyroscope/discussions/2783
* Write a note about LLVM static counters warning and possible mitigations: https://bugs.fuchsia.dev/p/fuchsia/issues/detail?id=111987
* Discuss comment from the Valve engineer: https://github.com/ValveSoftware/SteamOS/issues/1287#issuecomment-1840835012
* Sampling PGO support via `perf` on Mobile devices? Is it possible to perform?
* Cover question about CSIR PGO vs BOLT in the article
* PGO for Android games: https://developer.android.com/games/agde/pgo-overview and GDC talk (https://www.youtube.com/watch?v=CNbpFTyHOe8)
  - Contact the author and ask about more advanced PGO techniques: CSIR PGO, Sampling PGO, maybe BOLT/Propeller
* Documentation issues about PGO even from Google: https://issuetracker.google.com/issues/315487999
* Write about multiple instrumentation and sampling tooling and their compatibility with PGO infrastructure: https://thume.ca/2023/12/02/tracing-methods/
* magic-trace profiles and PGO: https://github.com/janestreet/magic-trace/discussions/285
* Apple question about PGO:
  - Apple StackExchange: https://apple.stackexchange.com/questions/467215/profile-guided-optimization-pgo-in-xcode , https://apple.stackexchange.com/questions/467182/sampling-profile-guided-optimization-pgo-with-xcode-and-clang , https://apple.stackexchange.com/questions/467218/profile-guided-optimization-pgo-differences-between-clang-and-apple-clang
  - Apple developer forum: https://developer.apple.com/forums/thread/742840
* Unity post for using LTO, PGO and PLO for games: https://forum.unity.com/threads/add-lto-pgo-and-plo-recommendations-to-il2cpp-documentation.1525315/
* Fix Android PGO docs: https://source.android.com/docs/core/perf/pgo - https://issuetracker.google.com/issues/315464624
* An idea about macos to linux profile converter (original link somewhere in the llvm-mingw repo)
* GraalVM's LLVM toolchain and PGO optimization: https://github.com/oracle/graal/discussions/7989
* Write a note that I made a mistake with FE PGO and IR PGO too - that's why all my C/C++ tests use FE PGO instead of IR PGO
* Ask GraalVM about dumping PGO profile to a memory region and other PGO-related runtime intrinsics: https://github.com/oracle/graal/discussions/7991
* little-fs PGO benchmarks: https://github.com/littlefs-project/littlefs
* GraalVM PGO profile optimization strategy for non-executed code: https://github.com/oracle/graal/discussions/7999
* Arch feature levels as an optimization idea: https://www.phoronix.com/news/Ubuntu-x86-64-v3-Experiment
  - Find more Phoronix publications on this topic
* AOSP switches to AFDO: https://issuetracker.google.com/issues/315464624
* Write about the instruction set bumping like https://www.phoronix.com/news/Ubuntu-x86-64-v3-Experiment
* Add to the article a note about using git blame to find responsible for PGO people - a nice thing to remember during the PGO journey
* Dataframe performance regressions with PGO in the benchmarks - what is the reason? Needs to be investigated
* Write a note about reporting problems with PGO to the upstream - how to do it properly? Especially, if we are talking about debuggability of the performance regression
* Check profile generation reproducibility across runs
* Get in touch with https://twitter.com/ohmypy/status/1744664212952297640 regarding the SQLite and PGO situation
* Some projects fail to build easily with PGO instrumentation: https://github.com/rust-lang/rust/issues/119848
* Add a funny note to the article about reaching the Pastebin limit for PGO results - I needed to switch to GitHub's Gist since this limit.
* Zen has a regression in the lexer benchmark - it needs to be investigated.
* Add a note to the article about debugging performance regressions with PGO - how you can do it
* Perform PGO benchmarks on Darktable - it has some benchmarks: https://github.com/darktable-org/darktable/blob/master/src/tests/benchmark/darktable-bench
* Talk with SerpentOS maintainers about enabling PGO for their packages: https://serpentos.com/
* Talk with Nate Graham about trying to enable PGO for KDE. Seems KDE cares about performance too - https://www.phoronix.com/news/KDE-Plasma-6-Next-Month .
  - Sent a message on 28.01.2024 via a website form
* Check PGO applicability with starship.rs
* Add PGO request to https://github.com/CHIP-SPV/chipStar
* Add PGO documentation to Lace: https://github.com/promised-ai/lace/issues/172#issuecomment-1917406027
* Track Datadog experience with PGO: https://twitter.com/felixge/status/1754772490889396415
* Run PGO benchmarks on https://github.com/StractOrg/stract
* Write about a failed idea of extending PGO support across the ecosystem and GSoC (was discussed at FOSDEM 2024 with Google representatives)
* Perform PGO benches on https://github.com/vercel/turbo/tree/main/crates/turbopack-bench
* Lack of maintenance resource for PGO: https://github.com/facebook/hhvm/issues/9385#issuecomment-1943760010
* Discuss a complicated case about applying PGO to a program that is written in multiple programming languages
* Discuss the PGO state across operating systems. Discuss about PGO-edge experience in distributions like ClearLinux, CachyOS, Gentoo
* Add a note about PGO in gamedev from drivers perspective: shader compilation, etc. - my Steam Deck case
* PGO support for WASM in Clang: https://discourse.llvm.org/t/profile-guided-optimization-pgo-and-webassembly/77090
* Write a note about Sampling PGO and special hardware requirements like Intel LBR or AMD BRS. It's not strictly required but recommended. What is the difference from the performance perspective in practice - needs to be investigated (but modern CPUs have support for this already. However, what about ARM - https://lwn.net/Articles/951316/ (Linux 6.7-rc1 - quite new stuff). What about RISC-V or LoongSoon?). ARM CPUs with it - https://en.wikipedia.org/wiki/ARM_architecture_family . What about PowerPC (as it was asked here - https://stackoverflow.com/questions/48616075/whats-the-equivalent-of-last-branch-record-lbr-on-arm-and-powerpc/)
* Do I need to prepare more materials about PGO in C#?
* Write a note about SPECINT benchmark in the article - it's a commonly used benchmark for evaluation performance of different compiler optimizations
* Great paper about different sampling FDO approaches: https://hpctest.cs.tsinghua.edu.cn/papers/tc11.pdf
* Check information about LIPO (lightweight IPO): https://static.googleusercontent.com/media/research.google.com/ru//pubs/archive/36355.pdf What is the current state of the LIPO in the compilers? ThinLTO is not the same thing. However, here I see some that ThinLTO somehow supports profile data: https://llvm.org/devmtg/2015-04/slides/ThinLTO_EuroLLVM2015.pdf . My question is here: https://discourse.llvm.org/t/thinlto-and-profile-based-module-groups/77120
* https://clang.llvm.org/docs/ThinLTO.html - mold linker is not mentioned. Recheck if mold actually supports it, and if supports - change the documentation
* Add to the article a note about llvm-profgen and its limitation with non-LBR profiles: https://discourse.llvm.org/t/llvm-profgen-with-non-lbr-profiles/77140
* Check https://storage.googleapis.com/gweb-research2023-media/pubtools/pdf/36358.pdf paper for LBR efficiency in Sampling PGO in practice
