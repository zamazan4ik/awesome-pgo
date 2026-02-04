# TODO

Here we collect various TODOs about LTO to cover in the repository:

* LTO proposal for Swift: https://forums.swift.org/t/pitch-support-lto-for-swift/67379
* LTO statistics - extract and write about it in the articale: zamazan4ik/lto-statistics repo
* LTO example in cargo-* extensions: https://github.com/kbknapp/cargo-outdated/blob/master/Cargo.toml#L48
* Write a dedicated "Call for action" about using LTO for binaries: share thoughts, performance numbers, plans, suggestions, etc. Because of I'm tired with creating issues like "Enable LTO" on GitHub
* Cargo extensions don't usually enable LTO for some reason. One of them - increased "installation" time for users since they compile extensions locally on their machines
* Yet another Cargo profile with LTO - https://github.com/n0-computer/iroh/blob/main/Cargo.toml#L24
* https://github.com/FalkorDB/FalkorDB/issues/742 - not only me make requests about LTO
* Fat and ThinLTO difference resulting size: https://github.com/rustsec/rustsec/issues/1247#issuecomment-2356247000 (need to be rechecked) . I also got the same result with `jj` - https://github.com/martinvonz/jj/discussions/4463#discussioncomment-10673633
* GCC LTO: LTO and WHOPR - https://stackoverflow.com/questions/64954525/does-gcc-have-thin-lto
* Incremental PGO support across compilers: https://www.phoronix.com/news/GCC-Incremental-LTO-Patches . What about other compilers?
* SlateDB build time (db_bench): 25s without LTO vs 1 min with LTO (lto = true)
* LTO enabled only in external scripts: https://github.com/rust-lang/mdBook/blob/master/ci/make-release-asset.sh#L15
* Even found redundant release options: https://github.com/phiresky/ripgrep-all/issues/254#issuecomment-2365428914
* LTO and zeroize issue: https://github.com/TheQuantumPhysicist/thash/issues/1#issuecomment-2380663513
* Enable LTO for benchmarks too (related mainly for the Rust ecosystem) - discuss it in the article
* Different LTO modes for different build profiles: https://github.com/kellnr/kellnr/blob/main/Cargo.toml#L84
* ThinLTO by default in CachyOS: https://github.com/CachyOS/linux-cachyos/pull/304
* Different places for LTO to be enabled: in project build scripts, user machines, OS distributions, etc.
* Discussions about enabling LTO by default: https://github.com/rust-lang/cargo/issues/11298 (also there are some Rust Zulip threads) + https://github.com/rust-lang/cargo/issues/11298#issuecomment-1949953946
* Add about CMake LTO state - ask AlexFails about it
* Check other build systems for LTO switches
* Distributed build systems and LTO - GL HF
* https://gist.github.com/MaskRay/24f4e2eed208b9d8b0a3752575a665d4
* Some black voodoo thin lto magic about linking order: https://reviews.llvm.org/D130229
* Rustc and Thin Local LTO description: https://blog.rust-lang.org/inside-rust/2020/06/29/lto-improvements.html
* Check other langs like D for different LTO support: https://blog.rust-lang.org/inside-rust/2020/06/29/lto-improvements.html
* Fat LTO objects: https://github.com/llvm/llvm-project/issues/55431#issuecomment-1694769726 + https://stackoverflow.com/questions/72260320/clang-support-for-fat-lto-objects
* LTO for D lang: https://opensource.ebay.com/tsv-utils/docs/dlang-meetup-14dec2017.pdf
* Thin and Fat LTO combination - seems like not possible according to https://opensource.ebay.com/tsv-utils/docs/dlang-meetup-14dec2017.pdf
* Distributed Thin LTO for other LLVM-based compilers except Clang?
* Increases build time (for a small project, lol): https://github.com/torymur/sqlite-repr/issues/4#issuecomment-2391470436
* Some people enable even more optimizations after LTO report: https://github.com/markcda/r2proto3/issues/1
* Another disabled LTO due to compilation time concerns: https://github.com/cargo-bins/cargo-quickinstall/pull/122#issuecomment-1381560915
* C++ Wesnoth example with LTO: added to the scripts, not enabled by default: https://github.com/search?q=repo%3Awesnoth%2Fwesnoth+ENABLE_LTO&type=code + some issues with LTO: https://github.com/search?q=repo%3Awesnoth%2Fwesnoth+ENABLE_LTO&type=issues
* A crash from enabling LTO for a Swift program: https://github.com/swiftlang/swift/issues/67520
* Sometimes enabling LTO leads to lenghty discussions: https://github.com/hyprwm/Hyprland/pull/5874
* Difficult relationships between LTO and Qt: https://github.com/RPCS3/rpcs3/issues/12013#issuecomment-1126979045 (and people even implement this as a prevention: https://github.com/RPCS3/rpcs3/pull/12018)
* Some people are against "advanced" optimization options in their build scripts: https://github.com/qbittorrent/qBittorrent/pull/16813 since they can be enabled in the corresponding distro-specific recipes . But LTO can be enabled here: https://github.com/qbittorrent/qBittorrent/blob/master/.github/workflows/ci_windows.yaml
* Not only me enables LTO in their builds: https://github.com/aome510/spotify-player/issues/149#issue-1602209595
* Ask why LTO was disabled for the project: https://github.com/curlpipe/ox/commit/1a49d02af34414cceaa4ae27fd2c9eeda15b2fa2#diff-2e9d962a08321605940b5a657135052fbcef87b5e360662bb527c96d9a615542
* Another example of disabled LTO in a project: https://github.com/doukutsu-rs/doukutsu-rs/commit/ef1c2a59307ff4df500d38a88d5ac249daf31304
* Another good example of tweaking the compiler settings for better optimizations: https://github.com/foresterre/cargo-msrv/issues/505
* Not always developers tend to enable optimizations for all things: https://github.com/starkware-libs/cairo/issues/6471#issuecomment-2402154301
* Another example of LTO-related issue in a program. Result - just disable LTO without further investigation: https://github.com/metalbear-co/mirrord/issues/906
* Pacman update about LTO: https://github.com/wez/wezterm/issues/4987#issuecomment-1935829222
* Funny notes about broken LTO on different compilers: https://github.com/pop-os/xz-utils/blob/6509762f03584bf9e3ac9a42d0a8484cbb7ca926/ChangeLog#L3151
* An interesting issue about ThinLTO (but not with Fat) and incremental error: https://github.com/DioxusLabs/dioxus/issues/2874
* Find why LTO was disabled for Dioxus CLI: https://github.com/DioxusLabs/dioxus/commit/f2cd6b8af65fc4c8f1e9f109884a2e8696986f15
* LTO recommendation for building with in systemd: https://github.com/pop-os/systemd/blob/master_jammy/docs/DISTRO_PORTING.md?plain=1#L42
* Fedora defaults to Fat LTO (ELF + bitcode) with Clang: https://discussion.fedoraproject.org/t/f41-change-proposal-llvm-19-system-wide/118552
* LTO and Ring: https://github.com/briansmith/ring/issues/1444
* ThinLTO and lld bugs: https://github.com/rust-lang/rust/issues/84395
* LTO and JIT issues: https://github.com/jank-lang/jank/blob/main/compiler%2Bruntime/CMakeLists.txt#L19
* Some projects are not interested enough in LTO even after showing the results (without a PR - this is open-source!): https://github.com/transmission/transmission/issues/5776
* Suggest an idea to IDE to enable LTO by default in project templates (as it's already done in VS for MSVC - /GL + /LTCG is enabled by default for the Release profile)
* People create forks with enabled LTO: https://github.com/timvisee/ffsend/issues/173#issuecomment-2142185616
* Ask different IDEs about defaulting to enabled LTO in their project templates (like VS already does): CLion, Qt Creator at least
* https://github.com/whitequark/superlinker/issues/4#issuecomment-2440023708 - such a funny person!
* https://github.com/rust-lang/cargo/issues/14719 - enable LTO for Cargo
* Another article doesn't recommend LTO: https://tech.dreamleaves.org/trimming-down-a-rust-binary-in-half/
* Good example that even if you have documentation - people don't read it: https://github.com/Abdenasser/neohtop/issues/2 (no blame for people)
* Random people are happy from LTO performance boost: https://github.com/IncognitoBin/IncognitoBin/issues/8#issuecomment-2454837707
* LTO and transitive dependencies (and cross-language LTO)
* People badly learns from issues: https://github.com/evgenyigumnov/cblt/issues/5 and https://github.com/evgenyigumnov/rustsn/issues/53
* Disabled LTO due to some bug: https://github.com/pathwaycom/pathway/blob/main/external/timely-dataflow/Cargo.toml#L14
* Rust cannot override LTO settings for deps: https://doc.rust-lang.org/cargo/reference/profiles.html#overrides
* Size improvement is enough: https://github.com/cdump/evmole/issues/18#issuecomment-2510645056 and sometimes is not: https://github.com/sched-ext/scx/issues/1010#issuecomment-2511341123
* Not all people care about build times: https://github.com/PRQL/prql/issues/5031#issuecomment-2510283063
* Add LTO to default examples like https://github.com/zellij-org/rust-plugin-example
* Power of small optimizations: https://maksimkita.com/blog/power-of-small-optimizations.html
* LTO helps even to Rust projects with a small amount of deps: https://github.com/PiaCOS/treels/pull/2
* Exact build times, binary size and match it with other code metrics - all of that and much more can be done with Rust Crater . That's why I didn't perform detailed time measurements during the LTO experiment
* Some of the ideas can be taken by other people or in some way like GSoC from Rust Foundation
* LTO affects non-cached installations like https://github.com/LukeMathWalker/cargo-chef?tab=readme-ov-file#without-the-pre-built-image
* Discuss enabling LTO for prebuilt Cargo artifacts - it exists but I don't remember the exact name for this one right now
* What about creating an "build mode" for CI systems (like Jenkins/TeamCity plugins, GitHub actions for various build systems, etc.) that could enable on their level? Crazy idea but why not?
* Rust survey mechanism: https://github.com/rust-lang/surveys/blob/main/README.md
* Need to investigate further: LTO support by default in Zig: https://github.com/ziglang/zig/issues/2845
* Binary size in Rust paper: https://dl.acm.org/doi/pdf/10.1145/3519941.3535075
* Performance advisory database - report "slow" programs and send them a corresponding issue/PR
* Rustc non-LTO vs LTO metrics by perf.rust-lang.org - asked Kobzol about it
* Rustc perf improvement: https://rust-lang.github.io/rust-project-goals/2025h1/perf-improvements.html
* Rustc perf doesn't test continuosly cranelift backend
* LTO linker plugin comment in mold: https://github.com/rui314/mold/blob/main/src/lto-unix.cc#L1
* Issues like https://github.com/sponkurtus2/GemFetch/issues/1 are more educational than useful for a project in practice - https://github.com/sponkurtus2/GemFetch/issues/1#issuecomment-2548933054
* Discussions over LTO: https://gitlab.com/sequoia-pgp/sequoia-chameleon-gnupg/-/issues/73
* LTO disabled without known (at least from the Git history) reasons: https://github.com/search?q=repo%3Avlcn-io%2Fcr-sqlite+lto&type=commits - https://github.com/vlcn-io/cr-sqlite/discussions/446
* Enable LTO for Rust in Pacman: https://gitlab.archlinux.org/pacman/pacman/-/merge_requests/131
* Please fix the defaults: https://gitlab.com/sequoia-pgp/sequoia-chameleon-gnupg/-/issues/73#note_1932630360
* "Funny" bugs with LTO in Rustc + LLVM: https://github.com/rust-lang/rust/issues/110256
* LTO + CSIR PGO bug: https://github.com/llvm/llvm-project/issues/112103
* LTO issues in LLVM label: https://github.com/llvm/llvm-project/issues?q=is%3Aissue%20state%3Aopen%20%20label%3ALTO%20 (non-exhaustive)
* LTO issues in GCC: https://gcc.gnu.org/bugzilla/buglist.cgi?quicksearch=LTO&list_id=455896
* Disable LTO for archs with weak hardware: https://discussion.fedoraproject.org/t/fedora-riscv-should-we-disable-lto-gcc-already-disabled-for-clang/96582
* Enabled LTO for Python packages written in Rust: https://github.com/deliro/moka-py/blob/master/Cargo.toml#L16
* Reproducible builds site: https://reproducible-builds.org/
* Fat LTO vs Thin LTO vs codegen units: https://github.com/llMBQll/OmniLED/issues/2#issuecomment-2557879321
* https://github.com/pgcentralfoundation/pgrx/issues/1954#issuecomment-2564291777 - unfortunately, users don't know which flags they need to tweak and where
* Turned off LTO in a project: https://github.com/Astrabit-ST/Luminol/commit/d59e77f9044ba1e737c51bdbc4e71e141889c8a8
* Nice Cargo profiles for different cases: https://github.com/serpent-os/tools/blob/main/Cargo.toml#L67
* People forget to enable release things: https://github.com/ffimnsr/mk-rs/issues/1#issuecomment-2564574816
* Mention codegen-units option and experiments with that like https://github.com/kpouer/Maurice/issues/18
* LTO evangelizer, heh: https://github.com/emilsharkov/bukvalno/issues/1#issuecomment-2564913477
* Rust vs Zig from unsafe perspective: https://zackoverflow.dev/writing/unsafe-rust-vs-zig/ - mention it in the article from the "different languages point of view have different limitations"
* Rust, slow compilation and possible solutions: https://corrode.dev/blog/tips-for-faster-rust-compile-times/
* LTO was enabled, codegen-units are not (even with the provided benchmarks): https://github.com/emilsharkov/bukvalno/issues/1
* Compile time is an important topic for Rust: https://corrode.dev/blog/tips-for-faster-rust-compile-times/#tweak-codegen-options-and-compiler-flags
* Ohh... https://github.com/chimera-linux/cports/blob/master/main/llvm/template.py#L67 - strange possibly LTO-triggered errors
* Some people prefer a dedicated profile for LTO: https://github.com/TornaxO7/shady/issues/33#issuecomment-2569185762
* Some new projects already use optimized profiles: https://github.com/sponkurtus2/appInstalleR/blob/bdac86fbf26d6fecf3d20e6389a43e3719d5e29c/Cargo.toml#L18
* https://github.com/tedsteen/nes-bundler/blob/199ef528aea95f764927e75c7abafbc470310d24/Cargo.toml#L31 - "LTO is not worth it in RELEASE profile". Wut?
* Heavy optimizations (longer compile times) against fast prototyping thought
* Stripped binaries is frequently the reason for the binary size improvements - https://github.com/Automattic/harper/discussions/141#discussioncomment-11768520
* Some people prefer just integrating into a Release profile: https://github.com/EricLBuehler/diffusion-rs/issues/29#issuecomment-2576642637
* LTO progress in Gentoo: https://www.gentoo.org/news/2025/01/05/new-year.html
* https://github.com/GyulyVGC/sniffnet/issues/670#issuecomment-2577412007 - for some people it's not enough
* https://github.com/tomsjansons/ridi-router/issues/60#issuecomment-2593888324 - Release for dev purposes
* Small binary size and performance improvements are still welcomed: https://github.com/ranjeethmahankali/ftag/issues/27#issuecomment-2601113810
* Check later this PR: https://github.com/katanemo/archgw/pull/370
* Not supported things like https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-flive-patching - and reference GCC docs as a good source for other interesting LTO details
* Write about difference between Clang and GCC docs for LTO. As an example: unified LTO docs - https://www.phoronix.com/news/LLVM-Unified-LTO-Front-End + https://discourse.llvm.org/t/rfc-a-unified-lto-bitcode-frontend/61774 . Research the whole thread for more insights about different LTO modes
* "Seems like it suits most apps": https://github.com/rotmh/cargo-unify/issues/1#issuecomment-2602426458
* https://github.com/koto-lang/koto/discussions/405#discussioncomment-11914832
* Sometimes people enable LTO in quite surprising locations: https://github.com/umi-eng/adapter/blob/main/.cargo/config.toml
* PGO for HPC: https://www.phoronix.com/news/PGO-Optimizations-HPC-3p
* Balance between various factors in LTO: https://github.com/ikatson/rqbit/pull/283#issuecomment-2503508671 (+ LTO results - https://github.com/ikatson/rqbit/pull/283#issue-2687630404 )
* Fuzzing issues with LTO can happen too: https://github.com/rust-fuzz/cargo-fuzz/issues/384
* DTLTO for Linux kernel from Google: https://www.phoronix.com/news/Distributed-ThinkLTO-Linux-Kern
* Developers care about compilation time of their programs on user machines: https://github.com/iced-rs/comet/issues/3#issuecomment-2843784783
* Cross-lang LTO availability: https://github.com/gyscos/zstd-rs/blob/main/Cargo.toml#L47
* Big build time increase due to LTO but people is still ok with it: https://github.com/spiceai/spiceai/pull/5709#issuecomment-2851907092
* LTO as a dedicated profile: https://github.com/greenbone/openvas-scanner/issues/1922#issuecomment-2921989125
* Small improvements are still improvements, and people care about it: https://github.com/ReagentX/imessage-exporter/discussions/542#discussioncomment-13369335
* Rust compiler support: https://kobzol.github.io/rust/rustc/2025/06/09/why-doesnt-rust-care-more-about-compiler-performance.html
* LTO can be enabled even in such places: https://github.com/tombi-toml/tombi/blob/43eddc6a0e6c8662078e10cd83d02035913e7d92/xtask/src/command/dist.rs#L32
* External tooling limitation for custom profiles: https://github.com/project-robius/robrix/issues/482#issuecomment-3002291999 Do I need to create a feature request for custom profile support?
* People cannot add two simple lines with LTO: https://github.com/Migorithm/duva/issues/462#issuecomment-3016741784
* Even so hyped companies like OpenAI can miss optimization opportunities: https://github.com/openai/codex/issues/1411#issuecomment-3016099036
* LTO for things like malware :D https://www.techzine.eu/blogs/security/132626/memory-safe-malware-rust-challenges-security-researchers/
* https://github.com/greenbone/openvas-scanner/issues/1922#issuecomment-2921989125 - LTO as a dedicated profile
* "Serious profile" can have different names: https://github.com/afnanenayet/diffsitter/blob/main/Cargo.toml#L129
* Reputation can be a hard thing to fight with: https://github.com/axboe/liburing/discussions/1047#discussioncomment-8374809
* "Is this necessary for client app?" https://github.com/mayocream/koharu/issues/1#issuecomment-2820146259
* https://github.com/microsoft/wassette/pull/106/files#r2260669955 - Copilot and thin LTO :)))
* https://github.com/numaproj/numaflow/blob/53f8c182f42a7c34ce7133991e7764bb2eae5921/rust/Cargo.toml#L50 - another build time increase report (12s -> 133s just from enabling FatLTO)
* Ratatui docs PR regarding LTO and other stuff: https://github.com/ratatui/ratatui-website/pull/931
* Another wasn't aware of LTO option even if the documentation exists: https://github.com/gethopp/hopp/issues/35#issuecomment-3227066199
* Switching from ThinLTO to FatLTO is a great improvement: https://github.com/spinel-coop/rv/pull/54#issue-3359893040
* "Switch to ThinLTO since reduces build time for the same performance gains" - is it really true for the project? https://github.com/louis-e/arnis/commit/028ef4bd50e26e244e68f69844cf330930f26bfb
* Cross-language LTO: https://github.com/rui314/mold/issues/181 (mold discussion) + https://github.com/jeremy-rifkin/wyrm (GCC GIMPLE to LLVM IR converter) + https://users.rust-lang.org/t/compiling-rust-and-c-in-the-same-llvm-codegen-unit/121577/11?u=zamazan4ik (people is not happy about matching the exact versions) + https://github.com/RediSearch/RediSearch/pull/6815 (people is already interested in cross-lang LTO)
* Few questions regarding linker-based LTO in Rustc: https://users.rust-lang.org/t/questions-regarding-linker-plugin-based-lto/134070
* https://users.rust-lang.org/t/dramatic-increase-in-compile-time-with-fat-lto-in-release-build-causes-and-troubleshooting/99539 - full LTO is too slow. So many slowdowns everywhere... - https://github.com/rust-lang/rust/issues/88580
* Linker error if LLVM version mismatches: "Unknown attribute kind (103) (Producer: 'LLVM21.1.1-rust-1.92.0-nightly' Reader: 'LLVM 20.1.8')"
* LTO-related issues in Rustc: https://github.com/rust-lang/rust/issues?q=is%3Aissue%20state%3Aopen%20label%3AA-LTO
* LTO performance degradation in Rustc (it's really tricky one!) - an example: https://github.com/rust-lang/rust/issues/146497
* https://www.phoronix.com/forums/forum/phoronix/latest-phoronix-articles/1580347-wild-a-very-fast-linker-written-in-rust-aims-to-outperform-mold-linker?p=1580357#post1580357 - linker-based LTO is not widely used nowadays
* https://users.rust-lang.org/t/questions-regarding-linker-plugin-based-lto/134070 - a topic about linker-based LTO
* https://github.com/atuinsh/atuin/issues/326#issuecomment-1126701188 - other people also care about binary size improvements from LTO
* LTO is not worth its overhead: https://github.com/mierak/rmpc/pull/589#issuecomment-3223225690
* https://github.com/google/magika/pull/1244#discussion_r2524648140 - Gemini lies about LTO
* https://www.phoronix.com/news/Linux-Rust-LTO-Inline-Coming - difficulties with cross-lang LTO in the Linux kernel
* https://github.com/glide-wm/glide/issues/68#issuecomment-3736633131 - people afraid of LTO since it sometimes has (had) bugs
* https://blog.rust-lang.org/2025/12/19/what-do-people-love-about-rust/ - "performance availability" should include LTO too!
* https://gist.github.com/zamazan4ik/6be63330d2c97b510fdfc6b7aa7988c5?permalink_comment_id=5953818#gistcomment-5953818 - such a funny "LTO" comment :)
* LTO performance improvements with numbers - https://github.com/1jehuang/mermaid-rs-renderer/issues/13#issue-3854096676
* https://github.com/linuxmobile/oxicord/commit/410f26cbb3a315b3f5fb3051a8fc00437995feda#diff-2e9d962a08321605940b5a657135052fbcef87b5e360662bb527c96d9a615542 - why do people tune Release profile for faster builds? Is it for cargo install-only things or people just use Release builds during the development phase?
* cc-rs and LTO propagation - what is the state?
* LTO-uncovered bugs in Flang: https://github.com/llvm/llvm-project/pull/177944
* https://github.com/lucasgelfond/zerobrew/pull/34 + https://github.com/vyrti - great to see another Zaitsev who cares about LTO!
* Different people, different decisions like "debug" option for Release binaries - https://github.com/raphamorim/rio/blob/45c9297aecfd37337a2372e2cab74e3b6cf153b9/Cargo.toml#L85
* People doesn't use even self-documented optimization opportunities (vibe-coding, yep!) - https://github.com/rust-works/succinctly/blob/main/docs/optimizations/targets.md + https://github.com/rust-works/succinctly/blob/main/Cargo.toml
* https://x.com/avelinorun/status/2016110646362861707 - even tweeting about LTO-based PRs? lol :D
* https://github.com/uutils/coreutils/pull/9004#issuecomment-3795994445
* LTO performance regressions:
 - https://github.com/rust-lang/rust/issues/148670
* https://github.com/pier-cli/pier/pull/98 - sometimes it takes some time to merge a one-liner :)
* cross-lang full-lto bug in LLVM strikes once again - https://github.com/llvm/llvm-project/issues/57501#issuecomment-3844869663
* Some interesting stuff about LTO internals at LLVM - https://gist.github.com/MaskRay/24f4e2eed208b9d8b0a3752575a665d4#distributed-thinlto
* ThinLTO lost once again - https://github.com/block/goose/pull/6586#discussion_r2764048912
* tikv and LTO - https://github.com/tikv/tikv/issues/4163#issuecomment-475057061
