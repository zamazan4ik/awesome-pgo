# Profile-Guided Optimization (PGO): a long hard road to the hell (or PGO: survey of industy-wide adoption, challenges and benefits as Amir suggested)

In this article, I want to discuss one compiler optimization technique - Profile-Guided Optimization (PGO). We will talk about PGO pros and cons, obvious and tricky traps with PGO, take a look at the current PGO ecosystem from different perspectives (programming languages, build systems, operating systems, etc.). I spent many days applying PGO to different applications... Tweaking build scripts, many measurements, sometimes tricky debug sessions, several compiler bugs and much more - it was (and still is) a great journey! I think my experience would be helpful for other people so you will be able to avoid my mistakes from the beginning and much nicer optimization experience.

The article is written in a story style. So the information about PGO is mixed with a lot of examples (because I like examples from the real world) and my own thoughts regarding multiple aspects PGO and PGO-related aspects.

The article is huge enough, so do not forget to grab some tea, beer or something like that before the begginning. Enjoy!

## Target audience

I think the article would be interesting for anyone, who cares about software performance. Especially it should be interesting for people who want to extract more performance from applications without tweaking the application itself.

Do not expect "hardcore" here. I am a usual software engineer, (un)fortunately not even a compiler engineer! And because of this I am sure my point of view could be interesting here - how looks like advanced (IMHO) compiler optimizations techniques from the average software engineer side.

## Disclaimer about the author

Most of my experience is with "native" technologies like C++ (it was my main lang for many years), C, and Rust. Right now I don't even writing a code for almost 2 years. They call me an architect but I prefer a "Confluence engineer".

So when we will talk about C/C++/Rust related things, you can expect some more advanced information. I try my best with covering other technologies with PGO like Go, C#, Java, etc. but do not expect much. However, at the end I put a bunch of helpful links for all technilogies.

Also, I am not a compiler engineer, so I cannot tell you much about internal implementation details. But I know people who knows about it a lot! I will try to reference all these awesome engineers too, and (hopefully) you will not be alone if you want to dig into the compiler internal details. Also, I left some anchors to PGO-related parts of different compilers for the most interested people!

## Attention anchor

Many of the technology-related people like numbers, especially when we are talking about performance. Since I want to get your attention to the topic of how PGO helps in real-life, I will show you at the very beginning some performance numbers with links to the actual benchmarks.

TODO: add the most famous applications and PGO results for it

`QPS` - Queries Per Second, `TPS` - Transactions Per Second, `RPS` - Requests Per Second, `EPS` - Events Per Second.

