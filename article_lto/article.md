# Are we LTO yet?

Plot:

1) Intro
2) What is LTO - a few words with many links
3) LTO issues in: different programming languages, different build systems
4) Current LTO state in different programming languages, build systems, applications and libraries
5.0) Pieces of advice for enabling LTO in your project
5) Idea about increasing the default performance level for the industry and why Rust is a good place to start
6) My efforts to do so in: Rust, Python libs, IDEs and problems on my way
7) Different findings and stories during the "research"
8) Possible future ideas and afterwords

## What is LTO

TODO: LTO kinds: Fat and Thin, add links to the corresponding talks, papers, presentations, etc.

TODO: Fat LTO vs ThinLTO: https://discourse.llvm.org/t/clang-lld-thin-lto-footprint-and-run-time-performance-outperformed-by-gcc-ld/78997/6

* https://llvm.org/devmtg/2016-11/Slides/Amini-Johnson-ThinLTO.pdf - original slides
* https://convolv.es/guides/lto/ (+ comments here: https://news.ycombinator.com/item?id=38215535) - a good article about LTO
* Distributed and incremental ThinLTO - https://blog.llvm.org/2016/06/thinlto-scalable-and-incremental-lto.html
* ThinLTO CppCon video from Teresa: https://www.youtube.com/watch?v=p9nH2vZ2mNo

## LTO issues

TODO: Increased build time, memory consumption, compiler bugs, software bugs

### Increased build times

TODO: usually, it's 1.5x - 2x build time increase from Fat LTO. As an example: https://github.com/meilisearch/meilisearch/pull/5106#issue-2711334231 + https://github.com/martinvonz/jj/discussions/4463#discussioncomment-10673633
TODO: less deps you have - smaller overhead will be in practice
TODO: Projects are switching from Fat to Thin LTO due to build times: https://github.com/neon-mmd/websurfx/issues/516
TODO: ThinLTO also increases build time - https://github.com/facebook/buck2/blob/main/Cargo.toml#L416 - 50s to 84s with ThinLTO

### Increased memory consumption

TODO: Fyrox Full LTO - 17 Gib RAM in peak
TODO: Mention LTO, memory usage and RAM limits like https://docs.github.com/en/actions/using-github-hosted-runners/using-github-hosted-runners/about-github-hosted-runners
TODO: Funny scripts about memory calculation for parallel LTO jobs: https://github.com/swiftlang/swift/blob/main/utils/build_swift/build_swift/defaults.py#L87 + https://github.com/swiftlang/swift/blob/main/utils/build_swift/build_swift/defaults.py#L102

### Bugs

TODO: switching from Thin to Fat due to bugs on Windows: https://github.com/rust-lang/rust/issues/98302#issuecomment-1436474857

In Rustc I can quickly say about the following LTO-related bugs:

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

### People awareness

TODO: People still don't know about LTO in 2024: https://github.com/hyprwm/Hyprland/pull/5874#issuecomment-2094785202
TODO: Lack of confidence in LTO: https://github.com/mohanson/gameboy/issues/43#issuecomment-2403730081
TODO: Write about Tauri, their guideleines and the uselessness of them since people ignore them. Do they need some `cargo tauri optimize`? :) Here I can collect data from my GitHub issues and show to the Tauri team to convince them about some changes in their advertisment/ a default template profile or smth else

### Which LTO mode should be used by default?

TODO: talk here about Fat vs Thin LTO, pros and cons of each

### Documentation

TODO: Tauri recommendations includes LTO: https://v1.tauri.app/v1/guides/building/app-size/#rust-build-time-optimizations

### Build systems

TODO: LTCG by default discussions in CMake: https://gitlab.kitware.com/cmake/cmake/-/issues/17720 . Visual Studio enables GL and LTCG flags by default for the Release configuration according to the link above
TODO: Ask different build systems about defaulting to LTO: CMake, Meson, Bazel, etc.
TODO: https://github.com/distcc/distcc/issues/495 - distributed thin lto in distcc

### IDE level

TODO: CLion and LTO by default: https://youtrack.jetbrains.com/issue/CPP-41883/Enable-Link-Time-Optimization-LTO-in-C-project-templates - no one cares

## Funny stories

TODO: We don't enable LTO because of parity with other tool (WAT): https://github.com/Shnatsel/wondermagick/issues/5#issuecomment-2457866538
TODO: Strange optimization level changes in a C project: https://github.com/ravachol/kew/discussions/169

## Other issues

TODO: people is now aware of LTO


## Other links to read

TODO: https://gitee.com/lyupaanastasia/llvm-adlt/tree/master - seems like LTO for 3rd party shared libs from Huawei
