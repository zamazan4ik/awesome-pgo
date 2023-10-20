# Profile-Guided Optimization (PGO): a long hard road out of hell

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

In general, PGO has two types:

* Instrumentation-based
* Sampling-based

Let's take a look on each.

#### PGO via instrumentation

Right now, instrumentation mode is the de-facto default PGO mode in many cases. So if you here somewhere something about PGO, in 99% it is instrumentation PGO. But how does it work?

Usually, the scenario is the following:

* Compile your application in instrumentation mode
* Run the in

#### PGO via Sampling

TODO: add info about Sampling PGO

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

* HAProxy: ([Clang](https://github.com/haproxy/haproxy/issues/2047#issuecomment-1728265165) and [GCC](https://github.com/haproxy/haproxy/issues/2047#issuecomment-1729606775))
* Fluent-bit: ~50% slowdown ([link](https://github.com/fluent/fluent-bit/discussions/6638#discussioncomment-6419880))
* ClickHouse: 
* Tarantool: ([link](https://github.com/tarantool/tarantool/issues/8089#issuecomment-1580628168))

TODO: write about cases when an instrumented binary is faster (I met such cases in some strange situations. Probably some combination of compiler optimizations does this - who knows, didn't investigate deep such cases).

#### Instrumented binary is larger

TODO: add several examples of such binaries (can look at my local left binaries)

#### Build issues with instrumentation PGO

TODO: add Envoy example

### Sampling PGO problems

TODO

### Common PGO problems

TODO: like building with PGO in multi-lang apps and passing PGO info to the dependencies.

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

## FAQ

### Do I need to use LTO with PGO?

No! Link-Time Optimization (LTO) and PGO are completely independent and can be used without each other.

However, usually LTO is enabled before PGO. Why it happens? Because both LTO and PGO are optimization technique with an aim to optimize your program. And usually enabling LTO is much easier than PGO because enable one compiler/linker switch with LTO is an easier task than introduce 2-stages build pipelines with PGO. So please - before trying to add PGO to your program try to use LTO at first.

There are some situations, when you may want to avoid using LTO with PGO:

* Weak build machines. LTO (even in ThinLTO mode) consumes a large amount of RAM on your build machines. That means if your build environment is highly memory-constrained - you may want to use PGO without LTO since PGO usually has lighter RAM requirements for your CI.
* Compiler bugs. Sometimes PGO does not work for some reason with LTO (TODO: add Rust compiler bug issue here)

## Related projects

TODO: write about ASOS, machine-learning based compilers, etc.

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