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
* Describe different PGO applications scenarios: for SaaS, for open-source, for closed-source but delivered to the customers, etc.
* Using built-in benchmarks can be not so good idea: benchmark code coverage issues, built-in benchmarks can be too slow in the instrumentation modes
* Check `simdjson` with PGO: https://github.com/simdjson/simdjson/blob/master/benchmark/CMakeLists.txt
* Check new Go PGO when Go 1.21 compiler update will arrive in the distrubutions. Some CPU-heavy scenarios like log parsing should be a good candidate to test. Right now it has a lot of places to improve: https://github.com/golang/go/issues?q=is%3Aissue+is%3Aopen+PGO
* Add PGO information to Rsyslog: https://github.com/rsyslog/rsyslog/issues/5048#issuecomment-1694533339
* Not everywhere PGO is available (e.g. WASM): https://github.com/rust-lang/rust/issues/81684
* Committing PGO profiles into a project - transparency problem if you do not describe how this profile is collected
* "Awesome-X" repositories are good places to start PGOify some domains
* Probably a good list to optimize command-line tooling: https://gist.github.com/sts10/daadbc2f403bdffad1b6d33aff016c0a
* Check https://trofi.github.io/posts/243-gcc-profiler-internals.html article about GCC profiler internals
* Discussion BOLT and Propeller: https://discourse.llvm.org/t/practical-compiler-optimizations-for-wsc-workshop-us-llvm-dev-meeting-2023/73998/
* What is https://prodfiler.com/ ? Check their work and probably write about them in the article
* Play locally with Propeller: https://github.com/google/llvm-propeller and discuss integrating it into the cargo-pgo
* Do not forget to mention about monitoring actual workload and tracking - does it match your current PGO-profile or not
* Example of an application where PGO exists but not used for prebuilt binaries: https://github.com/ispc/ispc/issues/2687
* Add a note about unavailable LBR in virtual machines (that can limit AutoFDO usage in virtualized envs)
* Why GraalVM (and consequently PGO in Java world) have so low adoption? I need to investigate it. GraalVM metadata repo - https://github.com/oracle/graalvm-reachability-metadata
* Write about Clang (LLVM) vs GCC PGO implementations: GCC has more PGO switches, Clang has more recently-researched PGO-related things inside, etc.
* Prepare an answer in Angie (Nginx fork) repository about PGO
* https://github.com/stevinz/awesome-game-engine-dev
* Possible LTO issues in compilers: https://github.com/rust-lang/rust/pull/113433
* Good tracking issue for the Rustc optimizations: https://github.com/rust-lang/rust/issues/103595
* PGO-related issue in Rustc: https://github.com/tensorchord/pgvecto.rs/issues/43#issuecomment-1788324707
* Check Android (and Android-dependent) project for PGO/BOLT usage
* Add information about PGO missed optimizations like GCC (https://gcc.gnu.org/bugzilla/buglist.cgi?quicksearch=PGO&list_id=403385), Clang (TODO), Go compiler (TODO), Rustc (TODO)
* Add CachyOS project as an example where BOLT is integrated at most to the OS build pipelines
* An interesting case of a try to enable PGO for Foot in Void Linux: https://github.com/void-linux/void-packages/issues/29526
* Create a generic PGO + BOLT issue in Gentoo (links reporting in the bugzilla restricted until 15.11.2023)
* Add to OpenWRT request about building its GCC with PGO
* Check https://hpsfoundation.github.io/ and its projects. Probably they will be interested in PGO as well since it's HPC foundation
* Check Windows-based package managers like Chocolatey and others for PGO enablement
* A comment about the build resources in Void Linux (why they do not use PGO): https://github.com/void-linux/void-packages/issues/39652#issuecomment-1265308710
* Become a Mageia maintainer to push PGO/PLO activity: https://bugs.mageia.org/show_bug.cgi?id=32511
* Check Firefox and Chromium derivatives like Thunderbird and Tor browser for PGO
* Clear Linux PGO packages: https://github.com/search?q=org%3Aclearlinux-pkgs+%22pgo+%3D+true%22&type=code + PGO integration as a USE flag
* Check *BSD (FreeBSD, OpenBSD, NetBSD, DragonflyBSD, etc) repositories for enabled PGO over projects
* Check for PGO compilers at https://cgit.freebsd.org/ports/tree/lang - can be a valuable source for the PGO experiments
* Add a note about missing BRS support on Zen 3 desktop CPUs required for AutoFDO: https://discord.com/channels/636084430946959380/930647188944613406/1175827177405698170
* Write about an observation that if in the upstream there is no PGO support - maintainers almost always do not enable PGO in their packages
* Intel compiler has support for Sampling PGO too even outside `perf` - https://www.intel.com/content/www/us/en/docs/dpcpp-cpp-compiler/developer-guide-reference/2024-0/hardware-profile-guided-optimization.html. Add to the article.
* Talk with Pavel Kosov about PGO in Telegram: https://www.youtube.com/watch?v=CvbIPGSnes8 (awesome talk btw)
* Test profiles PGO profiles overlap between OS with `llvm-profdata overlap` - they are not compatible at all for some reason. Write about it.
* Write more about `llvm-profdata` tool usage. Check the documentation for the tool and find missing parts in it
* Write about PGO profiles and assembly insertions (PGO cannot help here but BOLT can)
* Check https://www.youtube.com/watch?v=GBtQrYx_Jbc talk about PGO
* Experiments regarding compiler options and PGO profile stability:
  - Enable/Disable LTO: ZERO OVERLAP
  - Change opt-level from 3 to 2: ZERO OVERLAP
* Debian sources: https://sources.debian.org/
* Write about atomic counters update during the profiling phase (check both GCC and LLVM. Probably other compilers should be checked as well)
* Add a link about some issues with the current build infrastructure for PGO in Rust: https://github.com/yamafaktory/jql/discussions/244#discussioncomment-7665597
* Write a disclaimer in the article about why most tested projects are tested with Clang (I have nothing against GCC and GPL) and in Rust (because it's easier to apply PGO there - tooling matters)
* Add to the article: https://discourse.llvm.org/t/couple-of-general-questions-about-pgo/72279. Here there are some interesting insights about PGO state in the compiler. https://discourse.llvm.org/t/status-of-ir-vs-frontend-pgo-fprofile-generate-vs-fprofile-instr-generate/58323 thread also has many valuable insights about FE PGO vs IR PGO.
* PGO questions on LLVM forum - https://discourse.llvm.org/t/profile-guided-optimization-pgo-related-questions-and-suggestions/75232
* Find information about single and multi-threaded PGO counters - in Clang its https://clang.llvm.org/docs/UsersManual.html#cmdoption-fprofile-update . Related GCC bug about PGO profiles slowness: https://gcc.gnu.org/bugzilla/show_bug.cgi?id=89307 . GCC docs for it: https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html#index-fprofile-update . Problems in practice with such an approach: https://reviews.llvm.org/D34085 . More discussion in https://lists.llvm.org/pipermail/llvm-dev/2014-April/072172.html
* Extract PGO recommendations from Pavel Kosov's talk about PGO on C++ Russia to the article
* github.com recommendation system sometimes brings interesting projects for PGO, huh :D
* GCC merging multiple profiles: https://gcc.gnu.org/bugzilla/show_bug.cgi?id=47618
* Lapce maintainer does not believe in PGO: https://github.com/lapce/lapce/discussions/2817#discussioncomment-7674201
* Reproducibility issues from PGO - GCC example: https://gcc.gnu.org/bugzilla/show_bug.cgi?id=93398
* GCC and PGO profiles compatibility: https://gcc.gnu.org/bugzilla/show_bug.cgi?id=112717
* Sent an email about PGO to https://soqol.ru/ via the official contact form on the website - waiting for the response
* PGO and mobile development - what is the status?
* PGO and embedded - what is the status?
* ripgrep does not want to maintain PGO. it's a good indicator that maintaining PGO costs something - https://github.com/BurntSushi/ripgrep/issues/1225#issuecomment-1828965231
* ast-grep as an interesting example where `cargo-pgo` failed and I cannot figure out - what is wrong? Seems like `tree-sitter` is the issue since it's ont optimized with PGO via `cargo-pgo`
* Write more about the CompilerGym in the article: https://github.com/facebookresearch/CompilerGym
* PGO and cold paths in LLVM and GCC: https://github.com/llvm/llvm-project/issues/63024
* PGO and user hints: https://github.com/llvm/llvm-project/issues/58189
  - What about other compilers like GCC?
  - Go: https://github.com/golang/go/issues/64460
  - GCC: https://gcc.gnu.org/bugzilla/show_bug.cgi?id=112806
* Clang recommendataion IR PGO over FE PGO: https://github.com/llvm/llvm-project/issues/45668
* Write in the article about manual profile dump via the compiler intrinsics
* Interesting PGO benefit - startup time. Can be important for Serverless apps - add to the article
* Suggest an idea about AutoFDO at scale to Grafana Pyroscope (https://github.com/grafana/pyroscope) - https://github.com/grafana/pyroscope/discussions/2783
* Check project for using `-fprofile-instr-generate` for PGO and recommend switching to `-fprofile-generate`
* Ask Go developers about Sampling PGO support via external `perf` profiles or other external profilers
* Go PGO and weighted profiles support: https://github.com/golang/go/issues/64487
* Go PGO and Linux perf's tooling: https://github.com/golang/go/issues/64489
* Check runtime PGO capabilities in GCC: https://gcc.gnu.org/bugzilla/show_bug.cgi?id=112829
* Check Cloudlfare projects and add PGO requests to them
* PGO for size optimization - an experiment results: https://bugs.fuchsia.dev/p/fuchsia/issues/detail?id=79458
* Write a note about LLVM static counters warning and possible mitigations: https://bugs.fuchsia.dev/p/fuchsia/issues/detail?id=111987
* Write to Steam developers about PGO and optimizing their packages harder to improve Steam Deck performance and extend battery life
