# TODO

Here we collect various TODOs about LTO to cover in the repository:

* LTO proposal for Swift: https://forums.swift.org/t/pitch-support-lto-for-swift/67379
* LTO statistics - extract and write about it in the articale: zamazan4ik/lto-statistics repo
* LTO example in cargo-* extensions: https://github.com/kbknapp/cargo-outdated/blob/master/Cargo.toml#L48
* Perform LTO tests for https://github.com/martinvonz/jj/discussions/4463#discussioncomment-10673044: Release build - 1m 33s, lto = true - 2m 41s, lto = thin - 1m 39s
* Write a dedicated "Call for action" about using LTO for binaries: share thoughts, performance numbers, plans, suggestions, etc. Because of I'm tired with creating issues like "Enable LTO" on GitHub
* Cargo extensions don't usually enable LTO for some reason. One of them - increased "installation" time for users since they compile extensions locally on their machines
* LTO increased build time for Buck: https://github.com/facebook/buck2/blob/main/Cargo.toml#L416 - 50s to 84s with ThinLTO
* Yet another Cargo profile with LTO - https://github.com/n0-computer/iroh/blob/main/Cargo.toml#L24
* https://github.com/FalkorDB/FalkorDB/issues/742 - not only me make requests about LTO
* Fat and ThinLTO difference resulting size: https://github.com/rustsec/rustsec/issues/1247#issuecomment-2356247000 (need to be rechecked) . I also got the same result with `jj` - https://github.com/martinvonz/jj/discussions/4463#discussioncomment-10673633
* Projects are switching from Fat to Thin LTO due to build times: https://github.com/neon-mmd/websurfx/issues/516
* A slowdown from LTO: https://github.com/rust-lang/rust/issues/48371 + https://github.com/rust-lang/rust/issues/47745
* GCC LTO: LTO and WHOPR - https://stackoverflow.com/questions/64954525/does-gcc-have-thin-lto
* LTO and the binary size for embedded: https://discourse.llvm.org/t/clang-lld-thin-lto-footprint-and-run-time-performance-outperformed-by-gcc-ld/78997
* https://llvm.org/devmtg/2016-11/Slides/Amini-Johnson-ThinLTO.pdf - original slides
* https://convolv.es/guides/lto/ (+ comments here: https://news.ycombinator.com/item?id=38215535) - a good article about LTO
* Incremental PGO support across compilers: https://www.phoronix.com/news/GCC-Incremental-LTO-Patches . What about other compilers?
* SlateDB build time (db_bench): 25s without LTO vs 1 min with LTO (lto = true)
* Cross-language LTO limitations: https://github.com/gyscos/zstd-rs/blob/main/Cargo.toml#L47
* LTO enabled only in external scripts: https://github.com/rust-lang/mdBook/blob/master/ci/make-release-asset.sh#L15
* Even found redundant release options: https://github.com/phiresky/ripgrep-all/issues/254#issuecomment-2365428914
* LTO and zeroize issue: https://github.com/TheQuantumPhysicist/thash/issues/1#issuecomment-2380663513
* Enable LTO for benchmarks too (related mainly for the Rust ecosystem) - discuss it in the article
* Different LTO modes for different build profiles: https://github.com/kellnr/kellnr/blob/main/Cargo.toml#L84
* ThinLTO by default in CachyOS: https://github.com/CachyOS/linux-cachyos/pull/304
* Interesting LTO surprises in Cargo: https://github.com/rust-lang/cargo/issues/14612 + https://github.com/rust-lang/cargo/issues/14575 + https://github.com/rust-lang/cargo/issues/9672 + https://github.com/rust-lang/cargo/issues/7491
* Is Cargo itself built with LTO?
* Different places for LTO to be enabled: in project build scripts, user machines, OS distributions, etc.
* Discussions about enabling LTO by default: https://github.com/rust-lang/cargo/issues/11298 (also there are some Rust Zulip threads) + https://github.com/rust-lang/cargo/issues/11298#issuecomment-1949953946
* Add about CMake LTO state - ask AlexFails about it
* Check other build systems for LTO switches
* https://github.com/distcc/distcc/issues/495 - distributed thin lto in distcc
* Distributed build systems and LTO - GL HF
* Distributed and incremental ThinLTO - https://blog.llvm.org/2016/06/thinlto-scalable-and-incremental-lto.html
* https://gist.github.com/MaskRay/24f4e2eed208b9d8b0a3752575a665d4
* ThinLTO CppCon video from Teresa: https://www.youtube.com/watch?v=p9nH2vZ2mNo
* Some black voodoo thin lto magic about linking order: https://reviews.llvm.org/D130229
* Rustc and Thin Local LTO description: https://blog.rust-lang.org/inside-rust/2020/06/29/lto-improvements.html
* Check other langs like D for different LTO support: https://blog.rust-lang.org/inside-rust/2020/06/29/lto-improvements.html
* Fat LTO objects: https://github.com/llvm/llvm-project/issues/55431#issuecomment-1694769726 + https://stackoverflow.com/questions/72260320/clang-support-for-fat-lto-objects
* LTO for D lang: https://opensource.ebay.com/tsv-utils/docs/dlang-meetup-14dec2017.pdf
* Thin and Fat LTO combination - seems like not possible according to https://opensource.ebay.com/tsv-utils/docs/dlang-meetup-14dec2017.pdf
* Distributed Thin LTO for other LLVM-based compilers except Clang?
* Increases build time (for a small project, lol): https://github.com/torymur/sqlite-repr/issues/4#issuecomment-2391470436
* Some people enable even more optimizations after LTO report: https://github.com/markcda/r2proto3/issues/1
* Strange optimization level changes in a C project: https://github.com/ravachol/kew/discussions/169
* Rustc + ThinLTO errors on Windows: https://github.com/rust-lang/rust/issues/98302#issuecomment-1436474857
* Another disabled LTO due to compilation time concerns: https://github.com/cargo-bins/cargo-quickinstall/pull/122#issuecomment-1381560915
* LTO sometimes reduces compilation time (@foxtran story)
* C++ Wesnoth example with LTO: added to the scripts, not enabled by default: https://github.com/search?q=repo%3Awesnoth%2Fwesnoth+ENABLE_LTO&type=code + some issues with LTO: https://github.com/search?q=repo%3Awesnoth%2Fwesnoth+ENABLE_LTO&type=issues
* Funny scripts about memory calculation for parallel LTO jobs: https://github.com/swiftlang/swift/blob/main/utils/build_swift/build_swift/defaults.py#L87 + https://github.com/swiftlang/swift/blob/main/utils/build_swift/build_swift/defaults.py#L102
* Write a note about too shitty system OOM for LTO and why systemd-oomd (and similar things) are important - at least no freezing system
* A crash from enabling LTO for a Swift program: https://github.com/swiftlang/swift/issues/67520
* Sometimes enabling LTO leads to lenghty discussions: https://github.com/hyprwm/Hyprland/pull/5874
* People still don't know about LTO in 2024: https://github.com/hyprwm/Hyprland/pull/5874#issuecomment-2094785202
* Difficult relationships between LTO and Qt: https://github.com/RPCS3/rpcs3/issues/12013#issuecomment-1126979045 (and people even implement this as a prevention: https://github.com/RPCS3/rpcs3/pull/12018)
* More build issues with LTO: https://github.com/FreeCAD/FreeCAD/issues/13173
* A good comment about LTO and hidden UBs: https://github.com/FreeCAD/FreeCAD/issues/6698#issuecomment-1977945753
* Duckstation offers LTO only in the README file: https://github.com/stenzek/duckstation?tab=readme-ov-file#building-1 and is enabled in all corresponding build scripts
* Typical uncovered errors with LTO: https://github.com/aseprite/aseprite/issues/4413#issue-2237029279
* Split Dwarf and LTO limitations: https://github.com/CleverRaven/Cataclysm-DDA/pull/74067#issuecomment-2132435397 + https://gcc.gnu.org/bugzilla/show_bug.cgi?id=88389 + https://discourse.llvm.org/t/lto-thinlto-and-split-dwarf/70927
* OpenRCT2 enables LTO only for certain build configs: https://github.com/search?q=repo%3AOpenRCT2%2FOpenRCT2%20IPO_ENABLED_BUILDS&type=code (and disables them in CI sometimes - https://github.com/OpenRCT2/OpenRCT2/pull/22695)
* Some people are against "advanced" optimization options in their build scripts: https://github.com/qbittorrent/qBittorrent/pull/16813 since they can be enabled in the corresponding distro-specific recipes . But LTO can be enabled here: https://github.com/qbittorrent/qBittorrent/blob/master/.github/workflows/ci_windows.yaml
* Performance improvement from LTO: https://github.com/netdata/netdata/issues/15338#issuecomment-1630287811
* Dylib-lto Cargo feature available only as unstable: https://doc.rust-lang.org/beta/unstable-book/compiler-flags/dylib-lto.html
* Tor saves 14% binary size for Arti (Tor)
* 10% speed improvements from ThinLTO: https://github.com/MaterializeInc/materialize/blob/main/Cargo.toml#L250
* Not only me enables LTO in their builds: https://github.com/aome510/spotify-player/issues/149#issue-1602209595
* Ask why LTO was disabled for the project: https://github.com/curlpipe/ox/commit/1a49d02af34414cceaa4ae27fd2c9eeda15b2fa2#diff-2e9d962a08321605940b5a657135052fbcef87b5e360662bb527c96d9a615542
* Another example of disabled LTO in a project: https://github.com/doukutsu-rs/doukutsu-rs/commit/ef1c2a59307ff4df500d38a88d5ac249daf31304
* Last checkpoint - https://github.com/espanso/espanso (included)
* Another good example of tweaking the compiler settings for better optimizations: https://github.com/foresterre/cargo-msrv/issues/505
* Another LTO issue that was not investigated and just switched off LTO for a project: https://github.com/redlib-org/redlib/issues/50#issuecomment-1939755101
* Not always developers tend to enable optimizations for all things: https://github.com/starkware-libs/cairo/issues/6471#issuecomment-2402154301
* Another example of LTO-related issue in a program. Result - just disable LTO without further investigation: https://github.com/metalbear-co/mirrord/issues/906
* https://github.com/qarmin/czkawka - an example when LTO is disabled in configs but is enabled (via sed hacks) in CI
* Suggest ThinLTO to Ruffle instead of disabling LTO at all: https://github.com/search?q=repo%3Aruffle-rs%2Fruffle+thinlto&type=code
* Pacman update about LTO: https://github.com/wez/wezterm/issues/4987#issuecomment-1935829222
* Tauri recommendations includes LTO: https://v1.tauri.app/v1/guides/building/app-size/#rust-build-time-optimizations
* Funny notes about broken LTO on different compilers: https://github.com/pop-os/xz-utils/blob/6509762f03584bf9e3ac9a42d0a8484cbb7ca926/ChangeLog#L3151
* Lack of confidence in LTO: https://github.com/mohanson/gameboy/issues/43#issuecomment-2403730081
* An interesting issue about ThinLTO (but not with Fat) and incremental error: https://github.com/DioxusLabs/dioxus/issues/2874
* Find why LTO was disabled for Dioxus CLI: https://github.com/DioxusLabs/dioxus/commit/f2cd6b8af65fc4c8f1e9f109884a2e8696986f15
* LTO recommendation for building with in systemd: https://github.com/pop-os/systemd/blob/master_jammy/docs/DISTRO_PORTING.md?plain=1#L42
* Fedora defaults to Fat LTO (ELF + bitcode) with Clang: https://discussion.fedoraproject.org/t/f41-change-proposal-llvm-19-system-wide/118552
* Add Kew project as a C-based project example that was fine with enabling LTO
* People don't enable LTO since the Cargo upstream doesn't enable it: https://github.com/PyO3/maturin/issues/1529
* LTO and Ring: https://github.com/briansmith/ring/issues/1444
* ThinLTO and lld bugs: https://github.com/rust-lang/rust/issues/84395
* Funny cross-language LTO debugging on Windows: https://users.rust-lang.org/t/cross-language-lto-doesnt-really-work-on-windows/118805
* LTCG by default discussions in CMake: https://gitlab.kitware.com/cmake/cmake/-/issues/17720 . Visual Studio enables GL and LTCG flags by default for the Release configuration according to the link above
* LTO and JIT issues: https://github.com/jank-lang/jank/blob/main/compiler%2Bruntime/CMakeLists.txt#L19
* Build with LTO in CI to catch LTO-triggered errors (especially true for C/C++ projects)
* Some projects are not interested enough in LTO even after showing the results (without a PR - this is open-source!): https://github.com/transmission/transmission/issues/5776
* Suggest an idea to IDE to enable LTO by default in project templates (as it's already done in VS for MSVC - /GL + /LTCG is enabled by default for the Release profile)
* People create forks with enabled LTO: https://github.com/timvisee/ffsend/issues/173#issuecomment-2142185616
* Ask different IDEs about defaulting to enabled LTO in their project templates (like VS already does): CLion, Qt Creator at least
* Ask different build systems about defaulting to LTO: CMake, Meson, Bazel, etc.
* https://github.com/whitequark/superlinker/issues/4 - yet another spam reporter, lol
* https://github.com/whitequark/superlinker/issues/4#issuecomment-2440023708 - such a funny person!
* https://github.com/rust-lang/cargo/issues/14719 - enable LTO for Cargo
* Another article doesn't recommend LTO: https://tech.dreamleaves.org/trimming-down-a-rust-binary-in-half/
* People still don't know about LTO: https://github.com/mr-adult/JFC/issues/2#issuecomment-2442816728 + https://github.com/supreetsingh10/lyricist/issues/1#issuecomment-2454878919 + https://github.com/agentic-labs/lsproxy/issues/61#issuecomment-2457848586 + https://github.com/MuongKimhong/BaCE/issues/1#issuecomment-2460528962 + https://github.com/theiskaa/mdp/issues/7#issuecomment-2467361983
* Good example that even if you have documentation - people don't read it: https://github.com/Abdenasser/neohtop/issues/2 (no blame for people)
* Not updated tools are not LTO optimized without changing defaults: https://github.com/Keats/dbmigrate (like this and maaaany others)
* Add conduwuit examples as a Rust project with many dedicated profiles: https://github.com/girlbossceo/conduwuit/blob/main/Cargo.toml#L537
* Random people are happy from LTO performance boost: https://github.com/IncognitoBin/IncognitoBin/issues/8#issuecomment-2454837707
* LTO and transitive dependencies (and cross-language LTO)
* We don't enable LTO because of parity with other tool (WAT): https://github.com/Shnatsel/wondermagick/issues/5#issuecomment-2457866538
* A trick for enabling LTO only for one subproject in Rust ecosystem: https://www.reddit.com/r/rust/comments/w4pzd2/rust_cargo_workspace_how_to_specify_different/
* Why I propose to use lto = true instead of lto = "fat" or lto = "thin". Fat vs Thin LTO vs an abstraction over them
* Write about Tauri, their guideleines and the uselessness of them since people ignore them. Do they need some `cargo tauri optimize`? :) Here I can collect data from my GitHub issues and show to the Tauri team to convince them about some changes in their advertisment/ a default template profile or smth else
* People badly learns from issues: https://github.com/evgenyigumnov/cblt/issues/5 and https://github.com/evgenyigumnov/rustsn/issues/53
* Disabled LTO due to some bug: https://github.com/pathwaycom/pathway/blob/main/external/timely-dataflow/Cargo.toml#L14
* Fyrox Full LTO - 17 Gib RAM in peak
* `debug = true` makes a huge impact on RAM usage with LTO
* Rust cannot override LTO settings for deps: https://doc.rust-lang.org/cargo/reference/profiles.html#overrides
* LTO enabled on CI level, not build scripts: https://github.com/PyO3/maturin/pull/2344 that leads to https://src.fedoraproject.org/rpms/maturin/blob/rawhide/f/maturin.spec - LTO won't be enabled on CI level in downstreams
* Mention LTO, memory usage and RAM limits like https://docs.github.com/en/actions/using-github-hosted-runners/using-github-hosted-runners/about-github-hosted-runners
* Size improvement is enough: https://github.com/cdump/evmole/issues/18#issuecomment-2510645056 and sometimes is not: https://github.com/sched-ext/scx/issues/1010#issuecomment-2511341123
* Not all people care about build times: https://github.com/PRQL/prql/issues/5031#issuecomment-2510283063
* LTO build issues: https://github.com/meilisearch/meilisearch/issues/2718