| Application | Performance improvement | Benchmark link |
|---|---|---|
| PostgreSQL | up to 15% faster queries | [GitHub](https://github.com/zamazan4ik/awesome-pgo/blob/main/postrgresql_results.md)  |
| SQLite | up to 20% faster queries | [Forum](https://sqlite.org/forum/forumpost/19870fae957d8c1a) |
| ClickHouse | up to +13% QPS | [GitHub](https://github.com/ClickHouse/ClickHouse/issues/44567#issuecomment-1589541199) |
| MySQL | up to +35% QPS | [one](https://www.oneapi.io/blog/tencent-gains-up-to-85-performance-boost-for-mysql-using-intel-oneapi-tools/), [two](https://bugs.mysql.com/bug.php?id=99781) |
| MariaDB | up to +23% TPS | [one](https://mariadb.com/files/MariaDBEnteprise-Profile-GuidedOptimization-20150401_0.pdf), [two](https://clearlinux.org/news-blogs/profile-guided-optimization-mariadb-benchmarks), [three](https://kristiannielsen.livejournal.com/17676.html)  |
| MongoDB | up to 2x faster queries | [GitHub](https://github.com/zamazan4ik/awesome-pgo/blob/main/mongodb.md) |
| Clang | up to +20% compilation speed | [one](https://llvm.org/docs/HowToBuildWithPGO.html#introduction), [two](https://planet.kde.org/lubos-lunak-2021-04-18-the-effect-of-cpu-link-time-lto-and-profile-guided-pgo-optimizations-on-the-compiler-itself/), [three](https://cristianadam.eu/20160104/speeding-up-libclang-on-windows/) |
| GCC | up to +7-12% compilation speed | [one](https://bugs.archlinux.org/task/56856), [two](https://github.com/NixOS/nixpkgs/pull/112928#issuecomment-778508138) |
| Rustc | up to +10-15% compilation speed | [one](https://blog.rust-lang.org/inside-rust/2020/11/11/exploring-pgo-for-the-rust-compiler.html), [two](https://kobzol.github.io/rust/rustc/2022/10/27/speeding-rustc-without-changing-its-code.html) |
| CPython | +13% improvement | [Blog](https://www.activestate.com/blog/python-performance-boost-using-profile-guided-optimization/) |
| DMD (D compiler) | up to +45% improvement | [GitHub issue](https://github.com/dlang/dmd/pull/13791#issue-1164476335) |
| LDC (D compiler) | up to 10-15% improvement | [GitHub comment](https://github.com/ldc-developers/ldc/discussions/4524#discussioncomment-7537608) |
| Chromium | up to 12% faster | [one](https://blog.chromium.org/2016/10/making-chrome-on-windows-faster-with-pgo.html), [two](https://blog.chromium.org/2020/08/chrome-just-got-faster-with-profile.html) |
| Firefox | up to 12% faster | [Google groups](https://groups.google.com/g/mozilla.dev.platform/c/wwO48xXFx0A/m/ztg4i0DYAAAJ) |
| Envoy | up to +20% RPS | [GitHub comment](https://github.com/envoyproxy/envoy/issues/25500#issuecomment-1724584679) |
| HAProxy | up to +5% RPS | [GitHub issue](https://github.com/haproxy/haproxy/issues/2047) |
| Vector | up to +15% EPS | [GitHub issue](https://github.com/vectordotdev/vector/issues/15631) |

Of course, it's not a complete list - much more PGO showcases you can check right now [here](https://github.com/zamazan4ik/awesome-pgo#pgo-showcases). If you are interested - let's go our PGO journey!

## Intro

I like performant applications. I like when software doesn't make me wait for results for a long time. Faster software compilation, faster OS boot time, faster `grep`-ing over hundreds gibibytes of logs.

There are multiple where performance can be useful for you too:

* Infrastrusture cost reduction (could be valuable at large scales)
* Improving CI speed (faster Time-To-Market, cheaper infrastructure, better developer experience with build pipelines)
* Better developer experience during the local development (quicker write-evaluate loop)
* Better energy efficiency (your smartphone will say "Thank you")
* Limited hardware availability (wrong CAPEX, no budget for the new hardware or even no possibility to buy new hardware due to multiple other reasons)

Even if we are talking about "large" systems, having better efficiency per core could be important since in this case you need to implement horizontal scaling for your app later and postpone investing your time into things like sharding, load balancing and other cool stuff.

As a usual human, I have a (de)buff - I am lazy. That's why I don't like to perform manual optimizations. Profiling, finding a bottle-neck, trying to figure out what is going on in this bottleneck, thinking and implementing an optimization - too much things to do for a lazy engineer. I need something easier to achieve better performance, ideally without me at all.

Performing optimizations is one of the most important tasks for modern optimizing compilers. So we need a simple thing - tweak our compilers to perform more and more agressive optimizations. In this case, we can postpone the moment, when the compiler will need a help from a human. That's why at university (several years ago) I decided to invest my time into the compiler research project.

During the investigation, I found multiple very interesting compiler techniques to optimize programs. And one of these things was PGO.

## What is PGO?

Let's chat about PGO with the truly expert in everything - Gigglesome Punster Texterator aka GPT.

**Me**: What is PGO?

**Expert**: PGO could refer to a few different things depending on the context. In the world of gaming, PGO might stand for "Pokémon GO," a popular augmented reality mobile game. In a programming context, it could refer to "Profile-Guided Optimization," a technique used to improve the performance of compiled code. Can you provide more details or specify the context you're referring to?

**Me**: *khm-khm* What is Profile-Guided Optimization?

**Expert**: Profile-Guided Optimization (PGO) is a compiler optimization technique used in software development to improve the performance of compiled code. The optimization process is guided by information gathered from the profiling of the program's behavior during specific executions.

Here's a general overview of how Profile-Guided Optimization works:

1. **Profiling Phase:** The program is first compiled and executed with a profiling instrument enabled. During this phase, the compiler collects data on the frequency and distribution of different code paths, function calls, and other program behaviors.

2. **Profile Data:** The profiling phase generates a profile data file that contains information about the hotspots in the code—sections that are executed frequently—and other relevant details about the program's runtime behavior.

3. **Recompilation:** The compiler then uses this profile data to guide the optimization process. It can make informed decisions about which code paths are most critical and where optimizations are likely to have the greatest impact.

4. **Optimization:** Based on the profile information, the compiler can apply various optimizations such as inlining functions, reordering code for better branch prediction, and optimizing loop structures. These optimizations aim to improve the overall performance of the program.

5. **Rebuilding:** The code is recompiled with the optimizations applied, resulting in a new executable that should run more efficiently based on the insights gained from the profiling data.

Profile-Guided Optimization is particularly effective in scenarios where the program's execution behavior varies across different input data or usage patterns. It helps the compiler tailor optimizations to the specific needs of the application, leading to better overall performance.

Keep in mind that the effectiveness of PGO may depend on the quality and representativeness of the profiling data collected during the initial execution phase.

**Me**: Thanks buddy!

Sounds quite simple, doesn't it? I was young, so such a simple explanation (even not from non-existing at that moment Gigglesome Punster Texterator) bought me, and I decided to dig a little bit deeper. If according to the multiple articles PGO helps with optimizing software, why we don't use it everywhere right now?

Let's dig into PGO further!

TODO: write a note about PGO answer from ChatGPT about varying work profiles and how bad PGO works in this case.

### PGO and other similar names

The first funny thing that I found - PGO has multiple names! And they all mean **completely** the same thing:

* Profile-Guided Optimization (PGO)
* Feedback-Directed Optimization (FDO)
* Profile-Driven Optimization (PDO)
* Profile-Directed Feedback (PDF) (IBM AIX [documentation](https://www.ibm.com/docs/en/aix/7.3?topic=techniques-profile-directed-feedback))
* Profile Online Guided Optimization (POGO) (I found at least one [post](https://www.reddit.com/r/rust/comments/jwad2g/new_experiment_pogo_online_pgo_for_functions/) on Reddit)

At most places, you will see PGO (sometimes people pronouns it as "pogo") (like in Clang (TODO ref), GCC (TODO ref), Rustc (TODO ref)). Sometimes, FDO is used (like in Bazel (TODO ref)). Other names almost never used. In this article, I use only PGO abbreviation as the most known name for this optimization technique.

Quickly I found another issue: there are multiple meanings for PGO:

* [PostrgreSQL Operator](https://github.com/CrunchyData/postgres-operator). This one is the most annoying since Google shows it often (at least in my queries). I recommend to resolve it with excluding "postgresql" word from the results.
* PGO [company](https://pl.wikipedia.org/wiki/PGO_(polskie_przedsi%C4%99biorstwo)). I do not think they are Profile-Guided Optimization guru but who knows...
* Pokemon GO (as The Expert said above)

Not critical at all but keep in mind this information during your googling sessions.

### PGO types

PGO has two major types:

* Instrumentation-based
* Sampling-based

To make the things even more complicated - there are a bunch of hybrid approaches as well. We will check them a bit later. Right now let's take a deeper look on the major ones.

#### PGO via instrumentation

Right now, instrumentation mode is the de-facto default PGO mode in many cases. So if you here somewhere something about PGO, in 99% it is instrumentation PGO. But how does it work?

Usually, the scenario is the following:

* Compile your application in instrumentation mode
* Run the in

#### PGO via Sampling

TODO: add info about Sampling PGO

More about AutoFDO you can find here:

* Original AutoFDO paper: [Google research](https://research.google/pubs/pub45290/)

TODO:

* Add notes about system-wide PGO:
  - Google approach with AutoFDO: run `perf` on a subset of nodes, collect profiles for all running applications and use these profiles on CI
  - Ad-hoc approach: use AutoFDO approach from Google or "just" rebuild all packages with Instrumentation PGO, run on a real-life workload, then collect the profiles and use them during the optimized build

#### Other PGO types

TODO: Write here about other PGO types from https://aaupov.github.io/blog/2023/07/09/pgo

---

## PGO caveats

I have read **many** articles about PGO. Unfortunately, I found almost nothing about PGO issues and difficulties in different situations. So I just collected as much as possible traps myself and want to share my experience (not traps) with you. Niels Bohr once said: “An expert is a person who has made all the mistakes that can be made in a very narrow field.”. Am I a PGO expert now, huh?

TODO: insert here a meme with Garold with pain (about all my PGO mistakes)

### Instrumentation PGO problems

* Instrumentation PGO: problems

  - The instrumented binary is slower. How much - usually is unpredictable
  - Double compilation - it hurts a lot smaller platforms like PowerPC in different CI pipelines
  - Increased compilation resource usage during the compilation - increased memory usage by the compiler and compile time
  - Increased binary size
  - Exit-code issues - for some applications in the end the profile is not dumped (like Clangd). You need to tweak the application manually
  - Not all software can be built with PGO (Envoy) - some dependencies does not support PGO for some reason

#### Instrumented binary is slower

Instrumented binary is slower. But how much? Well, as usual - *it depends*. I didn't find before such benchmarks for real-life applications so I did them and ready to show you some numbers for several projects:

TODO: add as many projects as I can

| Application | Instrumentation to Release slowdown ratio | Benchmark |
|---|---|---|
| HAProxy (Clang) | 1.10x - 1.20x | links ([one](https://github.com/haproxy/haproxy/issues/2047#issuecomment-1729971279), [two](https://github.com/haproxy/haproxy/issues/2047#issuecomment-1728265165)) |
| HAProxy (GCC) | 1.24x | links ([one](https://github.com/haproxy/haproxy/issues/2047#issuecomment-1729971279), [two](https://github.com/haproxy/haproxy/issues/2047#issuecomment-1729606775)) |
| Fluent-Bit | 1.48x | [link](https://github.com/fluent/fluent-bit/discussions/6638#discussioncomment-6419880) |
| ClickHouse | up to **311x** (it's not a mistake!) | [link](https://github.com/ClickHouse/ClickHouse/issues/44567#issuecomment-1519136060) |
| Tarantool | up to 1.5x | [link](https://github.com/tarantool/tarantool/issues/8089#issuecomment-1580628168) |
| Lychee | 4.0x | [link](https://github.com/lycheeverse/lychee/issues/1247) |
| grcov | up to 10x | [link](https://github.com/mozilla/grcov/issues/1128#issue-1956099937) |
| quilkin | 1.5x | [link](https://github.com/googleforgames/quilkin/issues/834#issuecomment-1772719309) |
| frawk | 1.35x | [link](https://github.com/ezrosent/frawk/issues/103#issuecomment-1751781163) |
| sd | 1.8x | [link](https://github.com/chmln/sd/issues/237#issue-1931408669) |
| youki | 2.0x | [link](https://github.com/containers/youki/issues/2386#issue-1911324530) |
| broot | 2.37x | [link](https://github.com/Canop/broot/issues/741#issue-1895569419) |
| delta | 1.42x | [link](https://github.com/dandavison/delta/issues/1540#issue-1891473319) |
| Cemu | 2.0x | [link](https://github.com/cemu-project/Cemu/issues/797#issuecomment-1521169155) |
| httpd | 1.04x | [link](https://github.com/zamazan4ik/awesome-pgo/blob/main/httpd.md) |
| databend | 151.0x | [link](https://github.com/datafuselabs/databend/issues/9387#issuecomment-1566210063) |
| PostgreSQL | TODO | [link](https://github.com/zamazan4ik/awesome-pgo/blob/main/postrgresql_results.md) |
| clang-tidy | 2.28x | [link](https://github.com/llvm/llvm-project/issues/63486#issuecomment-1606147035) |
| uncrustify | 1.62x | [link](https://github.com/uncrustify/uncrustify/issues/4045#issue-1775843009) |
| typos | 2.61x | [link](https://github.com/crate-ci/typos/issues/827#issue-1888263250) |
| tfcompile | 1.43x | [link](https://github.com/tensorflow/tensorflow/issues/60944#issuecomment-1637143591) |
| drill | 1.72x | [link](https://github.com/fcsonline/drill/issues/185#issue-1795772245) |
| Vector | 12-14x | [link](https://github.com/vectordotdev/vector/issues/15631#issuecomment-1694554798) |
| lld | ~6.8x | [link](https://github.com/llvm/llvm-project/issues/63486#issuecomment-1607953028) |

The same applies for libraries as well:

| Library | Instrumentation to Release slowdown ratio | Benchmark |
|---|---|---|
| tantivy | 1.49x | [link](https://github.com/quickwit-oss/tantivy/issues/2163#issue-1872682955) |
| tonic | up to 1.55x | [link](https://github.com/hyperium/tonic/issues/1486#issue-1871968898) |
| quick-xml | up to 2.5x| [link](https://github.com/tafia/quick-xml/issues/632#issue-1852200475) |
| xml-rs | 1.45x | [link](https://github.com/netvl/xml-rs/issues/228#issue-1852007193) |
| tquic | up to 1.3x | [link](https://github.com/Tencent/tquic/issues/19#issue-1972443474) |
| lingua-rs | up to 146x (not an error!) | [link](https://github.com/pemistahl/lingua-rs/discussions/273#discussion-5864708) |

I could prepare more advanced analysis like slowdown p50/p95/p99 percentiles, testing across more different configurations (like trying to replicate tests across different hardware/software, etc) but I am a bit lazy for doing it right now. Maybe next time ;)

TODO: write about cases when an instrumented binary is faster (I met such cases in some strange situations. Probably some combination of compiler optimizations does this - who knows, didn't investigate deep such cases).

#### Instrumented binary is larger

Instrumentation PGO works via inserting into your code some counters for tracking the program execution characteriscs, so your binary will be larger after the instrumentation. In assembler it looks something like:

TODO: add before and after PGO instrumentation assembler for Clang and GCC: https://godbolt.org/z/ofMKzEscn

How much binary space does it take in practice? Let's check it (all tests are done on the Linux machine):

| Application | Release size | Instrumented size | PGO optimized size | Instrumented to Release size ratio | Compiler |
|---|---|---|---|---|---|
| ClickHouse | 2.0 Gib | 2.8 Gib | 2.0 Gib | 1.4x | Clang |
| MongoDB | 151 Mib | 255 Mib | 143 Mib | 1.69x | Clang |
| SQLite | 1.8 Mib | 2.7 Mib | 1.5 Mib | 1.5x | Clang |
| HAProxy | 13 Mib | 17 Mib | 13 Mib | 1.3x | Clang |
| HAProxy | 15 Mib | 19 Mib | 16 Mib | 1.27x | GCC |
| Nginx | 3.8 Mib | 4.3 Mib | 3.8 Mib | 1.13x | Clang |
| Envoy | 439 Mib | 1800 Mib | 433 Mib | 4.15x | Clang |
| httpd | 2.3 Mib | 2.7 Mib | 2.4 Mib | 1.17x | Clang |
| Quilkin | 33 Mib | 94 Mib | 30 Mib | 2.84x | Rustc |
| Rathole | 3.8 Mib | 11 Mib | 3.5 Mib | 2.89x | Rustc |
| bat | 5.4 Mib | 16 Mib | 5.2 Mib | 2.96x | Rustc |
| Catboost | 32 Mib | 88 Mib | 30 Mib | 2.75x | Clang |
| curl | 1.1 Mib | 1.4 Mib | 939 Kib | 1.27x | Clang |
| czkawka | 14 Mib | 38 Mib | 14 Mib | 2.71x | Rustc |
| Fluent-Bit | 7.9 Mib | 11 Mib | 6.4 Mib | 1.39x | Clang |
| Vector | 198 Mib | 286 Mib | 124 Mib | 1.44x | Rustc |
| htmlq | 6.7 Mib | 11 Mib | 6.8 Mib | 1.64x | Rustc |
| ouch | 3.5 Mib | 8.0 Mib | 3.3 Mib | 2.26x | Rustc |
| difftastic | 68 Mib | 75 Mib | 68 Mib | 1.10x | Rustc |
| slint-fmt | 3.1 Mib | 28 Mib | 3.3 Mib | 9.0x | Rustc |
| tokei | 7.5 Mib | 16 Mib | 7.5 Mib | 2.13x | Rustc |
| tealdeer | 7.4 Mib | 20 Mib | 7.6 Mib | 2.7x | Rustc |
| qsv | 42 Mib | 81 Mib | 39 Mib | 1.93x | Rustc |
| vtracer | 6.6 Mib | 13 Mib | 6.4 Mib | 1.97x | Rustc |

In general, there is no way to make a *precise* predict of how large your binary will be after the instrumentation without the actual compilation process. There are so many variables involved in this process (how much branches your application has, do you recompile with PGO your statically-linked dependencies, etc.) that much-much easier will be just recompile with instrumentation and check it. Maybe one day the compilers (or an ecosystem around the compiler) will provide you some estimations before the actual compilation process but not today.

PGO optimized size hugely depends on your test workflow. If you execute larger amount of code, with a higher probability the inlining will be triggered during the optimization, and your binary due to this will be larger.

TODO: add a note about non-stripped binaries

#### Build issues with instrumentation PGO

TODO: add Envoy example

### Sampling PGO problems

TODO

### Common PGO problems

TODO: like building with PGO in multi-lang apps and passing PGO info to the dependencies.

## PGO support in compilers

Different compilers has different PGO maturity levels. Some of them support PGO for decades, some of them added PGO just recently, and some (shame on you) has no PGO support at all! Below I prepared some overview of the PGO state across different compilers for multiple programming languages.

### C and C++

C has a veeeery long history, C++ is a bit younger but still is quite a mature technology. So compilers for C and C++ also evolved **a lot** during the decades from many viewpoints, including multiple optimizations.

TODO: add information when PGO first appeared in C compilers

Here are some PGO integration examples into the different compilers:

* [GCC](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html#Instrumentation-Options)
* [Clang](https://clang.llvm.org/docs/UsersManual.html#profile-guided-optimization)
* [MSVC](https://learn.microsoft.com/en-us/cpp/build/profile-guided-optimizations)
* [ICC](https://www.intel.com/content/www/us/en/docs/cpp-compiler/developer-guide-reference/2021-8/profile-guided-optimization-pgo.html)
* [AOCC](https://www.amd.com/en/developer/aocc.html#documentation) (supports but the documentation right now exists only as PDF files)

### Rust

Rust right now has one major compiler - `rustc`. Rustc is based on LLVM so PGO implementation shares almost all details with Clang.

TODO: write a note about sampling PGO support: https://github.com/rust-lang/rust/issues/117023 and https://github.com/rust-lang/rust/issues/64892

Rustc documentation about PGO is available [here](https://doc.rust-lang.org/rustc/profile-guided-optimization.html).

### Go

TODO: add links about PGO implementation in Go compiler
TODO: add my thoughts the current PGO implementation in Go and its disadvantages

Links:

* https://go.dev/doc/pgo
* https://go.dev/blog/pgo
* https://theyahya.com/posts/go-pgo/

### Fortran

TODO

### C#

Microsoft few years ago started investing into the PGO implementation in C#. Unfortunately, due to lack of C# experience from my side, I cannot say you much about the quality of this implementation, traps and nuances. Instead, I will share with you some interesting from my point of view articles and links about PGO state in C# for further reading:

* [Dynamic PGO in .Net 6](https://gist.github.com/EgorBo/dc181796683da3d905a5295bfd3dd95b). By the way, the article author, [EgorBo](https://gist.github.com/EgorBo), is an expert in C# PGO implementation. So probably you can ask him about more details directly!
* [PGO improvements in .Net 7](https://petabridge.com/blog/dotnet7-pgo-performance-improvements/)
* [Conversation about PGO in Microsoft blog](https://devblogs.microsoft.com/dotnet/conversation-about-pgo/)
* Dynamic PGO official design [documentation](https://github.com/dotnet/runtime/blob/main/docs/design/features/DynamicPgo.md)

### Java

[GraalVM](https://www.graalvm.org/) implements AoT compilation mode for Java. Unfortunately, I have no experience with GraalVM so I cannot tell you, how GraalVM's PGO implementation works in practice, what caveats you should expect there, etc. [Official documentation](https://www.graalvm.org/22.0/reference-manual/native-image/PGO/) has so few interesting details so I even hesitate with recommending it as a further reading. Maybe talking directly with the GraalVM engineers would be a good idea?

GraalVM even [supports](https://github.com/oracle/graal/discussions/7648) sampling PGO but the documentation is a bit outdated. Hopefully, it will be fixed soon.

PGO in GraalVM is available only in GraalVM Enterprise license but don't worry - Enterprise license is free [now](https://blogs.oracle.com/java/post/graalvm-free-license) so you do not need to pay at least for a license if you want to use PGO in Java. Good? Great!

TODO: write about the curently weak GraalVM support across the Java ecosystem.
TODO: add link about Kafka and GraalVM: https://cwiki.apache.org/confluence/display/KAFKA/KIP-974%3A+Docker+Image+for+GraalVM+based+Native+Kafka+Broker
TODO: tooling - https://github.com/oracle/graal/discussions/7901 and profiles compatibility - https://github.com/oracle/graal/discussions/7892

### Kotlin and Kotlin Native

TODO: https://youtrack.jetbrains.com/issue/KT-63357/Kotlin-Native-Profile-Guided-Optimization-PGO-support

TODO: Kotlin Native has no PGO support and has no plans to implement it: https://youtrack.jetbrains.com/issue/KT-63357/Native-Profile-Guided-Optimization-PGO-support#focus=Comments-27-8353088.0-0

### Swift

It's a complicated question. From one perspective, in the Swift compiler sources [there are](https://github.com/apple/swift/blob/main/include/swift/Option/Options.td#L1322) some PGO footprints. From another - even the compiler developer is [not sure](https://github.com/apple/swift/issues/69227) about the current PGO implementation state in the compiler. Anyway, there is an **unanswered** [topic](https://forums.swift.org/t/several-questions-regarding-profile-guided-optimization-pgo-and-llvm-bolt/67963) on the Swift forum. Hopefully, one day it will get an answer.

### Zig

TODO: write a story about a new Zig compiler and how it affects PGO dreams in this language

### D

TODO

### Dart

TODO: add information about Dart: https://github.com/dart-lang/sdk/issues/53855 in `dart2native` mode and https://github.com/dart-lang/sdk/issues/53856

### Nim

TODO: has no support https://github.com/nim-lang/RFCs/issues/130

### Pascal

TODO: https://forum.lazarus.freepascal.org/index.php/topic,65162.0.html

### Haskell

TODO: GHC - no support: https://gitlab.haskell.org/ghc/ghc/-/issues/18393, what about other compilers?

### Julia

TODO: https://github.com/JuliaLang/julia/pull/45641#issue-1268010204

### Ocaml

TODO: https://github.com/ocaml/ocaml/issues/12200

### Cobol

TODO: check Cobol compilers and PGO support in them (lol?)

---

TODO: create a table with programming languages and PGO support status

## PGO support in build systems

* Overview of the PGO state across build systems
  - Add a note about Bazel integration
  - Add links to the issues in other build systems like CMake, Meson, etc. (maybe I need to check it across all popular build systems and create proper issues for that)

In 99.9(9)% cases we do not compile our applications via direct compiler invocations - we use build systems. What kind of PGO support could we expect from a build system? I have the following wishes:

* I want to have a possibility to enable PGO for my application via *a build system flag* rather than *a compiler flag*. Why? Because I can use multiple compilers, each compiler can have different flags for enabling PGO (e.g. check the difference in PGO in MSVC and Clang). And writing such logic for each compiler is not what I want to do for each project where I want to enable PGO.
* Optimizing with PGO whole dependency tree, not only my project. Almost any modern application uses dependencies. And for making things even more complicated, this dependencies can be written in different languages, that are compiled by different compilers (with different PGO flags, as we know). In this case, enabling PGO build for the whole dependency tree is quickly becomes a non-trivial task.
* It should work.

So, what do we have now in the ecosystem?

### Cargo

Cargo (the default build system for Rust) has no built-in support for PGO. But community (particularly [Jakub "Kobzol" Beranek](https://github.com/kobzol)) developed an extension - [cargo-pgo](https://github.com/Kobzol/cargo-pgo). I highly recommend you to use this Cargo extension if you are going to start optimizing Rust projects with PGO. I used for **every** Rust project that I PGOed (and I did it for a lot of them) - it always worked like a charm. It even supports optimizing with LLVM BOLT (we will talk about it later) as an additional post-PGO optimization step! I wish every other ecosystem eventually will get something similar.

TODO: add a note about missing Sampling PGO optimization integration
TODO: add a note about C-dependencies: https://github.com/Kobzol/cargo-pgo/issues/38

### Bazel

Bazel [has](https://bazel.build/reference/command-line-reference#flag--fdo_instrument) built-in PGO support. With provided by Bazel command-line options, you will be able to optimize your program with PGO without needing to know of how to properly invoke PGO stuff in your favourite compiler.

But even such a "native" PGO integration into the build system could have some problems. I [found](https://github.com/envoyproxy/envoy/issues/25500#issuecomment-1724584679) one of them during the PGO testing for Envoy. For some unknown yet reason Bazel denied to compile with PGO one of the Envoy's dependency when I try to use the PGO option. But if I just pass the required compiler flags recursively for all dependencies - it works fine! I don't know what is the root cause for such behavior in this case (honestly, I don't care much since I got my job done there)... Anyway, now you know about this possible caveat and how to avoid the problem :)

### Meson

TODO: add Meson notes

### CMake

CMake (one of the most [popular](https://www.jetbrains.com/lp/devecosystem-2022/cpp/#Which-project-models-or-build-systems-do-you-regularly-use) build system for C++ nowadays) has no built-in PGO support - only a [request](https://gitlab.kitware.com/cmake/cmake/-/issues/19273) for it. So for CMake-based projects passing required PGO flags via `CMAKE_C_FLAGS` and/or `CMAKE_CXX_FLAGS` (or via env `CFLAGS` and/or `CXXFLAGS`) are the only options.

### Package managers (Conan, Vcpkg, etc.)

Not exactly the build systems but I want to mention them here as well.

TODO: Check Conan and Vcpkg support for PGO and how it could be done.

---

TODO: write a conclusion about build systems support - even without an explicit support it's doable to perform PGO optimizations. Write several tricks of how it can be implemented and what caveats I've seen in the projects (like ClickHouse overriding passed CMAKE_CXX_FLAGS and other similar stuff).

## Profile collection approaches

TODO: Describe possible ways to collect PGO profiles in practice

* How to collect PGO profiles?
  - Tests - please no, too many differences with your actual workload
  - Benchmarks - not so bad, but be careful regarding coverage and happy paths
  - Generating sample load and using it - a good one. But you need to maintain it and track that it does not differ much from your actual workload
  - Collecting directly from production - a good one too. But you need good tooling for that + Instrumentation PGO usually is not usable for that

Okay, you decided to give PGO a try in your application. But where can you collect a profile? And, what is more important, where can you collect *a good* profile? There are multiple approaches - let's discuss all of them!

### Tests

The first idea that can appear in your mind - "We have unit/functional/integration/e2e-tests - let's use them for as a sample workload for PGO". Please don't do that.

In most cases, the main aim of your tests is testing (!) your application in all possible situation (including rare and almost impossible scenarios). Instead, PGO optimizes not for all cases but for the most common from your *actual* workload. If you try to use tests as a sample workload for training PGO, you optimize your program not for the usual workload but for the corner cases. That's definetely not what you want to do.

### Benchmarks

Using benchmarks for training is a bit more complicated topic because the answer here is "It depends". It depends on how well your bench suite represents your actual workload. Because if you have a bench suite that measures something not so important in your application - you get completely the same problem as we have with tests.

From my experience, I met the following problems with using benchmarks as a PGO training workload:

* A bench suite measures unimportant things (TODO: add an example here from a real project).
* A bench suite does not provide good coverage for the optimized application.
* A bench suite is broken/not maintained well due to some reasons (TODO: add Quilkin and grcov examples at least)

TODO: insert here a meme about a parrot who learnt to say "It depends" and became an architect
TODO: add project examples where such approach is used
TODO: write about benchmark infra standartization in some ecosystems as Rust with `cargo bench`

Some examples:

* Firefox: [Uses](https://github.com/mozilla/gecko-dev/blob/master/build/pgo/profileserver.py#L25) WebKit performance test suite.
* Pydantic-core: [Uses](https://github.com/pydantic/pydantic-core/blob/main/Makefile#L74) its own benchmarks.

### Manually crafted workload

Here I collected some examples of real-life projects that use this approach for doing PGO:

* ISPC: [Has](https://github.com/ispc/ispc/tree/main/superbuild#build-process) special real-life `ispc-corpus` for PGO training purpose
* Clang: [Uses](https://llvm.org/docs/HowToBuildWithPGO.html#selecting-benchmarks) the instrumented Clang to build Clang, LLVM, and all of the other LLVM subprojects available to it.
* Rustc: [Compiles](https://github.com/rust-lang/rust/blob/master/src/tools/opt-dist/src/training.rs#L19) set of sample open-source crates as a training workload
* Windows Terminal: [Uses](https://github.com/microsoft/terminal/pull/10071) special PGO-oriented user test scenarios.
* Zstd: [Uses](https://github.com/facebook/zstd/blob/dev/programs/Makefile#L240) several Zstd CLI invocations with different parameters
* FoundationDB: [Uses](https://github.com/apple/foundationdb/blob/1a6114a66f3de508c0cf0a45f72f3687ba05750c/contrib/generate_profile.sh) custom workload
* Go compiler: [Compiles](https://github.com/golang/go/blob/master/src/cmd/compile/profile.sh) all targets in `std` and `cmd`.
* Foot: [Has](https://codeberg.org/dnkl/foot/src/branch/master/pgo) multiple predefined scenarios to choose from.



TODO: sort of this list to different categories

* GCC:
  - Official [docs](https://gcc.gnu.org/install/build.html), section "Building with profile feedback" (even AutoFDO build is supported)
  - A [part](https://github.com/gcc-mirror/gcc/blob/4832767db7897be6fb5cbc44f079482c90cb95a6/configure#L7818) in a "wonderful" `configure` script
* Python:
  - CPython: [README](https://github.com/python/cpython#profile-guided-optimization)
  - Pyston: [README](https://github.com/pyston/pyston#building)
* Swift: [CMake script](https://github.com/apple/swift/blob/main/CMakeLists.txt#L364)
* V8: [Bazel flag](https://github.com/v8/v8/blob/main/BUILD.gn#L184)
* ChakraCore: [Scripts](https://github.com/chakra-core/ChakraCore/tree/master/Build/scripts/pgo)
* Chromium: [Script](https://chromium.googlesource.com/chromium/src/build/config/+/refs/heads/main/compiler/pgo/BUILD.gn)
* PHP - [Makefile command](https://github.com/php/php-src/blob/master/build/Makefile.global#L138) and old Centminmod [scripts](https://github.com/centminmod/php_pgo_training_scripts)
* MySQL: [CMake script](https://github.com/mysql/mysql-server/blob/8.0/cmake/fprofile.cmake)
* OceanBase: [CMake flag](https://github.com/oceanbase/oceanbase/blob/master/cmake/Env.cmake#L55)


TODO: add projects where this approach is used

### Production environment

Advantages:

* Production is the most representative environment to get the actual workload profile.
* There is no need to maintain custom near real life workloads, sync them with the actual production workload, monitor a skew between the sample workload and the production, etc.

Disadvantages:

* It can be difficult to collect PGO profiles from external environments. E.g. let's imagine you have a mobile application. It's much more difficult to collect PGO profiles from consumer devices and transfer them to your CI environment for PGO optimization.
* Getting PGO profiles from production affects performance of your production. So do it carefully. It can be partially mitigated by using sampling PGO approach since with sampling you have an option to tweak the performance overhead via increasing/decreasing the sampling frequency ("Sampling overhead <-> Profile precision" tradeoff). Actually, instrumented PGO also can be used in production but before this you need to test/estimate the performance penalty and decide - is it acceptable in your case or not.

Especially for this case, Google created [AutoFDO](https://github.com/google/autofdo) project. Google has a corresponding internal infrastructure for collecting PGO profiles directly from the production environment with almost no overhead.

TODO: Write about the caveat of how to collect PGO profiles from the customer devices, the overhead of taking a profile from the production, etc.
TODO: add somewhere link to "PGO at scale" in Google

### PGO tips

TODO: rework this chapter

* PGO profiles are not written to a disk due to signal handlers. Overwrite them or customize. I highly recommend to tune software to write a profile to a disk with a signal (call `__llvm_dump_profile()` or [similar functions](https://github.com/llvm/llvm-project/blob/main/compiler-rt/lib/profile/InstrProfiling.h) if you use Clang)
* Partial profile sets are useful too since they usually covers a lot of hot paths (LLD and ClickHouse as en example).
* Merging multiple profiles - works well.
* Running on a test suite - generally is not a good idea. Real-life workload usually would differ and a profile from the tests would not be so useful. However, if you have "real-life" tests - it's fine to use them as a profile workload.
* PGO works well with libraries. However, will be a question - how to collect and distribute a profile for them? Especially if we are talking about general-purpose builds like in OS distros. Also, it's not easy to collect a profile from instrumented but non-instrumented binary (e.g. instrumented .so library which is used from Python software). Needs to be clarified but writing an instrumented wrapper right now is a recommended option.
* PGO profiles does not depend on time! However, if your code has time-dependent paths, PGO profiles could differ due to "time stuff" like time issues on a build machine, different speed of different build machines, etc - be careful with that
* PGO works fine without LTO (in normal compilers. MSVC does not belong to this family - you cannot use PGO without LTO there). So if LTO is too expensive to your build resources (especially if we are talking about FatLTO that consumes A LOT of RAM) - just use PGO without LTO, that's completely fine
* It's hard to estimate how slow would be your application in the Instrumentation mode without actual testing. It hugely depends on the hot paths of your application, number of branches, etc. It's possible to predict based on some metrics of the application. But much easier just to test it :) Be careful, some application could be too slow in Instrumentation mode (like ClickHouse, Clangd or LLD)
* When recompile software, would be useful to recompile 3rd parties with PGO as well. Here could be a challenge since often provided by OS packages dependencies are used. And it could be challenging to recompile them as well from the scratch (especially if we are talking about C and C++). Also, not for every software is obvious to understand - which 3rd parties are rebuilt from the scratch and which are taken from another place (like OS-provided or some package manager like Conan).
* For reproducibility purposes you can just collect profile and commit it to the repo

## Post-Link time Optimization (PLO)

What if PGO is not enough for us and we want to squeeze even more performance "for free" with no manual optimizations? Well, in this case, the industry already has something to offer. These kinds of tools are called Post-Link Time Optimizers or simply PLO.

TL;DR, the idea behind PLO is simple - try to reduce [CPU instruction cache (I-cache)](https://en.wikipedia.org/wiki/CPU_cache#Overview) and [Instruction Translation Lookaside Buffer (iTLB)](https://en.wikipedia.org/wiki/Translation_lookaside_buffer#Overview) misses for applications. We can achieve it by rearranging code in our binary according to our execution profile: hot code executed together we place together in the binary, so these pieces of code highly likely will be uploaded to a CPU at the same time.

Right now, there are two of the most mature tools in this area: [LLVM BOLT](https://github.com/llvm/llvm-project/blob/main/bolt/README.md) (from Facebook/Meta) and Propeller (from Google). Let's discuss each of them.

TODO: Write about BOLT, PROPELLER, and others (like Dynamic BOLT)

### BOLT

TODO: write about BOLT advantages and disadvantages
TODO: write about BOLT instrumentation slowdown like https://github.com/qarmin/czkawka/issues/1099 and https://github.com/crate-ci/typos/issues/827#issue-1888263250
TODO: write about the naming conflict - LLVM BOLT and https://gitlab.freedesktop.org/bolt/bolt
TODO: add https://discourse.llvm.org/t/bolt-open-projects/61857

| Application | BOLT Instrumentation to Release slowdown ratio | Benchmark link |
|---|---|---|
| czkawka | 2.66x | [link](https://github.com/qarmin/czkawka/issues/1099#issue-1946612100) |
|  |  |  |

TODO: add BOLT instrumentation binary large ratio

| Application | Release size | Instrumented size | BOLT optimized size | Instrumented to Release ratio |
|---|---|---|---|---|
|  |  |  |  |  |

BOLT is already integrated into the optimization pipelines in several projects:

* Rustc: [GitHub PR](https://github.com/rust-lang/rust/pull/116352)
* CPython: [GitHub PR](https://github.com/python/cpython/pull/95908)
* Pyston: [README](https://github.com/pyston/pyston#building)
* Clang: [CMake script](https://github.com/llvm/llvm-project/blob/main/clang/cmake/caches/BOLT.cmake)

E.g. `rustc` compiler is already PGO + BOLT optimized for Linux platforms.

More materials? Yeah, I have something to share:

* BOLT original paper: [Facebok engineering blog](https://research.facebook.com/publications/bolt-a-practical-binary-optimizer-for-data-centers-and-beyond/)
* Optimizing Linux kernel with BOLT: [Youtube](https://www.youtube.com/watch?v=ivTCCTSMGZg)

### Propeller

TODO: Do I have any benchmarks? I guess putting from the official Google papers is fine here

Do you want more? Here we go:

* Original Propeller repo: [GitHub](https://github.com/google/llvm-propeller)
* 2019 LLVM Developers’ Meeting: S. Tallam “Propeller: Profile Guided Large Scale Performance...”: [Youtube](https://www.youtube.com/watch?v=DySuXFGmB40)

### Other approaches

TODO: write here about Dynamic BOLT and other ongoing research in this area

### What to choose?

From my experience, right now BOLT would be the better option to start with if we are talking about PLO usage.

TODO: add thoughts about unification efforts between BOLT and Propeller: https://github.com/google/llvm-propeller/issues/10

## PLO state in the ecosystem

* PLO has no integration in the Linux distros yet
* PLO has no integration in the build systems

## LTO, PGO, PLO and proprietary software

Everyday we use not only open-source software, that can be (simply) optimized with LTO, PGO, PLO and other fansy stuff by our own hands in a Gentoo-style but also some proprietary software where we have no such an option. And still proprietary software performance can be valuable for us: CI performance with closed-source compilers, IDE performance, closed-source database performance (in both on-premise and SaaS versions) - there are many of them. So what could we do instead?

I have tried to test a silly thing: write directly to the companies with an idea of optimizing their software with PGO. Here are some results:

* Sent an email to <opensource@apple.com> about enabling PGO for Apple products like Apple Clang, prebuilt FoundationDB, etc - no response.
* Created the [topic](https://forums.percona.com/t/profile-guided-optimization-pgo-for-databases/24035) on the official Percona forum about optimizing its products with PGO - no response
* Sent an email to Nvidia via Developer Contact email about enabling PGO for their HPC compilers with all required information about PGO. Only got a suggestion about creating a topic on their HPC forum. I have tried multiple times to create an account on this resource but with no success due to unknown for me reasons. No more responses from them by email, btw.
* Sent an email to [NauEngine](https://nauengine.org/) devs about evaluating PGO for this game engine - **got a response** that PGO information was sent to development teams. Great!

TODO: write a section about how I was ignored by different companies with an idea of PGO and other stuff
TODO: write several pieces of advice/thoughts about how users can try to push PGO for their products being users AND a developer of such products

* PGO and proprietary software
  - ArenagraphDB just didn’t answer my request - maybe an email wasn’t enterprise-enough? :)
  - NauEngine: sent an email about PGO (no response yet)

## Why am I writing this?

* I like performant applications
There are multiple reasons for that:
* For me it would be easier to work in the IT industry, where we have "PGO by default" mindset. Because with faster software it's easier to achieve the required NFR (Non-Functional Requirements) before the horizontal scaling questions.
* Because I can
* Just for lulz

## Software pipeline

Let's take a look at how looks like a default **open-source** software pipeline:

TODO: insert a diagram here

## Test environments

### Hardware and OS

TODO: add a photo of my Mac with my PC test setup (add a screen with Yorha units with the question "Are Yorha units PGO-optimized or not?")

During the tests, I used two test setups: my Windows-Linux dual-boot PC and Macbook with macOS.

PC:

* CPU: AMD Ryzen 9 5900x
* RAM: 48 Gib of some default non-RGB RAM
* SSD: Samsung 980 Pro 2 Tib
* OS: Fedora 38
* Kernel: Linux kernel 6.* (mostly 6.2.* - 6.5.*)

Macbook:

* CPU: Apple M1 Pro 6+2
* RAM: 16 Gib
* SSD: 512 Gib
* OS: macOS (mostly 13.* versions)

In the article, my PC setup is called "Linux", and Macbook as "macOS". Sorry Windows people, I have no desire to perform all the tests on Windows. But anyway - it should work (and works) on this OS completely in the same way.

### Compilers

For all Rust projects, I use the `rustc` compiler with LLVM backend, versions somewhere in 1.68 - 1.73 range. For C and C++ I prefer using Clang 16, sometimes I use GCC 13 as an additional compiler.

All these compilers are in their "default" state with no custom patches from my side: `rustc` from the official website, Clang and GCC from the official Fedora repository.

## PGO state across Linux distributions

TODO: finish chapter

  - Sometimes PGO is disabled on some platforms due to a lack of build resources
  - Sometimes PGO is disabled due to reproducible builds concerns. Concern about reproducing the profile. Bugs: https://bugzilla.opensuse.org/show_bug.cgi?id=1040589 , NixOS comment (find and link it here)
  - Could we trust the profiles from the upstream to use them instead of our own? Good question. That’s why is important to commit scripts for reproducing the profile (and still can differ due to time-based things like I had in YDB)

In many cases, we don't use binaries directly from developers or we don't rebuild all the things in our perimeter (despite knowing such security requirements in some areas!) - we use *prebuilt* binaries from our favorite OS distribution. But if the binaries are prebuilt by someone, there is a chance that PGO can be disabled even if the PGO-optimized build is supported by the upstream. How does it work in practice?

Unfortunately, PGO is rarely enabled in the provided by OS repositories. Let's take a look at some examples:

Even if the upstream project supports building with PGO, it doesn't mean that in your favorite operating system, this package is built with PGO support. Let's study several examples:

| Application | Fedora | Alpine | Mageia | NixOS | Solus | Void Linux |
|---|---|---|---|---|---|---|
| GCC |  |  |  |  | [Yes](https://github.com/getsolus/packages/blob/main/packages/g/gcc/package.yml#L164) | [No](https://github.com/void-linux/void-packages/blob/master/srcpkgs/gcc/template) |
| Clang |  |  |  |  | [Yes](https://github.com/getsolus/packages/blob/main/packages/l/llvm/package.yml#L116) | [No](https://github.com/void-linux/void-packages/blob/master/srcpkgs/llvm15/template) |
| Rustc |  |  |  |  |  | [No](https://github.com/void-linux/void-packages/blob/master/srcpkgs/rust/template) |
| CPython |  |  |  |  | | [Yes](https://github.com/void-linux/void-packages/pull/43791) |
| Chromium |||||||
| Firefox | [Yes](https://src.fedoraproject.org/rpms/firefox/blob/rawhide/f/firefox.spec) ||||||
||||||||
||||||||

## PLO state across operating systems

TODO: finish chapter

https://github.com/getsolus/packages/blob/main/packages/l/llvm/package.yml#L116 - example of BOLT integration in the upstream

## PGO / PLO state across package managers

TODO: Check Altinity builds for ClickHouse and suggest PGO for their CH build. Their sources are available here: https://github.com/Altinity/ClickHouse

## Stories

TODO: finish chapter

  - Developers do not believe in their benchmarks (DuckDB and Broot)
  - Developers are unresponsive (SQLite, MongoDB and others)
  - Bugs with LTO and PGO in Rustc compiler
  - ClickHouse in the Instrumentation mode runs the ClickBench suite for more 30+ hours on my Linux setup
  - Got an additional RAM from my friend since 32 Gib is not enough for LLVM BOLT :)
  - Needed to complete registrations in many bugzillas or even trickier platforms - please add the possibility to login with Google, GitHub or smth like that
  - Some people integrate PGO even without benchmarks - just because they can :) Usually, it’s not a good idea (since PGO can give you nothing but you will pay for the longer compilation process)
  - Problems with hardware architecture availability in CI (GitHub Actions as an example of such CI service)


During my PGO tests across the ecosystem, I collected some interesting PGO-related stories. I think it would be a good thing to share them here and learn some lessons.

### YDB

TODO: Hardcoded timeout issue in YDB and Instrumentation PGO

When I was working on testing PGO with different databases, I found [YDB](https://ydb.tech/) - an interesting database from Yandex. And I decided to optimize it with PGO. YDB is written in C++ (PGO can be applied), [has](https://ydb.tech/en/docs/reference/ydb-cli/commands/workload/) built-in benchmarks (PGO can be easily trained at least on these benchmarks).

I recompiled YDB with instrumentation PGO and started the benchmark to collect PGO profiles. But for some reason, the benchmark failed. After a deeper investigation, I found an interesting behavior - hitting internal YDB deadlines. Since the instrumented binary was running slower, internal deadlines started to interrupt the PGO training phase. I increased [the limit](https://github.com/ydb-platform/ydb/blob/main/ydb/core/load_test/kqp.cpp#L321) (since I had no glue how to fix it properly) and it the issue was fixed.

However, since the upstream has no proper fix yet, and you decide to optimize YDB with PGO - just be aware of the issue!

Lessons learned:

* Having built-in benchmarks is a useful thing!
* Timeouts are not jokes and actually can hurt your PGO trip

### HAProxy

TODO: write a story about HAProxy multiple benchmarks

Once I [created](https://github.com/haproxy/haproxy/issues/2047) an issue about PGO in HAProxy repo, and [Willy Tarreau](https://github.com/wtarreau) (the HAProxy maintainer) said that there would be no effects from PGO on HAProxy. I argued a little bit about PGO, CPU consumption, HAProxy actual bottlenecks, and I was not able to win this dispute without an actual benchmark. So I decided to prove my point of view.

I crafted a benchmark, showed the positive results to Willy, he asked me to improve my benchmark scenario and show additional information like binary sizes and perform PGO tests on GCC too (I use Clang as a default C and C++ compiler). After several iterations, Willy [concluded](https://github.com/haproxy/haproxy/issues/2047#issuecomment-1729980766) that PGO shows measurable improvements. It was a huge win for me!

I **really** appreciate such attention to the details and wish to understand as many details as possible about PGO from Willy Tarreu. Even if it required performing multiple tests, I still think it was worth it.

Lessons learned:

* Maintainers know the domain much better than you and can help you with crafting the "right" benchmark
* Disputing about PGO is much easier with the actual benchmark results

### SQLite

Write a story about how I was ignored by SQLite developers

### Stolyarov

TODO: A comment from Stolyarov about PGO in Thalassa :)

In the Russian-speaking Internet segment, there is a meme - Aleksei Stolyarov. His own webpage is [here](http://stolyarov.info/). Recently he published his own C++ CMS - [Thalassa](https://github.com/a-croco-stolyarov/thalassa).

I decided to [propose](https://github.com/a-croco-stolyarov/thalassa/issues/1) testing PGO on Thalassa. And I expected, got a wonderful answer!

TODO: add a screenshot here: https://github.com/a-croco-stolyarov/thalassa/issues/1#issuecomment-1721060548

Besides jokes, don't be like Aleksey regarding PGO. PGO allows you spend less time with manual program optimization. So even if you are a **real** programmer - do not hesitate to use PGO!

## PGO tips

TODO: finish chapter

## FAQ

### Is PGO a silver bullet?

TODO: What to do with performance regressions after PGO? Rustc question: https://users.rust-lang.org/t/how-to-report-performance-regressions-with-profile-guided-optimization-pgo/98225

### Can I use PGO without LTO?

TODO: add info that ThinLTO is not so slow compared to FatLTO like it was proven in https://github.com/ldc-developers/ldc/issues/2168#issuecomment-313969487

Yes! Link-Time Optimization (LTO) and PGO are completely independent and can be used without each other.

However, usually, LTO is enabled before PGO. Why it happens? Because both LTO and PGO are optimization techniques with an aim to optimize your program. In most cases, enabling LTO is much easier than PGO because enabling one compiler/linker switch with LTO is an easier task than introducing 2-stage build pipelines with PGO. So please - before trying to add PGO to your program try to use LTO at first.

There are some situations, when you may want to avoid using LTO with PGO:

* Weak build machines. LTO (even in ThinLTO mode) consumes a large amount of RAM on your build machines. That means if your build environment is highly memory-constrained - you may want to use PGO without LTO since PGO usually has lighter RAM requirements for your CI. (TODO: add Linux distribution examples here in the build scripts)
* Compiler bugs. Sometimes PGO does not work for some reason with LTO (like [this](https://github.com/rust-lang/rust/issues/115344) and [this](https://github.com/rust-lang/rust/issues/117220) bugs in the Rustc compiler). Even without PGO enabling LTO can bring multiple bugs - e.g. check YugabyteDB LLVM [fix](https://github.com/yugabyte/llvm-project/commit/64d871949eb23145af7b97cb13feaeeeee7ab39a).

I have several examples of how LTO improves performance:

| Application | Release | Release + LTO | Link |
|---|---|---|---|
| legba |  |  |  |

Some Linux distributions a few years ago started to integrate LTO as a default compiler flag for building their software. Here are some examples:

* Debian: [link](https://wiki.debian.org/ToolChain/LTO)
* Ubuntu: [link](https://wiki.ubuntu.com/ToolChain/LTO)
* Fedora: [link](https://fedoraproject.org/wiki/LTOByDefault)
* OpenSUSE: [link](https://en.opensuse.org/openSUSE:LTO)

Enabling LTO by default is a huge step that highlights dozens of bugs in your software (especially C and C++). You see it on a Gentoo example - they already collected many LTO-related issues (links to [Gentoo bugzilla](https://bugs.gentoo.org/618550), [gentooLTO repository](https://github.com/InBetweenNames/gentooLTO/issues)). I am sure you can find similar issues in other bugtrackers as well.

However, not all distributions consider enabling LTO by default due to the problems discussed above. One of the examples is NixOS (the discussion can be found on [Reddit](https://www.reddit.com/r/NixOS/comments/146wdfk/lto_by_default/)). Hopefully, over time more and more issues with LTO will be resolved and we will see "LTO by default" policy enabled in more build guidelines.

If you are interested in more details about LTO, I collected several links for you:

* ThinLTO: [Google Research](https://research.google/pubs/pub47584/), [Youtube](https://www.youtube.com/watch?v=9OIEZAj243g)
* 

TODO:
- add more information about Fat and Thin LTO
- add about LTO state across Linux distributions
- add LTO performance improvement result examples
- add more LTO links to articles and talks

### What about PGO profile compatibility between compilers?

**Awful**. PGO profiles are incompatible between compilers. So if you have PGO profiles generated by GCC, it's imporssible to use them in Clang or MSVC. In theory, it's possible to write a converter between profile formats but I don't know such projects (of course you can try writing your own. If you do - please let me know!).

Compatibility is not guaranteed even between versions of the compiler! So if you upgraded from Clang X to Clang Y - probably you need to regenerate your PGO profiles as well. And if you want to support multiple compilers - you need to maintain multiple PGO profile versions (or just generate PGO profiles on-demand for each compiler).

I tried to find strict profile format definition, forward/backward compatibility guarantees, migration guides, built-in versioning support but with no success. If you are a compiler engineer and you can fix it somehow - please do it! It would be helpful for the whole community!

### Regenerating vs caching PGO profiles

TODO: finish section

Usually, there are two ways to work with PGO profiles:

* Generate PGO profiles again for each build
* Cache PGO profiles and reuse them for builds

Here are some examples of how it's done in some projects:

* Android Open Source Project (AOSP): [repository](https://android.googlesource.com/toolchain/pgo-profiles/+/refs/heads/main)
* file.d: [GitHub file](https://github.com/ozontech/file.d/blob/master/default.pgo)

### Algorithm optimizations always outperform machine optimizations

TODO: here we need to discuss an opinion from Rainer Grimm (Rsyslog dev). Rainer Grimm from Rsyslog thinks that algorithm always beats peephole optimizations (it’s wrong - they work at the same time)
TODO: also a comment from Khronos group: https://github.com/KhronosGroup/glslang/issues/3400#issuecomment-1817277433

### Is PGO/PLO CPU-specific optimization?

TODO: here write about the common mystery that PGO depends on the target CPU - no, it doesn't (but could be interesting things like runtime dispatching and non-covered code paths).

### Do I need to integrate LTO, PGO or PLO without benchmarks

TODO: discuss here such a topic

### How PGO and/or PLO changes a binary size?

TODO: PGO helps with optimizing binary size since we can inline less for actually cold paths of our programs (and it can help with performance as well since our program will be smaller and more friendly for CPU I-cache). Write that binary can become larger (due to more agressive inlining) or smaller (if inlining is not required in some places) - it depends on the software kind and actual profiles.

TODO: Do I need this chapter since this topic was discussed above with actual numbers?

## Related projects

TODO: write about ASOS, machine-learning-based compilers, etc.
TODO: write about OCOLOS - https://people.ucsc.edu/~hlitz/papers/ocolos.pdf
TODO: write about improved PGO workflow: https://github.com/llvm/llvm-project/issues/58422

Here I want to show you some PGO-related ongoing research or similar ideas that possibly you find at least interesting.

### Application Specific Operating Systems (ASOS)

TODO: add info about ASOS
TODO: add a diagram about how our applications are deployed on OS
TODO: write about an idea like ASOS but for interpreters - like application-specific interpreters, where each interpreter is built with application-specific PGO profile (my Lua experiments as an example)

Before we talked about optimizing with LTO/PGO/PLO our own applications. But in 99.99(9)% cases we run software on operating systems. But what if we try to PGO optimize an operating system too?

Already there are multiple PGO tests on this topic:

* Linux kernel:
  - "One Profile Fits All: Profile-Guided Linux Kernel Optimizations for Data Center
Applications" [paper](https://web.eecs.umich.edu/~takh/papers/ugur-one-profile-fits-all-osr-2022.pdf)
  - [Microsoft presentation about Linux with PGO](https://lpc.events/event/7/contributions/771/attachments/630/1193/Exploring_Profile_Guided_Optimization_of_the_Linux_Kernel.pdf)
  - [TCP Stream perf from Linux Plumbers conference](https://lpc.events/event/7/contributions/798/attachments/661/1214/LTO_PGO_and_AutoFDO_-_Plumbers_2020_-_Tolvanen_Wendling_Desaulniers.pdf)
  - [Phoronix post about Clang PGO for Linux](https://www.phoronix.com/news/Clang-PGO-For-Linux-Next)
  - Yet another attempt to PGO Linux kernel: http://coolypf.com/kpgo.htm
  - [Gentoo Wiki about the Linux kernel optimization](https://wiki.gentoo.org/wiki/Kernel/Optimization#Performance)
  - Optimizing Linux kernel with Clang. An [article](https://habr.com/ru/companies/ruvds/articles/696236/)(in Russian) and the [results](https://github.com/h0tc0d3/linux_pgo)
* Windows: 5-20% improvement according to the [presentation](https://lpc.events/event/7/contributions/771/attachments/630/1193/Exploring_Profile_Guided_Optimization_of_the_Linux_Kernel.pdf)

Unfortunately, I have no PGO results for other operating systems like macOS (almost impossible to get insights from Apple, you know) or *BSD (e.g. see [this](https://www.reddit.com/r/freebsd/comments/150codp/profileguided_optimization_pgo_on_freebsd_kernel/) Reddit post). If you can perform tests for other operating systems or already know some PGO results - please let me know!

Operating systems can be optimized with PGO in the same as we do for usual applications: compile with instrumentation (if we choose instrumentation PGO), run target workload on the operating system, collect PGO profiles, and recompile the OS once again.

More information can be found in the original paper [here](http://scis.scichina.com/en/2018/092102.pdf).

### Machine learning-based compilers

TODO: write about TF-based inlining for LLVM and write a little bit about the current inline implementation in LLVM
TODO: Intel and ML PGO: https://www.intel.com/content/www/us/en/docs/dpcpp-cpp-compiler/developer-guide-reference/2023-2/fprofile-ml-use.html

## Awesome PGO people

TODO: add links to the people (GitHub, their blogs, etc.)

If you decide to integrate PGO, probably at some point you will meet some problems, a corner case or find a nice way to improve PGO for your specific workload. So here I want to share the PGO experts list (not complete!). I believe these people could help you in your PGO journey:

* Jakub Beranek aka [Kobzol](https://github.com/Kobzol) - main PGO wizard in the Rust community.
* Maxim Panchenko aka [maksfb](https://github.com/maksfb) - BOLT developer
* Amir Aupov aka [aaupov](https://github.com/aaupov) - BOLT developer
* [ptr1337](https://github.com/ptr1337) - CachyOS founder and developer

Additionally, I reccomend you to [join](https://discord.gg/xS7Z362) LLVM Discord server. It has many kind and clever people who can help you with your questions regarding LTO, PGO, PLO and other compiler-like fancy stuff. There is a dedicated channel for BOLT too.

## Future plans

TODO: test and advertise PGO in the Go ecosystem (when it becomes a bit more mature)

## Ideas

TODO: write here some ideas for improving PGO situation

I have several ideas about PGO and PLO future improvements across the ecosystem. Maybe something from my list will be so interesting to someone, so they eventually will be implemented.

### Write a PGO handbook

TODO: write how I like handbook formats, give some examples like Rustbook, write why education is important and what I want to achieve with this book

Here are some good examples:

* A plenty of Rust-related books: [Rustbook](https://doc.rust-lang.org/book/), [The Rustonomicon](https://doc.rust-lang.org/nomicon/)
* [Unofficial Bevy Cheat Book](https://bevy-cheatbook.github.io/)

### Improve PGO state in various ecosystems

TODO: write here at least about Java AoT state (that blocks PGO efforts) and Go (we are waiting for the more mature PGO implementation in the main Go compiler), CNCF projects opportunity

* We have a lot of Java software nowadays. And before we Rewrite It In Rust (RIIR) we need to care about the Java performance too. [GraalVM](https://www.graalvm.org/) helps with AoT compiling Java applications to the native mode. Since we want to integrate PGO into Java too, at first we need to integrate GraalVM well into the ecosystem.

### Improve PGO tooling

From the tooling perspective, we have a big room for improvements. And here I am not even talking about build systems/package managers integrations. There are much more ways to integrate PGO into the daily life:

* I created an [issue](https://github.com/compiler-explorer/compiler-explorer/issues/5414) about integrating PGO to [Godbolt](https://godbolt.org/) platform. Godbolt is a great place to play with different compilers, compiler flags to measure their influence on the compiled programs. I think it would be great to have a possibility to show differences betwenn PGOed and non-PGOed code on some toy examples directly on Godbolt. Right now it's not possible to do since PGO requires to store somehow the gathered profiles from the instrumentation phase but Godbolt is stateless (from this perspective). Hopefully, someone would be interested enough to implement it.
* It seems to be an interesting idea to implement some kind PGO profiles integration into IDE. Let's say you open your favorite IDE/text editor, open some source code and right here you can see hot and cold paths in your code. And this information is not just your mental predicion (like "exception path highly-likely is a cold path") but based on the actual PGO profiles. How can you use this information? Well, e.g. for load balancing your efforts regarding optimizations across your program - take care at first about the hot paths in your application. If there is a [request](https://youtrack.jetbrains.com/issue/IDEA-332604/Support-voice-messages-code-comments) voice message in code comments support (and someone even [implemented](https://github.com/polina4096/voices) it!) - why not to have a PGO support? :)

### Write PGO/PLO profiles gathering ecosystem

TODO: write about mine and Amir's idea for a profiles gathering daemon like it's done in Google (or they'll open source it eventually)

The original idea is described [here](https://github.com/aaupov/school_topics/blob/main/perfd.md) (in Russian, so if you don't know the lang - please use the translator or ask Amir about making it English-first :), a few initial commits are available [here](https://github.com/linux-perfd/perfd).

Another idea is gathering PGO profiles (instrumentation or sampling, whatever) via crowd-sourcing like mechanism. One of the biggest issue with PGO is collecting actual usage profiles. But what if we had a repository where each software engineer can get actual PGO profiles for its application and use it for PGO-optimized builds on their official site? It could be done e.g. via a specific OS build where all PGO-potent applications are built with instrumentation or continually collect sampling profiles via `perf` or something like that? There are **a lot** of concerns here like privacy issues (potential mapping "some_user_id -> application execution profiles"), security issues (organize an attack on an application performance via messing publicly-gathered PGO profiles), implementation (who and how would implement all this stuff?!), and much more. Maybe community-driven projects like [Boinc](https://boinc.berkeley.edu/) could be a good example here - right now it's just an idea, no more. And I am [not alone](https://www.phoronix.com/forums/forum/phoronix/latest-phoronix-articles/1422805-llvm-now-using-pgo-for-building-x86_64-windows-release-binaries-~22-faster-builds?p=1422977#post1422977) with this idea.

### Improve PGO/PLO visibility across the industry

PGO and similar optimizations have too poor visibility across the industry (IMHO). I think spread the word about PGO, its benefits, traps, etc. would be valuable investment to the whole industry. It's one of the reasons why I wrote this article. I have several ideas of how the situation can be improved:

* Talk more about PGO on the conferences. I like conferences, having a lot of tech-minded people around me - it's a lot fun! I think it would be great to give multiple talks about PGO on different conferences. It could be language-specific conferences (like [Rust](https://foundation.rust-lang.org/events/) or [C++](https://isocpp.org/wiki/faq/conferences-worldwide) conferences) where we can discuss language-specific PGO aspects. It could be something more specific like "PGO from a maintainer perspective" on [PackagingCon](https://packaging-con.org/). Maybe even prepare some PGO-workshops (like [performance workshops](https://conf.researchr.org/track/cgo-2023/cgo-2023-workshops-and-tutorials) on CGO) - why not?
* I am thinking about trying to collaborate with Google with a project about integrating PGO into different kinds of software as a part of [Google Summer Of Code](https://summerofcode.withgoogle.com/) (GSoC). Didn't have such experience before but heard a lot of warm words about this activity. Maybe it would be possible to integrate PGO/PLO into several projects with GSoC help.

TODO: collaboartion with companies, etc.

## Helpful/interesting links to know about

TODO: Finish links section

Here I collected a bunch of PGO and optimization related links that I can recommend to read:

* [Awesome PGO](https://github.com/zamazan4ik/awesome-pgo). It's my repository, where I collect **everything** about PGO: benchmarks for different software, bugs in compilers, tracking PGO issues in multiple projects, etc. If anyone asks you something like "Hi ${PERSON_NAME}! Could you please send me a link to dive into PGO?" - it should the first appeared in your mind link to share! From my side, I will try to maintain it on the corresponding quality level (at least for some time).
* [PGO in details in Go](https://theyahya.com/posts/go-pgo/). I think it would be interesting to read for Go people.

## Conclusion

TODO: finish the chapter

* Piece of advice for the developers:
  - If you are a language/compiler designer - please consider integrating PGO into your language/compiler
  - If you are a project developer - please consider providing better PGO integration into your project if you care about the performance
  - If you are a maintainer - please consider enabling PGO in your packages
  - If you are an OS package comittee member (like Fedora FESCO) - please consider general PGO movement across the whole package policy
  - If you have experience with PGO in production - please share your numbers/pains/experience with us!

I want to finish with the call to the community.

If you are a language designer - please at least consider implementing AoT compilation model for your language. Of course this model is not suitable for all cases but would be nice to have a possibility to compile (and preoptimize with PGO) a program before the launch. Cold start time matters nowadays, you know.

If you are a language/compiler designer - please consider integrating PGO into your language/compiler

I hope you enjoyed the article! I tried to share all my experience with PGO. And if at least one person after reading all these materials will try to integrate PGO into their applications - I would be happy!

## My contacts

If you want to discuss with me more things about PGO, think about contributing to PGO or know anything PGO-related, you can find me via:

* Email: zamazan4ik@tut.by (primary email) or zamazan4ik@gmail.com (secondary email)
* Telegram: zamazan4ik
* Discord: zamazan4ik

Also, you can create an issue or open a discussion in https://github.com/zamazan4ik/awesome-pgo, if you want to contribute PGO results or discuss something publicly. Just be careful about possible NDA violation and similar stuff.

TODO: note about MSVC compiler and combined LTO/PGO mode as the only option to enable PGO.

TODO: Possible plot for the talk:

1. Why did I write this post? Intro why I like fast software, etc.
2. Show, why PGO is a worth thing to discuss with performance numbers for the most famous projects like PostgreSQL, Clang, MongoDB
3. PGO's current ecosystem state from multiple perspectives: languages, compilers, problems, etc.
4. What it can be in the future?

TODO: article meme ideas

* meme about Nier androids and "Are they PGO optimized?"
* meme about DMC Vergil and "Show me your PGO motivation"
* meme about AI in compilers and AYAYYAAYAYAYAY meme 10h
* meme about Rick and Morty "Go here and there  - 10 minutes" about PGO journey
* think about Rika Makiba meme and PGO? What would be a better idea here?
