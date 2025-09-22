# Are we LTO yet?

Plot:

Main parts:

* A bit of theory about LTO - mostly links to other articles
* Why LTO is useful nowadays?
* LTO issues from different perspectives: compilers, applications, programming languages, etc.
* LTO pieces of advice
* My LTO journey
* Q&A
* What we can do better together for LTO (and other similar things)?
* Fun stories
* End

1) Intro
2) What is LTO - a few words with many links
3) LTO issues in: different programming languages, different build systems
4) Current LTO state in different programming languages, build systems, applications and libraries
5.0) Pieces of advice for enabling LTO in your project
5.1) Idea about increasing the default performance level for the industry and why Rust is a good place to start. Continuous performance tracking (like https://perf.rust-lang.org/ or like Google OSS-Fuzz but for performance)
6) My efforts to do so in: Rust, Python libs, IDEs and problems on my way
7) Different findings and stories during the "research"
8) Possible future ideas and afterwords

## What is LTO

In this article, I don't want to give a detailed description about what LTO is, how does it work internally in compilers, etc. - this topic has already a good coverage in the Internet. Let's get a *very* quick overview. For more inquisitive people I will leave many helpful on the topic below. However, as always, the most interesting insides about LTO quirks in practice can be taken only from LTO implmentations in compilers, commit messages and bug trackers ;)

Link-Time Optimization aka LTO is a compiler optimization technique that allows compilers perform optimization between compilation units (e.g. a translation unit in C++, crates in Rust). That's it!

TODO: LTO kinds: Fat and Thin, add links to the corresponding talks, papers, presentations, etc.

