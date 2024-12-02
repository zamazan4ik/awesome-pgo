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

In Rustc I can quickly say about the following LTO-related bugs:

* Some projects [cannot](https://github.com/rust-lang/rust/issues/115344) be compiled when LTO and Profile-Guided Optimization (PGO) are used at the same time but if we enable only LTO or only PGO for such projects - everything works fine. Not a critical bug for now (since PGO sadly is not widely used nowadays across the industry) but anyway. Hopefully, this bug will be fixed [soon](https://github.com/rust-lang/rust/pull/133250).
* A strange [error](https://github.com/rust-lang/rust/issues/132677) when ThinLTO and [split-debuginfo = "packed"](https://doc.rust-lang.org/rustc/codegen-options/index.html#split-debuginfo) are enabled at the same time. Luckily, this problem can be mitigated by using Fat LTO or by disabling `split-debuginfo`. Since the problem is not widespread, I don't think it will be fixed in the near future.
* A strange error was found in Maturin when LTO was [enabled](https://github.com/PyO3/maturin/discussions/2337#discussioncomment-11420511) - a built for `ppc64le` platform started to fail with an `failed to get bitcode from object file for LTO (could not find requested section)` error. After a very quick investigation I found two similar issues in the upstream with the same error message - [one](https://github.com/rust-lang/rust/issues/129888), [two](https://github.com/rust-lang/rust/issues/94232). I don't know the root cause of the issue and I suppose the issue won't be fixed soon either.

More can be checked by [this](https://github.com/rust-lang/rust/issues?q=is%3Aissue%20state%3Aopen%20lto%20label%3AA-LTO) filter on GitHub.

### Decreased performance

TODO: https://github.com/rust-lang/rust/issues/47745 +

### Other

TODO: Split Dwarf and LTO limitations: https://github.com/CleverRaven/Cataclysm-DDA/pull/74067#issuecomment-2132435397 + https://gcc.gnu.org/bugzilla/show_bug.cgi?id=88389 + https://discourse.llvm.org/t/lto-thinlto-and-split-dwarf/70927

## Current LTO state in different ecosystems

TODO: Rust LTO state
TODO: OS distributions LTO state

## LTO for applications

## LTO for libraries

TODO: LTO for native libraries with bindings

## Difficulties with enabling LTO by default

TODO: documentation issues - https://github.com/rust-lang/rust/issues/48518 + at least in Rust people mess sometimes Thin Local LTO with Thin LTO (it's completely separate things)
TODO: C and C++ problems with UB
TODO: Rust problems with profile backward compatibility
TODO: LTO bugs in compilers - enumerate all of them here like Rustc bugs (LTO + PGO, LTO + split-debuginfo, other bugs)

## LTO by default

### Which LTO mode should be used by default?

TODO: talk here about Fat vs Thin LTO, pros and cons of each

### Documentation

TODO: Tauri recommendations includes LTO: https://v1.tauri.app/v1/guides/building/app-size/#rust-build-time-optimizations

### Build systems

TODO: Ask different build systems about defaulting to LTO: CMake, Meson, Bazel, etc.
TODO: https://github.com/distcc/distcc/issues/495 - distributed thin lto in distcc

### IDE level

TODO: CLion and LTO by default: https://youtrack.jetbrains.com/issue/CPP-41883/Enable-Link-Time-Optimization-LTO-in-C-project-templates - no one cares

## Funny stories

TODO: We don't enable LTO because of parity with other tool (WAT): https://github.com/Shnatsel/wondermagick/issues/5#issuecomment-2457866538

## Other issues

TODO: people is now aware of LTO


## Other links to read

TODO: https://gitee.com/lyupaanastasia/llvm-adlt/tree/master - seems like LTO for 3rd party shared libs from Huawei
