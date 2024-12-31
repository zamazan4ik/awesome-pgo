# Are we LTO yet?

Plot:

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
TODO: Fat LTO vs ThinLTO: https://discourse.llvm.org/t/clang-lld-thin-lto-footprint-and-run-time-performance-outperformed-by-gcc-ld/78997/6

* https://llvm.org/devmtg/2016-11/Slides/Amini-Johnson-ThinLTO.pdf - original slides
* https://convolv.es/guides/lto/ (+ comments here: https://news.ycombinator.com/item?id=38215535) - a good article about LTO
* Distributed and incremental ThinLTO - https://blog.llvm.org/2016/06/thinlto-scalable-and-incremental-lto.html
* ThinLTO CppCon video from Teresa: https://www.youtube.com/watch?v=p9nH2vZ2mNo
* LLVM docs about LTO: https://llvm.org/docs/LinkTimeOptimization.html
* LLVM LTO-related sources: https://github.com/llvm/llvm-project/tree/main/llvm/lib/LTO
* GCC docs about LTO internals: https://gcc.gnu.org/onlinedocs/gccint/LTO.html
* GCC LTO-related sources: https://github.com/gcc-mirror/gcc/tree/master/gcc/lto

## LTO issues

TODO: Increased build time, memory consumption, compiler bugs, software bugs

### Increased build times

TODO: usually, it's 1.5x - 2x build time increase from Fat LTO. As an example: https://github.com/meilisearch/meilisearch/pull/5106#issue-2711334231 + https://github.com/martinvonz/jj/discussions/4463#discussioncomment-10673633
TODO: less deps you have - smaller overhead will be in practice
TODO: Projects are switching from Fat to Thin LTO due to build times: https://github.com/neon-mmd/websurfx/issues/516
TODO: ThinLTO also increases build time - https://github.com/facebook/buck2/blob/main/Cargo.toml#L416 - 50s to 84s with ThinLTO
TODO: insert a meme about an electricity bill due to enabled LTO :D

### Increased memory consumption

TODO: Fyrox Full LTO - 17 Gib RAM in peak
TODO: Mention LTO, memory usage and RAM limits like https://docs.github.com/en/actions/using-github-hosted-runners/using-github-hosted-runners/about-github-hosted-runners
TODO: Funny scripts about memory calculation for parallel LTO jobs: https://github.com/swiftlang/swift/blob/main/utils/build_swift/build_swift/defaults.py#L87 + https://github.com/swiftlang/swift/blob/main/utils/build_swift/build_swift/defaults.py#L102

Since the compiler needs to perform some optimizations on **whole** program, it consequently needs to store many corresponding information in memory to work with it. It leads to the increased memory consumption. I don't know a good way to estimate how much of RAM LTO will take for a random project since it can depend on multiple factors like a compiler (different compilers have different LTO implementations), a compiler version (since LTO implementations can vary between commits/releases from the performance perspective too), etc.

Is there any difference between compilers from the memory perspective? I think they are but I didn't perform such measurements yet. The same applies to different compiler versions - the in theory is also "yes" but I suppose the chance of getting troubles from incresed memory consumption with LTO after the compiler upgrade are too low. Though, I didn't check it too.

### Bugs

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

### Decreased performance

TODO: A slowdown from LTO: https://github.com/rust-lang/rust/issues/48371 + https://github.com/rust-lang/rust/issues/47745

### Other

TODO: Split Dwarf and LTO limitations: https://github.com/CleverRaven/Cataclysm-DDA/pull/74067#issuecomment-2132435397 + https://gcc.gnu.org/bugzilla/show_bug.cgi?id=88389 + https://discourse.llvm.org/t/lto-thinlto-and-split-dwarf/70927

TODO: Cross-language LTO limitations: https://github.com/gyscos/zstd-rs/blob/main/Cargo.toml#L47

## Current LTO state in different ecosystems

TODO: Rust LTO state
TODO: OS distributions LTO state

### OS distributions



## LTO for applications

## LTO for libraries

TODO: LTO for native libraries with bindings

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