TODO: naming differences between LTO, WPO, [Interprocedural optimizations](https://en.wikipedia.org/wiki/Interprocedural_optimization)
TODO: Fat LTO vs ThinLTO: https://discourse.llvm.org/t/clang-lld-thin-lto-footprint-and-run-time-performance-outperformed-by-gcc-ld/78997/6 + ThinLTO vs Full LTO: https://discourse.llvm.org/t/rfc-a-unified-lto-bitcode-frontend/61774/52

* https://llvm.org/devmtg/2016-11/Slides/Amini-Johnson-ThinLTO.pdf - original slides
* https://convolv.es/guides/lto/ (+ comments here: https://news.ycombinator.com/item?id=38215535) - a good article about LTO
* Distributed and incremental ThinLTO - https://blog.llvm.org/2016/06/thinlto-scalable-and-incremental-lto.html
* ThinLTO CppCon video from Teresa Johnson: https://www.youtube.com/watch?v=p9nH2vZ2mNo
* LLVM docs about LTO: https://llvm.org/docs/LinkTimeOptimization.html
* LLVM LTO-related sources: https://github.com/llvm/llvm-project/tree/main/llvm/lib/LTO
* GCC docs about LTO internals: https://gcc.gnu.org/onlinedocs/gccint/LTO.html
* GCC LTO-related sources: https://github.com/gcc-mirror/gcc/tree/master/gcc/lto

## LTO benefits

TODO: Performance improvement from LTO: https://github.com/netdata/netdata/issues/15338#issuecomment-1630287811
TODO: Tor saves 14% binary size for Arti (Tor)
TODO: 10% speed improvements from ThinLTO: https://github.com/MaterializeInc/materialize/blob/main/Cargo.toml#L250
TODO: https://postgrespro.com/list/thread-id/2634776 - LTO for PostgreSQL gives +10% on math at least
TODO: Why does binary size matter? https://github.com/facebookincubator/fastmod/issues/57#issuecomment-2978333044
TODO: Binary is large because of Rust: https://github.com/icann/icann-rdap/issues/117#issuecomment-2751856737
TODO: Performance improvements from LTO for small apps: https://github.com/tsowell/wiremix/issues/1#issuecomment-2846168920
TODO: Improvements are not so impressive for everyone: https://github.com/anacrolix/possum/issues/5#issuecomment-2628762228

## LTO issues

TODO: Increased build time, memory consumption, compiler bugs, software bugs

TODO: cover two separate issues - ThinLTO vs FatLTO
TODO: People sometimes is confused from LTO results due to ThinLTO stuff: https://github.com/EmbarkStudios/cargo-deny/issues/701

### Increased build times

TODO: usually, it's 1.5x - 2x build time increase from Fat LTO. As an example: https://github.com/meilisearch/meilisearch/pull/5106#issue-2711334231 + https://github.com/martinvonz/jj/discussions/4463#discussioncomment-10673633 + https://github.com/numaproj/numaflow/blob/53f8c182f42a7c34ce7133991e7764bb2eae5921/rust/Cargo.toml#L50 - another build time increase report (12s -> 133s just from enabling FatLTO)
TODO: less deps you have - smaller overhead will be in practice
TODO: Projects are switching from Fat to Thin LTO due to build times: https://github.com/neon-mmd/websurfx/issues/516
TODO: ThinLTO also increases build time - https://github.com/facebook/buck2/blob/main/Cargo.toml#L416 - 50s to 84s with ThinLTO
TODO: insert a meme about an electricity bill due to enabled LTO :D
TODO: Build time concerns from Fat LTO: https://github.com/shadps4-emu/shadPS4/pull/1636#issuecomment-2538866412
TODO: Comments about build time and LTO: https://github.com/deuxfleurs-org/garage/blob/3f4ab3a4a38e382fc54c15580862af55b0257848/Cargo.toml#L149
TODO: LTO sometimes reduces compilation time (@foxtran story)
TODO: Suggest ThinLTO to Ruffle instead of disabling LTO at all: https://github.com/search?q=repo%3Aruffle-rs%2Fruffle+thinlto&type=code
TODO: LTO overhead for Debug profiles: https://gitlab.com/wireshark/wireshark/-/merge_requests/3735
TODO: Limited build resource availability: https://github.com/quickwit-oss/quickwit/issues/5813#issuecomment-2995841734

### Increased memory consumption

TODO: Fyrox Full LTO - 17 Gib RAM in peak
TODO: Mention LTO, memory usage and RAM limits like https://docs.github.com/en/actions/using-github-hosted-runners/using-github-hosted-runners/about-github-hosted-runners
TODO: Write a note about too shitty system OOM for LTO and why systemd-oomd (and similar things) are important - at least no freezing system
TODO: Funny scripts about memory calculation for parallel LTO jobs: https://github.com/swiftlang/swift/blob/main/utils/build_swift/build_swift/defaults.py#L87 + https://github.com/swiftlang/swift/blob/main/utils/build_swift/build_swift/defaults.py#L102

Since the compiler needs to perform some optimizations on **whole** program, it consequently needs to store many corresponding information in memory to work with it. It leads to the increased memory consumption. I don't know a good way to estimate how much of RAM LTO will take for a random project since it can depend on multiple factors like a compiler (different compilers have different LTO implementations), a compiler version (since LTO implementations can vary between commits/releases from the performance perspective too), etc.

Is there any difference between compilers from the memory perspective? I think they are but I didn't perform such measurements yet. The same applies to different compiler versions - the in theory is also "yes" but I suppose the chance of getting troubles from incresed memory consumption with LTO after the compiler upgrade are too low. Though, I didn't check it too.

### Bugs

#### C++

TODO: Incremental LTCG is not compatible with ASAN: https://developercommunity.visualstudio.com/t/starting-application-with-address-sanitizer-in-rel/1537669 - and ofc issue was closed due to other bugs :DDDD
TODO: https://github.com/dbry/WavPack/pull/103#discussion_r626374911 - LTO can break UB-dependent code

#### Rust

TODO: switching from Thin to Fat due to bugs on Windows: https://github.com/rust-lang/rust/issues/98302#issuecomment-1436474857

In Rustc (the default compiler for Rust) I can quickly say about the following LTO-related bugs/limitations:

* Some projects [cannot](https://github.com/rust-lang/rust/issues/115344) be compiled when LTO and Profile-Guided Optimization (PGO) are used at the same time but if we enable only LTO or only PGO for such projects - everything works fine. Not a critical bug for now (since PGO sadly is not widely used nowadays across the industry) but anyway. Hopefully, this bug will be fixed [soon](https://github.com/rust-lang/rust/pull/133250).
* A strange [error](https://github.com/rust-lang/rust/issues/132677) when ThinLTO and [split-debuginfo = "packed"](https://doc.rust-lang.org/rustc/codegen-options/index.html#split-debuginfo) are enabled at the same time. Luckily, this problem can be mitigated by using Fat LTO or by disabling `split-debuginfo`. Since the problem is not widespread, I don't think it will be fixed in the near future.
* A strange error was found in Maturin when LTO was [enabled](https://github.com/PyO3/maturin/discussions/2337#discussioncomment-11420511) - a built for `ppc64le` platform started to fail with an `failed to get bitcode from object file for LTO (could not find requested section)` error. After a very quick investigation I found two similar issues in the upstream with the same error message - [one](https://github.com/rust-lang/rust/issues/129888), [two](https://github.com/rust-lang/rust/issues/94232). I don't know the root cause of the issue and I suppose the issue won't be fixed soon either.

More can be checked by [this](https://github.com/rust-lang/rust/issues?q=is%3Aissue%20state%3Aopen%20lto%20label%3AA-LTO) filter on GitHub.

TODO: Dylib-lto Cargo feature available only as unstable: https://doc.rust-lang.org/beta/unstable-book/compiler-flags/dylib-lto.html
TODO: Funny cross-language LTO debugging on Windows: https://users.rust-lang.org/t/cross-language-lto-doesnt-really-work-on-windows/118805
TODO: LTO build issues: https://github.com/meilisearch/meilisearch/issues/2718
TODO: LTO bugs are everywhere: https://github.com/rust-lang/rust/issues/141306
TODO: https://github.com/rust-lang/rust/issues/121124 - LTO issues in Rustc on specific platforms

### Decreased performance

TODO: A slowdown from LTO: https://github.com/rust-lang/rust/issues/48371 + https://github.com/rust-lang/rust/issues/47745

### Other

TODO: Split Dwarf and LTO limitations: https://github.com/CleverRaven/Cataclysm-DDA/pull/74067#issuecomment-2132435397 + https://gcc.gnu.org/bugzilla/show_bug.cgi?id=88389 + https://discourse.llvm.org/t/lto-thinlto-and-split-dwarf/70927

TODO: Cross-language LTO limitations: https://github.com/gyscos/zstd-rs/blob/main/Cargo.toml#L47

## Current LTO state in different ecosystems

TODO: Rust LTO state

### OS distributions

TODO: add information about C++ LTO in distributions
TODO: Recommended using cross-lang LTO too by Ubuntu: https://documentation.ubuntu.com/rockcraft/en/latest/common/craft-parts/reference/plugins/rust_plugin/#performance-tuning

However, if a distribution enables LTO for C or C++ programs, it doesn't mean that LTO is enabled for other applications that are written in other LTO-supported technologies. Since during the last few years I was working closely with the Rust ecosystem, let's check Rust-based apps:

* Debian: No ([Debian's Salsa merge request](https://salsa.debian.org/rust-team/rust/-/merge_requests/41) + [issue](https://salsa.debian.org/rust-team/debcargo/-/issues/67))
* OpenSUSE: No ([Reddit post](https://www.reddit.com/r/openSUSE/comments/1hh4qe4/linktime_optimization_lto_by_default_for_rust/))
* Fedora: No ([Reddit post](https://www.reddit.com/r/Fedora/comments/1hrzx70/linktime_optimization_lto_for_rust_packages_by/), [Fedora Discussion](https://discussion.fedoraproject.org/t/link-time-optimization-lto-for-rust-packages-by-default-in-fedora/140086), [Build macros for Rust](https://pagure.io/fedora-rust/rust-packaging/blob/main/f/macros.d/macros.rust))
* Arch Linux: No ([GitLab merge request](https://gitlab.archlinux.org/pacman/pacman/-/merge_requests/131))

TODO: write more about the reasons why LTO can be ignored my maintainers in OSs

As you see, for Rust LTO is not enabled by distributions by default. I hope one day the sitaution will change.

## LTO for applications

TODO: LTO, huge software installation and weak computers - it could be problem

## LTO for libraries

TODO: LTO for native libraries with bindings

## LTO integration pitfalls and traps

TODO: Duckstation offers LTO only in the README file: https://github.com/stenzek/duckstation?tab=readme-ov-file#building-1 and is enabled in all corresponding build scripts
TODO: https://github.com/qarmin/czkawka - an example when LTO is disabled in configs but is enabled (via sed hacks) in CI

## LTO integration guideline

TODO: Build with LTO in CI to catch LTO-triggered errors (especially true for C/C++ projects)
TODO: `debug = true` makes a huge impact on RAM usage with LTO
TODO: Add conduwuit examples as a Rust project with many dedicated profiles: https://github.com/girlbossceo/conduwuit/blob/main/Cargo.toml#L537
TODO: A trick for enabling LTO only for one subproject in Rust ecosystem: https://www.reddit.com/r/rust/comments/w4pzd2/rust_cargo_workspace_how_to_specify_different/
TODO: you can disable LTO for some targets - OpenRCT2 enables LTO only for certain build configs: https://github.com/search?q=repo%3AOpenRCT2%2FOpenRCT2%20IPO_ENABLED_BUILDS&type=code (and disables them in CI sometimes - https://github.com/OpenRCT2/OpenRCT2/pull/22695)

## Difficulties with enabling LTO by default

TODO: documentation issues - https://github.com/rust-lang/rust/issues/48518 + at least in Rust people mess sometimes Thin Local LTO with Thin LTO (it's completely separate things)
TODO: C and C++ problems with UB
TODO: Rust problems with profile backward compatibility
TODO: LTO bugs in compilers - enumerate all of them here like Rustc bugs (LTO + PGO, LTO + split-debuginfo, other bugs)

### Uncovered issues

TODO: More build issues with LTO - https://github.com/FreeCAD/FreeCAD/issues/13173
TODO: A good comment about LTO and hidden UBs: https://github.com/FreeCAD/FreeCAD/issues/6698#issuecomment-1977945753
TODO: Typical uncovered errors with LTO: https://github.com/aseprite/aseprite/issues/4413#issue-2237029279
TODO: Another LTO issue that was not investigated and just switched off LTO for a project: https://github.com/redlib-org/redlib/issues/50#issuecomment-1939755101

## LTO by default

TODO: people wondering why LTO is not enabled by default - https://github.com/theaddonn/brarchive-rs/issues/1#issuecomment-2511682461
TODO: People don't enable LTO since the Cargo upstream doesn't enable it: https://github.com/PyO3/maturin/issues/1529
TODO: Not updated tools are not LTO optimized without changing defaults: https://github.com/Keats/dbmigrate (like this and maaaany others)
TODO: Why I propose to use lto = true instead of lto = "fat" or lto = "thin". Fat vs Thin LTO vs an abstraction over them

### Programming languages "committees"

Every language has some sort of governance mechanism. It could be a small (like several or even a single-person!) langs (e.g. many [DSL](https://en.wikipedia.org/wiki/Domain-specific_language)s), company-owned langs such as Go(oogle), Swift (Apple), etc, or with more decentralized governance mechanisms like [C++](https://isocpp.org/std/the-committee) or [Rust](https://www.rust-lang.org/governance). Maybe I missed some other forms but nevermind.

TODO: add a note below about ISO and its prevention of free C++ standard. Make a note about blaming people to participate in ISO structure with this limitation (and many others - the undefined question once again)

I am talking about programming languages not like just the "language specification" (e.g. the [C++ Standard](https://isocpp.org/std/the-standard). Fun fact: you cannot read the specification for free due to ISO quirks. If you still want, you can find a free rendered version [here](https://eel.is/c++draft/)).

#### C++

TODO: finish the part below about my standardization experience

I personally experienced several forms of participation in a language design process:

* Talking at different conferences with people from the C++ committee about proposals, future design directions, etc.
* Worked with Russian C++ standardization group (national body: [Antony Polukhin](https://github.com/apolukhin))
* Wanted to become a National Body for the C++ committee from Belarus but it required so much bureacracy that we (me and my emplyer at that time) decided to postpone the idea. Later I dropped the idea since I lost faith in the current C++ governance process due to many factors (some of them are described above).

#### Rust

Rust has **far better** ecosystem tooling support compared to other languages. It has a tool - [Crater](https://github.com/rust-lang/crater) - that allows you to perform experiments across multiple Rust codebases to check your assumptions/find bugs/gather statistics at scale.

Rust has [rustc-perf](https://github.com/rust-lang/rustc-perf) - a tool + the [website](https://perf.rust-lang.org/) where you can **continuosly** track the Rustc compiler performance. Fun fact: the performance-oriented tool also has [disabled](https://github.com/rust-lang/rustc-perf/blob/master/Cargo.toml) LTO. Of course, I understand possible reasons behind that like "we don't care here too much about performance", "it wastes our CI time for almost nothing, etc." but I see it as a bit of irony, isn't it? :D

There are already some discussions about changing Cargo defaults: a lengthy [talk](https://github.com/rust-lang/cargo/issues/4122) about stripping binaries in the Release mode by default (and the [solution](https://github.com/rust-lang/cargo/commit/fbf9251b4d3ddfc63ad0760ac121211a6c48a2d9)),

From my perspective, this awesome infrastructure still can be improved in different directions. E.g. even if `rustc-perf` [supports](https://github.com/rust-lang/rustc-perf/pull/2010) testing multiple Rustc backends (LLVM and Cranelift) but on CI only the LLVM backend is tested continuosly. If Cranelift becomes popular in the future - it will be nice to test it too. [Here](https://rust-lang.github.io/rust-project-goals/2025h1/perf-improvements.html) you can read more about already accepted changes to the tool.

TODO: difficult committee structures that it needs additional explanations - https://www.reddit.com/r/cpp/comments/1h0ximn/comment/lz7j2rx/
TODO: a committee over standardization processes? https://www.incits.org/home/ was mentioned on the isocpp.org
TODO: Rust Binary size working group links to cover in the article: https://github.com/rust-lang/wg-binary-size/issues/3 + others
TODO: Cargo links: https://github.com/rust-lang/cargo/issues/784#issuecomment-410365426 + https://github.com/rust-lang/cargo/issues/4122 - lengthy discussions about changing the defaults. More can be found in the Rust Zulip
TODO: Rust perfbot and binary sizes: https://github.com/rust-lang/rustc-perf/issues/1888 + https://github.com/rust-lang/rustc-perf/issues/145 + https://github.com/rust-lang/rustc-perf/pull/1772
TODO: Rust dev team has other priorities for various reasons: Rust main pain points in other places (since Rust is already "fast-enough by default")
TODO: The C++ ecosystem performance Question
TODO: I am too tired of continuous battling with people for free so don't expect much activity from me here
TODO: Does rustc-perf test Cranelift? Asked Bjorn about that, waiting for the answer . If not, I can create an issue for rustc-perf
TODO: Interesting LTO surprises in Cargo: https://github.com/rust-lang/cargo/issues/14612 + https://github.com/rust-lang/cargo/issues/14575 + https://github.com/rust-lang/cargo/issues/9672 + https://github.com/rust-lang/cargo/issues/7491

### People awareness

One of the interesting reasons of why LTO is not enabled in project... people just don't know about LTO! Even if LTO is available in compilers for decades, many [people](https://github.com/theiskaa/mdp/issues/7#issuecomment-2467361983) [still](https://github.com/mr-adult/JFC/issues/2#issuecomment-2442816728) [don't](https://github.com/supreetsingh10/lyricist/issues/1#issuecomment-2454878919) [know](https://github.com/agentic-labs/lsproxy/issues/61#issuecomment-2457848586) [about](https://github.com/MuongKimhong/BaCE/issues/1#issuecomment-2460528962) LTO - here I shared just *some* evidences, not all of them.

TODO: Lack of confidence in LTO: https://github.com/mohanson/gameboy/issues/43#issuecomment-2403730081
TODO: Write about Tauri, their guideleines and the uselessness of them since people ignore them. Do they need some `cargo tauri optimize`? :) Here I can collect data from my GitHub issues and show to the Tauri team to convince them about some changes in their advertisment/ a default template profile or smth else
TODO: lto = true and lto = fat confusion: https://github.com/daemyn/dwarf-rs/issues/1#issuecomment-2624323227
TODO: "Is this necessary for client app?" https://github.com/mayocream/koharu/issues/1#issuecomment-2820146259

### Which LTO mode should be used by default?

TODO: talk here about Fat vs Thin LTO, pros and cons of each

Since we have more than one major LTO type, one of the first questions comes to a mind - which one should I use? Here we step into a semi-holywar topic. Short answer: *it depends* (as usual, huh).

My personal view on this question is the following. By default, **Fat LTO** should be used by applications since it results in better final optimizations (see the "Fat vs Thin LTO" section above). My logic here is pretty simple. Our aim with the Release build is to provide as much as possible optimized binary as we can achieve with other constraints (like build resources). Why? Because you know, "optimize once, run optimized everywhere" (on over 100500 billions of devices. hehe). It means for Release we enable only the most efficieint from the resulting optimizations LTO kind - it's Fat LTO.

Only if for some reason Fat LTO doesn't work in your case - too long build times even for the dedicated "Please enable all heavy optimizations to bless our users" build profile; you don't have enough RAM on your build machines and you cannot upgrade them; you met a bug only with Fat LTO (and this bug in the compiler since otherwise you need to fix your code/build scripts/build wrappers); you need to use distributed LTO (like Google) - only in this case use Thin LTO.

I know that many people can disagree with me. I heard in many discussions that ThinLTO should be used by default since it's faster to compile with Thin LTO than with Fat LTO, requires less RAM during the build, etc. Since all of these arguments are valid, I still prefer to deliver to users (or get if I am a user) as optimized binary as possible even if it requires more resources during the compilation. I value users experience more than CI experience, and I hope all of you think the same.

TODO: People is okay with switching from Fat to Thin LTO: https://github.com/hlsxx/tukai/issues/8#issuecomment-2577047640 + https://github.com/NikitaRevenco/patchy/issues/1 + https://github.com/roc-lang/roc/blob/cb762688de61db4de6f7d6061e6db352789ae708/Cargo.toml#L252 + https://github.com/alexpasmantier/television/issues/185 + https://github.com/alexpasmantier/television/pull/191
TODO: https://github.com/microsoft/wassette/pull/106/files#r2260669955 - Copilot and thin LTO :)))

### Documentation

What if we simply write more documentation about recommended compiler options like LTO? Sure, we can - as an example we have Tauri [documentation page](https://v2.tauri.app/concept/size/) about recommended Cargo switches for building Tauri-based apps. The problem is obvious - people don't read documentation! We won't discuss it in this article - let's just assume that this a sad true. Here we see a pattern - people didn't know about LTO (and other recommended options) before my issue but after pointing to the right piece of the documentation, they quickly copy&paste the recommended options into their projects. At least there was no need to explain in detail each option - people tend to believe to framework authors more than to a random stranger from GitHub since, with high chances, framework devs know more about their product (and recommended configuration for it) than me.

In ideal world, we can automate

TODO: Tauri recommendations includes LTO: https://v2.tauri.app/concept/size/
TODO: An example of "strong" min-sized profile for Rust: https://github.com/johnthagen/min-sized-rust/blob/main/Cargo.toml#L9 . Maybe we need to add a dedicated one for Cargo too?
TODO: Add some examples of Tauri apps without LTO from my issues
TODO: Even for quite popular apps (1k stars at the moment) LTO is not enabled even if we have instructions: https://github.com/braden-w/whispering/issues/574
TODO: Ratatui request - similar thing exists with Ratatui - https://github.com/ratatui/ratatui-website/issues/930
TODO: Bevy prerelease actions - Documentation about recommended steps with an engine before game release: https://github.com/bevyengine/bevy/issues/4586
TODO: https://deterministic.space/high-performance-rust.html - another site with performance recommendations for Rust
TODO:  https://github.com/the-lean-crate/criner?tab=readme-ov-file#the-lean-crate-initiative - "Lean crate initiative"
TODO: Even if docs exist - people dont' know about it: https://github.com/CyberTimon/RapidRAW/issues/47#issuecomment-3057878762
TODO: Sometimes you need to resurrect topics: https://github.com/zed-industries/zed/discussions/21450#discussioncomment-13609291
TODO: People forget to enable LTO: https://github.com/spirali/twinsong/issues/1#issuecomment-2674202278
TODO: That was an oversight - https://github.com/matze/splat/issues/2#issuecomment-2574987186

### Build systems

TODO: rephrase the part below since it was copied from the PGO article

In 99.9(9)% cases, we do not compile our applications via direct compiler invocations - we use build systems. What kind of LTO support could we expect from a build system? I have the following wishes:

* I want to have the possibility to enable LTO for my application via *a build system flag* rather than *a compiler flag*. Why? Because I can use multiple compilers, each compiler can have different flags for enabling LTO (e.g. check the difference in LTO flags between MSVC and Clang). Writing such logic for each compiler is not what I want to do for each project where I want to just use LTO.
* Optimizing with LTO whole dependency tree, not only my project. Almost any modern application uses dependencies. For making things even more complicated, these dependencies can be written in different languages, that are compiled by different compilers (with different LTO flags, as we know). In this case, enabling PGO build for the whole dependency tree is quickly becomes a non-trivial task.
* It must work fine for real world applications not just for single-file/single-dependency toys.

What do we have now in the ecosystem?

CMake has a [INTERPROCEDURAL_OPTIMIZATION](https://cmake.org/cmake/help/latest/prop_tgt/INTERPROCEDURAL_OPTIMIZATION.html#prop_tgt:INTERPROCEDURAL_OPTIMIZATION) switch for enabling LTO, and [CheckIPOSupported](https://cmake.org/cmake/help/git-stage/module/CheckIPOSupported.html) flag for checking the LTO support in a compiler since CMake supports multiple compilers. However, CMake doesn't enable LTO by default for any builtin profile (such Debug, Release, etc.). There are [discussions](https://gitlab.kitware.com/cmake/cmake/-/issues/17720) for enabling LTO on the CMake level for VS projects in the Release profile since VS does it but no discussions about enabling the same thing by default for other supporting LTO compilers.

More issue for CMake can be found on their GitLab by [this](https://gitlab.kitware.com/cmake/cmake/-/issues/?sort=created_date&state=opened&search=LTO&first_page_size=20) filter.

TODO: CMake by default uses ThinLTO (lol)
TODO: more LTO feature requests about LTO: https://gitlab.kitware.com/cmake/cmake/-/issues/26353
TODO: small issues with CMake settings and LTO: https://gitlab.kitware.com/cmake/cmake/-/issues/20546 + https://gitlab.kitware.com/cmake/cmake/-/issues/25202
TODO: LTCG by default discussions in CMake: https://gitlab.kitware.com/cmake/cmake/-/issues/17720 . Visual Studio enables GL and LTCG flags by default for the Release configuration according to the link above .
TODO: FIPS (a build system) enables LTO by default for VS projects - https://github.com/floooh/fips/blob/master/CHANGELOG.md?plain=1#L194
TODO: Ask different build systems about defaulting to LTO: CMake, Meson, Bazel, etc.
TODO: Wanna more insanity? What about cross-language LTO with a proprietary module? ;)
TODO: Cargo logic around LTO: https://github.com/rust-lang/cargo/blob/master/src/cargo/core/compiler/mod.rs#L1423 + https://github.com/rust-lang/cargo/blob/master/src/cargo/core/compiler/lto.rs ( + https://github.com/rust-lang/rust/blob/master/compiler/rustc_session/src/config.rs#L2441 in the Rustc compiler)

### External tooling

TODO: Reason for Thin LTO instead of Fat about cargo-dist defaults: https://opensource.axo.dev/cargo-dist/book/workspaces/simple-guide.html#the-dist-profile + https://github.com/axodotdev/cargo-dist/issues/118#issuecomment-1442132862

If we cannot modify a build system behavior directly, we can try to external tooling for that - in our case we can with tools enable LTO in project in a (semi-)automatic way.

I don't know about other ecosystems much but in Rust for Cargo it's [very popular](https://github.com/rust-unofficial/awesome-rust?tab=readme-ov-file#build-system) to write small extensions. Regarding tweaking compilation options we have several tools like [cargo-wizard](https://github.com/Kobzol/cargo-wizard) and [cargo-kit](https://github.com/trinhminhtriet/cargo-kit).

However, even if we have these helpers, and information about such tools is [published](https://www.reddit.com/r/rust/comments/1bbcdzs/cargowizard_configure_your_cargo_project_for_max/) on one of the most popular news channel for Rust - Rust's subreddit - we still see that almost no new Rust project use optimized settings. I see multiple reasons for that:

* People simply don't know about these tools. Not all engineers read Reddit (or any other information source like conferences) for various reasons: they are not interested enough, they don't have time, they don't have colleagues who read Reddit, a language barrier, etc. But they still are developers, and they are responsible for technical decisions here and there.
* They don't care about optimization too much, so convincing them to spend their time to figure out right tools, how to run them properly, etc. is a problem. And it's not only their fault since minds of these people can (and likely are) busy with other things - the simply don't have enough time to dig into their nuances.

So even if we have and advertize tools like `cargo-wizard` - it's **not enough for the whole community** to bring optimizations into a new level. I am not saying that these tools are useless - not at all! For people who already explicitly started to care about optimizing software such helpers can save time: instead of researching a zillion of existing options they can run a helper, click several times, and get a good improvement for their apps. I encourage you to develop similar things for other ecosystems!

### IDE level

One day an idea came to my mind. If most developers nowadays use different IDEs to work with a code (including creating new projects), what if we will enable LTO on the IDE level? IDEs usually some kind of "project templates", which include different project settings like compilation flags. We can try to enable LTO for Release profiles in such default project templates, and all new projects will get LTO automatically! So if we cannot change defaults on build systems level, we at least can try to tweak their defaults on an IDE level. In theory, it sounds at least partially as a clever idea.

So I made a quick research, and found that Visual Studio already [enables](https://gitlab.kitware.com/cmake/cmake/-/issues/17720) LTO (VS calls it "WholeProgramOptimization" but it's just a different name) by default for Release builds! I was surprised (in a good way) by this! Unfortunately, other IDEs are pretty conservative in this area, and they rely on build systems current default, without huge additions from IDE devs developers. I created several issues for popular C++ IDEs ([one](https://youtrack.jetbrains.com/issue/CPP-41883/Enable-Link-Time-Optimization-LTO-in-C-project-templates) for CLion, [another](https://bugreports.qt.io/browse/QTCREATORBUG-32330) for Qt Creator). And as you can expect, they don't care. If you are an IDE developer/product owner/any other decision-making person - please consider this option. However, I am almost 100% sure that nothing will be changed in this area.

### Distributed builds and LTO

For especially large applications for speeding-up the compilation process it's possible to use distributed builds. Distributed builds can be achieved with various tools like [DistCC](https://github.com/distcc/distcc), [IceCream](https://github.com/icecc/icecream) (looks like it's already dead), [FASTBuild](https://github.com/fastbuild/fastbuild), [Incredibuild](https://www.incredibuild.com/) or with built-in into the build systems capabilities like in [Bazel](https://bazel.build/basics/distributed-builds) or [Buck2](https://buck2.build/). Usually such systems are deployed in large companies where binaries tend to be large and there is a huge fleet of build machines. Personally, I never used distributed builds professionally but just for fun once I built a small build farm from 3 PCs at work because why not? :D

Here we are not interested in distributed builds itself - there are many materials in the Internet about this topic. We need to understand only a simple concept of how these builds work. We "simply" divide compilation process into N parts (like multiple translation units for C and C++ or multiple crates for Rust), where each part is remotely compiled on another build machine. Then all compiled parts are sent to one machine, where the final binary is linked.

Full LTO cannot be run in a distributed way since, as we already know, it cannot be parallelized. Thin LTO is a different thing since it's by design splits workload into N parts, so we can try to perform each part of ThinLTO on different build machines.

At the moment, I know only two compilers that support running Thin LTO in a distributed way: for C and C++ it's Clang ([blog](https://blog.llvm.org/2016/06/thinlto-scalable-and-incremental-lto.html)), for Rust it's Rustc([a forum discussion](https://internals.rust-lang.org/t/distributed-thinlto-support-in-rustc/22157)) (for Rust) - both are LLVM-based. As you see, even in the supported compilers this possibility is almost undocumented and hidden from "regular" compiler users - we definitely need here documentation improvements like minimally required [compiler versions](https://github.com/facebook/buck2/issues/323#issuecomment-1618952582) to use it. And this support [is not error-free](https://github.com/llvm/llvm-project/issues?q=is%3Aissue%20distributed%20lto%20) as well.

Despite compiler support, we also need to support distributed LTO in our **distributed** build systems since a build system needs to pass proper compiler switches to compiler invocations, perform final merging in a proper way, etc. I found that only Bazel and Buck2 support this scenario out of the box. I am not surprised since their major users - Google and Meta correspondingly - use extensively LTO in their optimization pipelines, and for their use cases they need distributed builds as well. Unfortunately, other distributed build systems doesn't support this scenario yet: DistCC [issue](https://github.com/distcc/distcc/issues/495), FASTbuild [issue](https://github.com/fastbuild/fastbuild/issues/1030) and [PR](https://github.com/fastbuild/fastbuild/pull/1054), SCCache [question](https://github.com/mozilla/sccache/discussions/2317).

If you are interested in Distributed LTO support for other programming languages/build systems/distributed build software - please check them on your own (and create an issue if the functionality is missing). If you have questions regarding distributed Thin LTO implementation in LLVM, don't hesitate to ping Teresa Johnson ;)

TODO: insert a link to Sony's slides about DTLTO efficiency - https://discourse.llvm.org/uploads/short-url/A2a3LQ12a2Wzy75ZY36OB2ueOeJ.pdf

## Changes in ecosystems at scale

TODO: Mention Google approach for applying fixes "at Google scale"

Google described their approach to this problem in their awesome "Software Engineering at Google" [book](https://abseil.io/resources/swe-book) - I highly recommend to read this book if you didn't it. Our question is described in this [chapter](https://abseil.io/resources/swe-book/html/ch22.html). TL;DR - all or almost all Large Scale Changes (LSC) are done centralized via tooling, not manually. The reason is simple: tooling scales good enough (especially if it's written with scale in mind), humans - worse. It works well in Google because they own as much as possible of their ecosystem: own VCS (monorepo btw - for good [reasons](https://abseil.io/resources/swe-book/html/ch16.html)), own tools, own code.

TODO: finish the part below

We can try to use a similar way but for open source ecosystems: create a tool for an ecosystem (like Crater in Rust), and push changes to open-source projects via this tool. But with the open-source use case, we have an interesting bunch of additional problems:

* Possible spam complaints.
* Lack of ownership.
* Dependency on the "performance" definition.

## Q&A

### Do I need to enable LTO for CI builds?

TODO: Disable LTO for CI stuff: https://github.com/roc-lang/roc/issues/1036#issuecomment-787009192

### When do I need to enable LTO?

TODO: People disable LTO on early project stages: https://github.com/the-lean-crate/criner/blob/a075e734dede8e1de5fe1652ec86f42da0162c41/Cargo.toml#L44

### Where do I need to enable LTO?

TODO: Enabling LTO earlier in the application delivery pipeline brings benefits for all downstream users of this application, including maintainers
TODO: Cargo defaults are for developers, not for users: https://github.com/YaLTeR/niri/discussions/968#discussioncomment-11818902 - and maintainers should be more responsible for these things
TODO: LTO enabled on CI level, not build scripts: https://github.com/PyO3/maturin/pull/2344 that leads to https://src.fedoraproject.org/rpms/maturin/blob/rawhide/f/maturin.spec - LTO won't be enabled on CI level in downstreams
TODO: People don't want to slowdown `cargo install` with LTO: https://github.com/rerun-io/rerun/issues/9339#issuecomment-2742767254
TODO: People don't care about LTO if they don't provide binaries: https://github.com/rust-fuzz/cargo-fuzz/issues/405#issuecomment-2751721972
TODO: People don't have time for LTO: https://github.com/nalgeon/sqlean/issues/137

### If you are so huge LTO fanboy, why your projects don't enable LTO?

TODO: reference by Rust projects like Telegram bots

TL;DR - I am too lazy for enabling these optimizations for unmaintained projects

### What about other optimizations like `codegen-units = 1`?

TODO: tell a story about similarity between LTO and CG1?
TODO: https://github.com/smallauncher/smallauncher/issues/1 - codegen-units = 1 small improvement
TODO: Small CG1 addition to existing LTO improves things too: https://github.com/andrewdavidmackenzie/pigg/issues/818 + https://github.com/andrewdavidmackenzie/pigg/pull/819 - build speed-up from CG1
TODO: https://github.com/fish-shell/fish-shell/commit/d9381d1ab60c3d428088be64dfb80e7f1a878e88 - people think that codegen-units = 1 isn't worth for some cases

## Stories

TODO: https://github.com/whitequark/superlinker/issues/4 - yet another spam reporter, lol

Here I collected some stories that I met during the short LTO journey. I found them interesting enough to share with you.

### Issue close policy

TODO: People closes issues berfore actually resolving them: https://github.com/NLnetLabs/rotonda/issues/118 + https://github.com/iwe-org/iwe/issues/8#issuecomment-2909541815

### Random-driven optimization flags

TODO: Strange optimization level changes in a C project: https://github.com/ravachol/kew/discussions/169

Sometimes I enable LTO not only for Rust apps but for other programming languages too. One day I found quite interesting project to play with - a terminal-based music player, [Kew](https://github.com/ravachol/kew). Since the project is written in C, I quickly jumped to the build scripts (a Makefile, in this case) and checked the optimization options. Of course, LTO was not enabled - that's fine - I can report it as usual. But the more interesting thing was that by default Kew was compiled with `-O1`. I know "-O2 vs -O3" battles but didn't see before using `-O1` by default.

Since I spent/wasted a couple of years as a C++ engineer, I was almost 100% sure - the reason for was some kind of error, highly likely it's a Undefined Behavior (UB), one of the sweetest toys for C and C++ devs. I quickly [raised](https://github.com/ravachol/kew/discussions/169) the question, and the author [confirmed](https://github.com/ravachol/kew/discussions/169#discussioncomment-10845724) it. Many devs still believe that it's not their fault - it's just a buggy compiler. Well, sometimes it's true but in 99.9(9)% cases the root cause lays somewhere in your code - and [sanitizers](https://github.com/google/sanitizers) are your best friends for such scenarios. I must admit that at my first paid job we did similar things for some projects - we disabled strict aliasing rules because otherwise our C++ codebase blown up in runtime randomly. Unfortunately, the responsible for these projects developers simply didn't know about sanitizers so they disabled optimizations and prayed to C++ Gods for forgiveness. By the way, for making prayers more powerful, we didn't update our C++ toolchains :)

Luckily enough, the Kew's authors are sane folks, and after a quick testing they enabled `-O2` + LTO (Fat LTO, more precisely). I hope later they will integrate builds with sanitizers into a CI system, and at least some bugs will be catched earlier. If you met a similar case with a 3rd party application or even with your own - please, start with investigating the actual problem with code. Disabling optimizations is not a robust solution since with any code change, compiler update, etc. your code has a chance to become broken even with a disabled optimization.

### Performance parity with other projects

After creating yet another LTO issue, I met an interesting [argument](https://github.com/Shnatsel/wondermagick/issues/5#issuecomment-2457866538) on why LTO shouldn't be enabled for a project. Since the project has an aim to replace ImageMagick, the project should be compiled with the same options as ImageMagick compiled in the wild.

Oh... I see multiple issues with this argument. ImageMagick and WonderMagick are written in different langs, so they use different compilers, even if we are talking about LLVM-based Clang and Rustc. These two compilers are different: they perform different optimizations under-the-hood (in some places Rust can optimize more, in other places - Clang), and you as a developer is out of control of all of these things. If we are talking about "how ImageMagick is compiled in the wild" I can assume that we still have GCC as a major compiler for C and C++ programs in major distributions. There are distributions that are trying moving to the LLVM-based ecosystem but for now it's mostly exception than a rule. So you are trying to compare GCC and LLVM-based Rustc - they differ **much** more, as you can guess :) Yep, in Rust we have `gcc-rs` but no one uses it seriosly at the moment. Maybe things will change later - who knows, personally I don't care much about this question. For making things more difficult: all distributions are using slightly different compiler flags. Some of them use more modern instruction set versions, some of them are not, etc. So about which exactly "wilderness" we are talking about?

Also, if we still want to use "almost the same" compiler options for performance comparisons, we can do a simple thing - just create a dedicated Cargo profile for comparisons with ImageMagick like `[profile.imagemagick-like]` where you can maintain as close as possible to ImageMagick options. But for the Release profile - the profile that is used by actual users - we can enable as much optimizations as possible. In this way, we will deliver the most optimized version to the users **and** will be able to compare our solution with the ImageMagick's baseline. That's it!

The second argument about non-important binary size nowadays is also isn't strong enough since I don't see a reason to have larger binary if we can have it smaller. Just extrapolate this way of thinking a bit further, and you will get pretty soon to the point "Electron-based apps are fine since storage is cheap-enough nowadays". I strongly believe that the Rust community should care about binary size too since this topic is still important for several domains like embedded (yeah, ImageMagick-like projects are used in such domains too) or network-constrained environments, where the Internet connection is too slow but you need to transfer a binary in any way.

By the way, I can agree with the third argument that LTO effects on performance **sometimes** can be unpredictable. Since I respect and like good technical arguments, I quickly made benchmarks and [didn't find](https://github.com/Shnatsel/wondermagick/issues/5#issuecomment-2457927232) performance improvements or regressions from enabling LTO for `wondermagick`. Unfortunately, only the binary size improvement wasn't enough to convince the author to enable LTO, and the issue was closed.

As a small conclusion. Don't expect from project's authors the most optimized versions of their programs - they can have various reasons to not enable them. If in your case some missing flags are important - change them, recompile and enjoy using a more optimized version of your favorite app.

### Enabling LTO to my respect

One day I routinely [created](https://github.com/GoldenStack/stupidfs/issues/1) yet another LTO-related issue in a project that I found interesting for some cybersec-related experiments. The author of the projected decided to perform a quick due diligence of my activity and kindly asked - why am I doing all of these LTO things. I assume (just an assumption!) that they initially thought that I am a simple possibly AI-driven spammer, hehe. Actually, that's completely fine - people want to know background of my activity, and I'm happy to share my incentives. However, later I got a bit disappointing [answer](https://github.com/GoldenStack/stupidfs/issues/1#issuecomment-2588496816) - the person enabled LTO and `codegen-units = 1` option due to "out of respect for your incredible dedication". I am happy to see that my work is recognized as valuable but please - don't enable things only due to a respect for someone. In this thread, I provided at least one such an argument - the binary size reduction, and I was ready to do performance benchmarks if it was required.

We are technical people here, and I highly encourage you making *technical* decisions based **only** on *technical* arguments - or at least try to do so. If for good technical reasons you cannot enable LTO but you value my work - please, don't enable LTO. It will make things only worse. I hope that in this case we just have a little misunderstanding, and the `stupidfs`'s author truly believes that enabling LTO brings positive value to users of the filesystem.

### Schrodinger's LTO

Few times I met a bit interesting pattern about resolving my LTO-related issues. People say "Yeah, thanks a lot for the suggestion!" and... close the issue (like [this](https://github.com/baehyunsol/ragit/issues/1#issuecomment-2475325970))! Sometimes LTO is not enabled at all, sometimes it's enabled but not committed (so an issue is closed **before** merging into a master/main branch).

I don't know the exact reason why people doing it - I suppose they simply forget about LTO (since forgetting non-important things is a common thing for humans) but please - if you are going to enable LTO, close an issue after merging LTO into your main branch. If you don't plan to enable LTO - just say it in the issue and then close the issue as "Not planned". It brings much more transparency for all of us. Thanks in advance!

### A DMCA abuser

TODO: write a story about DMCA abuser and it affected me
TODO: Remove cargo-kit mention due to potential DMCA abuser

## Other issues

TODO: people are not aware of LTO

## Current developments in the LTO area

TODO: New LTO modes: https://www.phoronix.com/news/NVIDIA-GCC-flto-locality
TODO: https://gitee.com/lyupaanastasia/llvm-adlt/tree/master - seems like LTO for 3rd party shared libs from Huawei

## Why do I do all of these?

TODO: describe motivation part
TODO: describe the way and background thoughts about LTO stuff
TODO: Started reporting build times too: https://github.com/mitsuhiko/systemfd/issues/33 , https://github.com/ddoemonn/ur-commit-mentor/issues/1 , https://github.com/rotmh/cargo-unify/issues/1 , https://github.com/BlankZhu/rustant-film/issues/3
