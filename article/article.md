# Profile-Guided Optimization (PGO): a long hard road to the hell

In this article, I want to discuss one compiler optimization technique - Profile-Guided Optimization (PGO). We will talk about PGO pros and cons, obvouis and tricky traps with PGO, take a look on the current PGO ecosystem from different perspectives (programming languages, build systems, operating systems, etc.). I spent many days with applying PGO to different applications... Tweaking build scripts, many measurements, sometimes tricky debug sessions, several compiler bugs and much more - it was (and still is) a great journey! I think my experience would be helpful for other people so you will be able to avoid my mistakes from the beginning and much nicer optimization experience.

The article is written in a story style. So the information about PGO is mixed with a lot of examples (because I like examples from the real world) and my own thoughts regarding multiple aspects PGO and PGO-related aspects.

The article is huge enough, so do not forget to grab some tea, beer or something like that before the begginning. Enjoy!

## Target audience

I think the article would be interesting for anyone, who cares about software performance. Especially it should be interesting for people who wants to extract more performance from applications without tweaking the application itself.

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
| Chromium | up to 12% faster | [one](https://blog.chromium.org/2016/10/making-chrome-on-windows-faster-with-pgo.html), [two](https://blog.chromium.org/2020/08/chrome-just-got-faster-with-profile.html) |
| Firefox | up to 12% faster | [Google groups](https://groups.google.com/g/mozilla.dev.platform/c/wwO48xXFx0A/m/ztg4i0DYAAAJ) |
| Envoy | up to +20% RPS | [GitHub comment](https://github.com/envoyproxy/envoy/issues/25500#issuecomment-1724584679) |
| HAProxy | up to +5% RPS | [GitHub issue](https://github.com/haproxy/haproxy/issues/2047) |
| Vector | up to +15% EPS | [GitHub issue](https://github.com/vectordotdev/vector/issues/15631) |

Of course it's not a complete list - much more PGO showcases you can check right now [here](https://github.com/zamazan4ik/awesome-pgo#pgo-showcases). If you are interested - let's go to our PGO journey!

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
* Profile-Directed Feedback (PDF)
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

TODO: add as much projects as I can here



| Application | Instrumentation to Release slowdown ratio | Benchmark link |
|---|---|---|
| HAProxy (Clang) | 1.10x - 1.20x | links ([one](https://github.com/haproxy/haproxy/issues/2047#issuecomment-1729971279), [two](https://github.com/haproxy/haproxy/issues/2047#issuecomment-1728265165)) |
| HAProxy (GCC) | 1.24x | links ([one](https://github.com/haproxy/haproxy/issues/2047#issuecomment-1729971279), [two](https://github.com/haproxy/haproxy/issues/2047#issuecomment-1729606775)) |
| Fluent-Bit | 1.48x | [link](https://github.com/fluent/fluent-bit/discussions/6638#discussioncomment-6419880) |
| ClickHouse | up to **311x** (it's not a mistake!) | [link](https://github.com/ClickHouse/ClickHouse/issues/44567#issuecomment-1519136060) |
| Tarantool | up to 1.5x | [link](https://github.com/tarantool/tarantool/issues/8089#issuecomment-1580628168) |
| Lychee | 4.0x | [link](https://github.com/lycheeverse/lychee/issues/1247) |
| grcov | up to 10x | [link](https://github.com/mozilla/grcov/issues/1128#issue-1956099937) |
| quilkin | 1.5x | [link](https://github.com/googleforgames/quilkin/issues/834#issuecomment-1772719309) |
|  |  |  |
|  |  |  |
|  |  |  |

TODO: write about more deep statistics like slowdown percentiles, testing different configurations, etc.

TODO: write about cases when an instrumented binary is faster (I met such cases in some strange situations. Probably some combination of compiler optimizations does this - who knows, didn't investigate deep such cases).

#### Instrumented binary is larger

Instrumentation PGO works via inserting into your code some counters for tracking the program execution characteriscs, so your binary will be larger after the instrumentation. In assembler it looks something like:

TODO: add before and after PGO instrumentation assembler for Clang and GCC: https://godbolt.org/z/ofMKzEscn

How much binary space does it take in practice? Let's check it:

| Application | Release size | Instrumented size | PGO optimized size | Instrumented to Release ratio | Compiler |
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

TODO: add information when PGO firstly appeared in C compilers

Here are some PGO integration examples into the different compilers:

* [GCC](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html#Instrumentation-Options)
* [Clang](https://clang.llvm.org/docs/UsersManual.html#profile-guided-optimization)
* [MSVC](https://learn.microsoft.com/en-us/cpp/build/profile-guided-optimizations)
* [ICC](https://www.intel.com/content/www/us/en/docs/cpp-compiler/developer-guide-reference/2021-8/profile-guided-optimization-pgo.html)
* [AOCC](https://www.amd.com/en/developer/aocc.html#documentation) (supports but the documentation right now exists only as PDF files)

### Rust

Rust right now has one major compiler - `rustc`. Rustc is based on LLVM so PGO implementation shares almost all details with Clang.

TODO: write a note about sampling PGO support: https://github.com/rust-lang/rust/issues/117023

Rustc documentation about PGO is available [here](https://doc.rust-lang.org/rustc/profile-guided-optimization.html).

### Go

TODO: add links about PGO implementation in Go compiler

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

### Java

[GraalVM](https://www.graalvm.org/) implements AoT compilation mode for Java. Unfortunately, I have no experience with GraalVM so I cannot tell you, how GraalVM's PGO implementation works in practice, what caveats you should expect there, etc. [Official documentation](https://www.graalvm.org/22.0/reference-manual/native-image/PGO/) has so few interesting details so I even hesitate with recommending it as a further reading. Maybe talking directly with the GraalVM engineers would be a good idea?

GraalVM even [supports](https://github.com/oracle/graal/discussions/7648) sampling PGO but the documentation is a bit outdated. Hopefully, it will be fixed soon.

PGO in GraalVM is available only in GraalVM Enterprise license but don't worry - Enterprise license is free [now](https://blogs.oracle.com/java/post/graalvm-free-license) so you do not need to pay at least for a license if you want to use PGO in Java. Good? Great!

TODO: write about the curently weak GraalVM support across the Java ecosystem.

### Swift

It's a complicated question. From one perspective, in the Swift compiler sources [there are](https://github.com/apple/swift/blob/main/include/swift/Option/Options.td#L1322) some PGO footprints. From another - even the compiler developer is [not sure](https://github.com/apple/swift/issues/69227) about the current PGO implementation state in the compiler. Anyway, there is an **unanswered** [topic](https://forums.swift.org/t/several-questions-regarding-profile-guided-optimization-pgo-and-llvm-bolt/67963) on the Swift forum. Hopefully, one day it will get an answer.

### Zig

TODO: write a story about a new Zig compiler and how it affects PGO dreams in this language

### Dart

TODO: add information about Dart: https://github.com/dart-lang/sdk/issues/53855 in `dart2native` mode and https://github.com/dart-lang/sdk/issues/53856

### Nim

TODO: has no support https://github.com/nim-lang/RFCs/issues/130

## PGO support in build systems

* Overview of the PGO state across build systems
  - Add a note about Bazel integration
  - Add links to the issues in other build systems like CMake, Meson, etc. (maybe I need to check it across all popular build systems and create proper issues for that)

In 99.9(9)% cases we do not compile our applications via direct compiler invocations - we use build systems. What kind of PGO support could we expect from a build system? I have the following wishes:

* I want to have a possibility to enable PGO for my application via *a build system flag* rather than *a compiler flag*. Why? Because I can use multiple compilers, each compiler can have different flags for enabling PGO (e.g. check the difference in PGO in MSVC and Clang). And writing such logic for each compiler is not what I want to do for each project where I want to enable PGO.
* Optimizing with PGO whole dependency tree, not only my project. Almost any modern application uses dependencies. And for making things even more complicated, this dependencies can be written in different languages, that are compiled by different compilers (with different PGO flags, as we know). In this case, enabling PGO build for the whole dependency tree is quickly becomes a non-trivial task.
* It should work.

### Cargo

Cargo (the default build system for Rust) has no built-in support for PGO. But community (particularly [Jakub "Kobzol" Beranek](https://github.com/kobzol)) developed an extension - [cargo-pgo](https://github.com/Kobzol/cargo-pgo). I highly recommend you to use this Cargo extension if you are going to start optimizing Rust projects with PGO. I used for **every** Rust project that I PGOed (and I did it for a lot of them) - it always worked like a charm. It even supports optimizing with LLVM BOLT as an additional post-PGO optimization step! I wish every other ecosystem eventually will get something similar.

TODO: add a note about missing Sampling PGO optimization integration

### Bazel

Bazel [has](https://bazel.build/reference/command-line-reference#flag--fdo_instrument) built-in PGO support. With provided by Bazel command-line options, you will be able to optimize your program with PGO without needing to know of how to properly invoke PGO stuff in your favourite compiler.

TODO: add a note about Envoy, Bazel and PGO

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

Here I collected some examples of real-life project that use this approach for doing PGO:

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


TODO: add project where this approach is used

### Production environment

Advantages:

* Production is the most representative environment to get the actual workload profile.
* There is no need to maintain custom near real life workloads, sync them with the actual production workload, monitor a skew between the sample workload and the production, etc.

Disadvantages:

* It can be difficult to collect PGO profiles from external environments. E.g. let's imagine you have a mobile application. It's much more difficult to collect PGO profiles from consumer devices and transfer them to your CI environment for PGO optimization.
* Getting PGO profiles from production affects performance of your production. So do it carefully. It can be partially mitigated by using sampling PGO approach since with sampling you have an option to tweak the performance overhead via increasing/decreasing the sampling frequency ("Sampling overhead <-> Profile precision" tradeoff). Actually, instrumented PGO also can be used in production but before this you need to test/estimate the performance penalty and decide - is it acceptable in your case or not.

Especially for this case, Google created [AutoFDO](https://github.com/google/autofdo) project. Google has a corresponding internal infrastructure for collecting PGO profiles directly from production environment with almost no overhead.

TODO: Write about caveat of how to collect PGO profiles from the customer devices, overhead of taking a profile from the production, etc.
TODO: add somewhere link to "PGO at scale" in Google

## Post-Link time Optimization (PLO)

TODO: Write about BOLT, PROPELLER, and others (like Dynamic BOLT)

### BOLT

TODO: write about BOLT advantages and disadvantages
TODO: write about BOLT instrumentation slowdown like https://github.com/qarmin/czkawka/issues/1099

| Application | BOLT Instrumentation to Release slowdown ratio | Benchmark link |
|---|---|---|
| czkawka | 2.66x | [link](https://github.com/qarmin/czkawka/issues/1099#issue-1946612100) |
|  |  |  |

### Propeller

TODO: Do I have any benchmarks? I guess putting from the official Google papers is fine here

### Other approaches

TODO: write here about Dynamic BOLT and other ongoing researches in this area

## LTO, PGO, PLO and proprietary software

Everyday we use not only open-source software, that can be (simply) optimized with LTO, PGO, PLO and other fansy stuff by our own hands in a Gentoo-style but also some proprietary software where we have no such an option. And still proprietary software performance can be valuable for us: CI performance with closed-source compilers, IDE performance, closed-source database performance (in both on-premise and SaaS versions) - there are many of them. So what could we do instead?

I have tried to test a silly thing: write directly to the companies with an idea of optimizing their software with PGO. Here are some results:

* Sent an email to <opensource@apple.com> about enabling PGO for Apple products like Apple Clang, prebuilt FoundationDB, etc - no response.
* Created the [topic](https://forums.percona.com/t/profile-guided-optimization-pgo-for-databases/24035) on the official Percona forum about optimizing its products with PGO - no response
* Sent an email to Nvidia via Developer Contact email about enabling PGO for their HPC compilers with all required information about PGO. Only got a suggestion about creating a topic on their HPC forum. I have tried multiple times to create an account on this resource but with no success due to unknown for me reasons. No more responses from them by email, btw.
* Sent an email to [NauEngine](https://nauengine.org/) devs about evaluating PGO for this game engine - **got a response** that PGO information was sent to development teams. Great!

TODO: write a section how I was ignored by different companies with an idea of PGO and other stuff
TODO: write several advices/thoughts of how users can try to push PGO for their products being a users AND a developer of such products

* PGO and proprietary software
  - ArenagraphDB just didn’t answer my request - maybe an email wasn’t enterprise-enough? :)
  - NauEngine: sent an email about PGO (not response yet)

## Why am I writing this?

* I like performant applications
There are multiple reasons for that:
* For me it would be easier to work in the industry, where we have "PGO by default" mindset. Because with faster software it's easier to achieve required NFR (Non-Functional Requirements) before the horizontal scaling questions.
* Because I can
* Just for lulz

## Software pipeline

Let's take a look on how looks like a default **open-source** software pipeline:

TODO: insert a diagram here

## Beyond PGO

TODO: write about BOLT, Propeller, its history and perspectives

What else could we do beyond PGO? Well, the optimization industry has an answer - it's called Post-Link time Optimization (PLO)

## Test environments

### Hardware and OS

TODO: add a photo of my Mac with my PC test setup

During the tests, I use two test setups: my Windows-Linux dual-boot PC and Macbook with macOS.

PC:

* CPU: AMD Ryzen 9 5900x
* RAM: 48 Gib of some default non-RGB RAM
* SSD: Samsung 980 Pro 2 Tib
* OS: Fedora 38
* Kernel: Linux kernel 6.x (mostly 6.2 - 6.5)

Macbook:

* CPU: Apple M1 Pro 6+2
* RAM: 16 Gib
* SSD: 512 Gib
* OS: macOS (mostly 13.x version)

In the article my PC setup is called "Linux", and Macbook as "macOS". Sorry Windows people, I have no desire to perform all the tests on Windows. But anyway - it should work (and works) on this OS completely in the same way.

### Compilers

For all Rust projects I use `rustc` compiler with LLVM backend, versions somewhere in 1.68-1.73 range. For C and C++ I prefer using Clang 16, sometimes I use GCC 13 as an additional compiler.

All these compilers are in their "default" state with no custom patches from my side: `rustc` from the official website, Clang and GCC from the official Fedore repository.

## FAQ

### Do I need to use LTO with PGO?

No! Link-Time Optimization (LTO) and PGO are completely independent and can be used without each other.

However, usually LTO is enabled before PGO. Why it happens? Because both LTO and PGO are optimization technique with an aim to optimize your program. And usually enabling LTO is much easier than PGO because enable one compiler/linker switch with LTO is an easier task than introduce 2-stages build pipelines with PGO. So please - before trying to add PGO to your program try to use LTO at first.

There are some situations, when you may want to avoid using LTO with PGO:

* Weak build machines. LTO (even in ThinLTO mode) consumes a large amount of RAM on your build machines. That means if your build environment is highly memory-constrained - you may want to use PGO without LTO since PGO usually has lighter RAM requirements for your CI.
* Compiler bugs. Sometimes PGO does not work for some reason with LTO (TODO: add Rust compiler bug issue here)

## Related projects

TODO: write about ASOS, machine-learning based compilers, etc.

## Awesome PGO people

If you decide to integrate PGO, probably at some point you will meet some problems, a corner case or find a nice way to improve PGO for your specific workload. So here I want to share the PGO experts list (not complete!). I believe these people could help you in your PGO journey:

* Jakub Beranek
* Maxim Panchenko
* Amir Aupov

## Future plans

## Helpful/interesting links to know about

TODO: Finish links section

Here I collected a bunch of PGO and optimization related links that I can recommend to read:

* [Awesome PGO](https://github.com/zamazan4ik/awesome-pgo). It's my repository, where I collect **everything** about PGO: benchmarks for different software, bugs in compilers, tracking PGO issues in multiple projects, etc. If anyone asks you something like "Hi ${PERSON_NAME}! Could you please send me a link to dive into PGO?" - it should the first appeared in your mind link to share! From my side, I will try to maintain it on the corresponding quality level (at least for some time).
* [PGO in details in Go](https://theyahya.com/posts/go-pgo/). I think it would be interesting to read for Go people.

TODO: note about MSVC compiler and combined LTO/PGO mode as the only option to enable PGO.

TODO: Possible plot for the talk:

1. Why did I write this post? Intro why I like fast software, etc.
2. Show, why PGO is a worth thing to discuss with performance numbers for the most famous projects like PostgreSQL, Clang, MongoDB, 
2. PGO current ecosystem state from multiple perspectives: languages, compilers, problems, etc.
3. What it can be in the future?