TODO: difficult committee structures that it needs additional explanations - https://www.reddit.com/r/cpp/comments/1h0ximn/comment/lz7j2rx/
TODO: a committee over standardization processes? https://www.incits.org/home/ was mentioned on the isocpp.org
TODO: Rust Binary size working group links to cover in the article: https://github.com/rust-lang/wg-binary-size/issues/3 + others
TODO: Cargo links: https://github.com/rust-lang/cargo/issues/784 + https://github.com/rust-lang/cargo/issues/4122 - lengthy discussions about changing the defaults. More can be found in the Rust Zulip
TODO: Rust ecosystem tooling: https://github.com/rust-lang/crater + https://perf.rust-lang.org/
TODO: Rust perfbot and binary sizes: https://github.com/rust-lang/rustc-perf/issues/1888 + https://github.com/rust-lang/rustc-perf/issues/145 + https://github.com/rust-lang/rustc-perf/pull/1772
TODO: Rust dev team has other priorities for various reasons: Rust main pain points in other places (since Rust is already "fast-enough by default")
TODO: The C++ ecosystem performance Question
TODO: I am too tired of continuous battling with people for free so don't expect much activity from me here
TODO: Does rustc-perf test Cranelift? Asked Bjorn about that, waiting for the answer . If not, I can create an issue for rustc-perf

### People awareness

TODO: People still don't know about LTO in 2024: https://github.com/hyprwm/Hyprland/pull/5874#issuecomment-2094785202
TODO: Lack of confidence in LTO: https://github.com/mohanson/gameboy/issues/43#issuecomment-2403730081
TODO: Write about Tauri, their guideleines and the uselessness of them since people ignore them. Do they need some `cargo tauri optimize`? :) Here I can collect data from my GitHub issues and show to the Tauri team to convince them about some changes in their advertisment/ a default template profile or smth else

### Which LTO mode should be used by default?

TODO: talk here about Fat vs Thin LTO, pros and cons of each

### Documentation

TODO: Tauri recommendations includes LTO: https://v1.tauri.app/v1/guides/building/app-size/#rust-build-time-optimizations


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

### IDE level

TODO: CLion and LTO by default: https://youtrack.jetbrains.com/issue/CPP-41883/Enable-Link-Time-Optimization-LTO-in-C-project-templates - no one cares

### Distributed build systems

TODO: https://github.com/distcc/distcc/issues/495 - distributed thin lto in distcc and https://github.com/fastbuild/fastbuild/issues/1030 for fastbuild

## Changes in ecosystems at scale

TODO: Mention Google approach for applying fixes "at Google scale"

Google described their approach to this problem in their awesome "Software Engineering at Google" [book](https://abseil.io/resources/swe-book) - I highly recommend to read this book if you didn't it. Our question is described in this [chapter](https://abseil.io/resources/swe-book/html/ch22.html). TL;DR - all or almost all Large Scale Changes (LSC) are done centralized via tooling, not manually. The reason is simple: tooling scales good enough (especially if it's written with scale in mind), humans - worse. It works well in Google because they own as much as possible of their ecosystem: own VCS (monorepo btw - for good [reasons](https://abseil.io/resources/swe-book/html/ch16.html)), own tools, own code.

TODO: finish the part below

We can try to use a similar way but for open source ecosystems: create a tool for an ecosystem (like Crater in Rust), and push changes to open-source projects via this tool. But with the open-source use case, we have an interesting bunch of additional problems:

* Possible spam complaints.
* Lack of ownership.
* Dependency on the "performance" definition.

## Funny stories

TODO: We don't enable LTO because of parity with other tool (WAT): https://github.com/Shnatsel/wondermagick/issues/5#issuecomment-2457866538
TODO: Strange optimization level changes in a C project: https://github.com/ravachol/kew/discussions/169

## Other issues

TODO: people is now aware of LTO

## Other links to read

TODO: https://gitee.com/lyupaanastasia/llvm-adlt/tree/master - seems like LTO for 3rd party shared libs from Huawei
