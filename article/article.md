# Profile-Guided Optimization (PGO): the Way

TODO: what about the "Life and Suffering with Profile-Guided Optimization (PGO)" name for the article?

In this article, I want to discuss one compiler optimization technique - Profile-Guided Optimization (PGO). We will talk about PGO pros and cons, obvious and tricky traps with PGO, and take a look at the current PGO ecosystem from different perspectives (programming languages, build systems, operating systems, etc.). I spent many days applying PGO to different applications... Tweaking build scripts, many measurements, sometimes tricky debug sessions, several compiler bugs and much more - it was (and still is) a great journey! I think my experience would be helpful for other people so you will be able to avoid my mistakes from the beginning and much nicer optimization experience.

The article is written in a story style. So the information about PGO is mixed with a lot of examples (because I like examples from the real world) and my thoughts regarding multiple aspects PGO and PGO-related aspects. Also, I prepared a lot of inline links if you are interested in deeper details in certain topics so don't hesitate to click on them!

The article is huge enough, so do not forget to grab some tea, beer or something like that before the begginning.

## Target audience

I think the article would be interesting for anyone, who cares about software performance. Especially it should be interesting for people who want to extract more performance from applications without tweaking the application itself.

Do not expect "hardcore" here. I am a usual software engineer, (un)fortunately not even a compiler engineer! And because of this I am sure my point of view could be interesting here - how look like advanced (IMHO) compiler optimizations techniques from the average software engineer side.

## Disclaimer about the author

Most of my experience is with "native" technologies like C++ (it was my main language for many years), a bit of pure C, and Rust (my current favorite toy). Right now I don't even write code for production for almost 2 years (what a shame). They call me an architect but I prefer a "Confluence & Draw.io engineer".

So when we talk about C/C++/Rust-related things, you can expect some more advanced information. I try my best to cover other technologies with PGO like Go, C#, Java, etc. but do not expect a lot of deep details here. However, at the end I put a bunch of helpful links for all technologies - hopefully, it would help you.

Also, since I am not a compiler engineer, I cannot tell you much about PGO compiler implementation details. But I know people who knows about it a lot! I will try to reference all these awesome engineers too, and (hopefully) you will not be alone if you want to dig into the compiler internal details. I left some anchors to PGO-related parts of different compilers for the most interested people!

## Attention anchor

Many of the technology-related people like numbers, especially when we are talking about performance. Since I want to get your attention to the topic of how PGO helps in real life, I will show you at the very beginning some performance numbers with links to the actual benchmarks for the most famous (FMPOV) open-source projects.

(just for reference: `QPS` - Queries Per Second, `TPS` - Transactions Per Second, `RPS` - Requests Per Second, `EPS` - Events Per Second)

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

Of course, it's not a complete list - much-much more PGO showcases you can check right now [here](https://github.com/zamazan4ik/awesome-pgo#pgo-showcases). If you are interested - let's begin our PGO journey together!

## Intro

I like performant applications. I like when software doesn't make me wait for results for a long time. Faster software compilation, faster OS boot time, faster `grep`-ing over hundreds of gibibytes of logs - I appreciate all of that.

There are multiple where performance can be useful for you too (the list can be extended):

* Infrastructure cost reduction (could be valuable at large scales)
* Improving CI speed (faster Time-To-Market, cheaper infrastructure, better developer experience with build pipelines)
* Better developer experience during the local development (quicker write-evaluate loop)
* Better energy efficiency (your smartphone and laptop will say "Thank you")
* Limited hardware availability (wrong [CAPEX](https://en.wikipedia.org/wiki/Capital_expenditure) estimations, no budget for the new hardware or even no possibility to buy new hardware due to multiple other reasons)
* Better startup time for your application. I hear this problem mostly from the Java world (and seems like it's one of the main GraalVM selling points but we will discuss this tool later). Faster launch -> shorter deployment windows -> shorter [RTO](https://csrc.nist.gov/glossary/term/recovery_time_objective) -> easier to achieve your [SLO](https://cloud.google.com/blog/products/devops-sre/sre-fundamentals-slis-slas-and-slos) requirements. Another use case - serverless applications where cold start time can be a [real problem](https://maxday.github.io/lambda-perf/).

Even if we are talking about "large" systems, having better efficiency per core could be important since, in this case, you need to implement horizontal scaling for your app later and postpone investing your time/money into things like sharding, load balancing and other cool (and difficult) stuff.

As a regular human, I have a built-in (de)buff - I am lazy. That's why I don't like to perform manual optimizations. Profiling, finding a bottleneck, trying to figure out what is going on in this bottleneck, thinking and implementing an optimization - too much things to do for a lazy engineer. I need something easier to achieve better performance, ideally without me at all.

Performing optimizations is one of the most important tasks for modern optimizing compilers. So we need a simple thing - tweak our compilers to perform more and more aggressive optimizations. In this case, we can postpone the moment, when the compiler will need help from a human. That's why at university (several years ago) I decided to invest my time into a compiler research project.

There are two major ways of how our software can be compiler: Ahead-of-Time ([AOT](https://en.wikipedia.org/wiki/Ahead-of-time_compilation)) and Just-in-Time ([JIT](https://en.wikipedia.org/wiki/Just-in-time_compilation)) (let's leave interpretation, transpilation, etc. out of the scope - we don't care about them much in this article).

In AOT world, we once compile our source code to a native binary, then we run this binary on all target machines without further modifications. Examples of such compilers are GCC, Clang, Rustc, GraalVM, etc.

In JIT world, the source code usually is compiled to some intermediate representation, transferred to target machines, and then on these machines it's compiled to the native code, optionally using **runtime statistics** for performing better optimization decisions. Examples of such systems are modern Javascript engines like V8.

Good modern compilers nowadays perform many [optimizations](https://learn.microsoft.com/en-us/cpp/build/profile-guided-optimizations?view=msvc-170#optimizations-performed-by-pgo) like:

* **Inlining**
* Loop roll/unroll
* Devirtualization
* Hot/cold code splitting
* Link-Time Optimization (LTO) (if you are brave enough to use it! Just joking - it works good-enough nowadays)
* And many other funny things!

Inlining is a special thing here since it influences a lot on other compiler optimizations since it gives a *context* to other optimizations - that's why it's so special.

Each compiler has something like an internal optimization model of a *generic* application. Multiple hardcoded constants like [inlining thresholds](https://github.com/llvm/llvm-project/blob/main/llvm/lib/Analysis/InlineCost.cpp#L57), [LTO constants](https://issues.chromium.org/issues/40219076) and similar stuff - that's it. The caveat here is that these constants are made/calculated for a generic problem but not for your specific use case. A good JIT compiler can avoid such a limitation via collecting all required information about your workload and tune these constants dynamilcally on a target machine. AOT compiler has no such ability since it doesn't work on a target machine. And it's a problem since we miss huge optimization opportunity with AOT.

Profile-Guided Optimization (PGO) is an attempt to eliminate this gap between JIT and AOT worlds. With PGO we do a simple thing: collect runtime statistics about how our application is executed in runtime, and pass this information to a compiler. The compiler uses this information for doing **data-driven** optimization decision instead of guessing (there are a lot of guessing inside compilers - trust me). Since we runtime statistics from actual workloads we can expect performance boost from this optimization technique.

Just a note. In this article I will cover PGO for AoT compilation model. Good JIT compilers also could have some kind of PGO built-in to collect and use runtime statistics. However, I don't have enough experience with JIT-based technologies. From this point of the article we will discuss specifically PGO for AoT world. Later, we will quickly discuss why using JIT is not a silver bullet and why we still need AoT with PGO.

## What is PGO?

Let's chat about PGO with a true expert in everything - Gigglesome Punster Texterator aka GPT.

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

**Me**: Thanks, buddy!

Sounds quite simple, doesn't it? I was young, so such a simple explanation (even not from non-existing at that moment Gigglesome Punster Texterator) bought me, and I decided to dig a little bit deeper. If according to the multiple articles PGO helps with optimizing software, why we don't use it everywhere right now?

By the way, one statement from The Expert is quite interesting - "Profile-Guided Optimization is particularly effective in scenarios where the program's execution behavior varies across different input data or usage patterns". We will discuss later why different workloads actually can become a problem for PGO (resolvable problem but with some efforts) - stay tuned.

Let's check an example. For a simple C++ function:

```cpp
bool some_top_secret_checker(int var)
{
   if (var == 42) return true;
   if (var == 322) return true;
   if (var == 1337) return true;
   return false;
}
```

with regular Release (`-O3`) flag with Clang 17 you will get something like:

```
mov	al, 1
cmp	edi, 42
je	.LBB0_4
cmp	edi, 322
je	.LBB0_4
cmp	edi, 1337
je	.LBB0_4
xor	eax, eax

.LBB0_4:                               
ret
```

If we apply PGO and "train" our compiler on data where with higher frequency `322` appears as an input variable, we will get a bit different assembly:

```
mov	al, 1
cmp	edi, 322
jne	.LBB0_1

.LBB0_4:                                   
ret

.LBB0_1:
cmp	edi, 42
je	.LBB0_4
cmp	edi, 1337
je	.LBB0_4
xor	eax, eax
jmp	.LBB0_4
```

Here we see some changes in the resulting code. The most interesting change is how branch orders is changed. Since `322` frequently appeared as an input variable, compiler decided to move the corresponding branch to the beginning of the function. This decision has simple logic: as earlier we check for the most probable value - the less redundant instructions we execute.

Second interesing is how `ret` location is changed. Instead of leaving it at the end of the function, after PGO the return statement was moved to the beginning of the function. It's done for improving the CPU instruction cache hit-rate: since after the `322` branch with high probability will be a return from the function, it makes sense to store them together. Pretty clever decision!

By the way, there is an interesting [difference](https://users.rust-lang.org/t/differences-in-an-optimization-decision-between-rustc-and-clang/112593) between Clang and Rustc (both are LLVM-based compilers) from the code generation perspective in this case: Rustc for some unknown yet reason swaps 2nd and 3rd branches, Clang doesn't do it. You could try to figure out the root cause for such a difference ;)

Let's dig into PGO deeper!

### PGO and other similar names

The first funny thing that I found - PGO has multiple names! They all mean **completely** the same thing:

* Profile-Guided Optimization (PGO)
* Feedback-Directed Optimization (FDO)
* Profile-Driven Optimization (PDO)
* Profile-Based Optimization (PBO)
* Profile-Directed Feedback (PDF) (IBM AIX [documentation](https://www.ibm.com/docs/en/aix/7.3?topic=techniques-profile-directed-feedback), please don't try to google with this abbreviation :)
* Profile Online Guided Optimization (POGO) (I found at least one [post](https://www.reddit.com/r/rust/comments/jwad2g/new_experiment_pogo_online_pgo_for_functions/) on Reddit)

At most places, you will see PGO (sometimes people pronouns it as "pogo") (like in Clang, GCC, Rustc documentations). Sometimes, FDO is used (e.g. Bazel and many tools/articles from Google). Other names almost never used. In this article, I use only PGO abbreviation as the most known name for this optimization technique.

Quickly I found another issue: there are multiple meanings for PGO:

* [PostrgreSQL Operator](https://github.com/CrunchyData/postgres-operator). This one is the most annoying since Google shows it often (at least in my queries). I recommend resolving it with excluding "postgresql" word from the results.
* PGO [company](https://pl.wikipedia.org/wiki/PGO_(polskie_przedsi%C4%99biorstwo)). I do not think they are Profile-Guided Optimization gurus but who knows...
* Ponto-geniculo-occipital waves or [PGO waves](https://en.wikipedia.org/wiki/PGO_waves). I tried to understand what it is but quickly gave up - too boring stuff (IMHO).
* Pokemon GO (as The Expert said above).

Not critical at all but keep in mind this information during your Googling/Binging/DuckDuckGoing/whatever sessions.

### PGO types

PGO has two major types:

* Instrumentation-based
* Sampling-based

To make things even more complicated - there are a bunch of hybrid approaches as well. We will check them a bit later. Right now let's take a deeper look at the most common PGO type - Instrumentation PGO.

### Instrumentation PGO

Right now, instrumentation mode is the de-facto default PGO mode in for almost all PGO implementations (except the Go compiler). So if you hear somewhere something about PGO, in 99% of cases, it is the instrumentation PGO. How does it work?

Usually, the scenario is the following:

* Compile your application in the Instrumentation mode.
* Run the instrumented application on your typical workload.
* Collect runtime statistics about how the application is executed (which parts of it are executed, how frequently, etc.) This information known as PGO profiles.
* Recompile the application once again with the PGO profiles.
* ...
* Profit!

What does "instrumentation" mean in reality? Let's check in a simple example.

For a simple C++ function:

```cpp
bool is_meaning_of_life(int var)
{
   return var == 42;
}
```

with regular `-O3` flag and Clang you get something like that in assembly (I prefer the Intel syntax btw):

```
cmp     edi, 42
sete    al
ret
```

However, things are changing a bit when the instrumentation is enabled (in this case with `-O3 -fprofile-generate` switches for Clang. Some redundant instructions are deleted for clarity):

```
inc     qword ptr [...]
cmp     edi, 42
sete    al
ret

__llvm_profile_raw_version:
.quad   72057594037927944

__llvm_profile_filename:
.asciz  "default_%m.profraw"
```

Here we see some additional extra instructions. At first, we see `inc     qword ptr [...]` instruction. It's an automatically inserted by the compiler **counter** into the function. Each time when the function is executed, the counter is increased. At the exit of the application counters from whole applications (for large applications it can be thousands of counters) are dumped to a disk with `__llvm_profile_filename` filename (since we use Clang. Other compilers [do similar things](https://godbolt.org/z/1Yxnso6e6) but with different machinery). `__llvm_profile_raw_version` is an additional machinery for a compiler. Since the PGO profile by default is saved to a disk, the file should have some format, and this format also has some versioning. We will discuss these topics in details a bit later.

In LLVM, there are several instrumentation PGO subtypes:

* FrontEnd PGO (FE PGO)
* IR PGO
* Context-Sensitive IR PGO (CSIR PGO or CS IRPGO - both variants are fine)

A big "Thank you!" to [Amir](https://twitter.com/disruptnhandlr) for his excellent [write-up](https://aaupov.github.io/blog/2023/07/09/pgo) about this topic!

#### FrontEnd PGO (FE PGO)

We need to understand (just a bit!) the internal LLVM [architecture](https://aosabook.org/en/v1/llvm.html). It consists of three main parts:

* Frontend: where your language is parsed from a source to a intermediate representation (LLVM IR). Each programming language has a dedicated frontend (like Clang, Rustc, LDC, etc.)
* Middle-end (aka "the optimizer"): where almost all compiler optimizations are performed on LLVM IR.
* Backend - where the LLVM IR is converted to the machine code is generated (and some more low-level optimizations are also performed).

Of course I omitted a lot of interesting details but we don't need them here. Some compilers can have fewer internal parts, some of them - more, actually doesn't matter for our case.

Knowing it, FE PGO approach is simple. The compiler during the compilation process automatically inserts counters and other PGO-related things on the frontend part, before translating to the LLVM IR. A bit more information about this approach you can read [here](https://clang.llvm.org/docs/SourceBasedCodeCoverage.html). As far as I know, Clang is the only compiler (at least the only open-source compiler) that implements this approach. In Clang this type of PGO is enabled with `-fprofile-instr-generate`/`-fprofile-instr-use`.

Despite source-based coverage being widely used for code coverage calculations and similar stuff, for PGO nowadays [it's not a recommended](https://github.com/llvm/llvm-project/issues/45668) approach anymore. All current investments into the PGO infrastructure in LLVM are done into IR PGO-based approaches (described below). Some deeper details can be found [here](https://discourse.llvm.org/t/couple-of-general-questions-about-pgo/72279) and [here](https://discourse.llvm.org/t/status-of-ir-vs-frontend-pgo-fprofile-generate-vs-fprofile-instr-generate/58323).

#### IR PGO

The idea is completely the same as with FE PGO but instead inserting PGO things on the frontend side, we insert them on the middle-end. What does it change from the user perspective? Nothing. What does it change from the compiler perspective? I don't know enough LLVM implementation details so I guess here. At least inserting PGO things on the middle-end can help to unify PGO infrastructure and make it reusable across multiple languages: you don't need to implement PGO instrumentation code on each frontend, instead we do most things once on the middle-end. On the frontend side usually you need to pass only a few things.

Clang implements this approach with `-fprofile-generate`/`-fprofile-use` options. This is the recommended way to use instrumentation PGO with Clang. AFAIK, GCC implements a similar approach with its [GIMPLE](https://gcc.gnu.org/onlinedocs/gccint/GIMPLE.html) representation.

#### Context-Sensitive IR PGO (CSIR PGO)

The idea with CSIR PGO is completely the same as with IR PGO with one difference: the PGO machinery is inserted **after** the inlining phase - with IR PGO it's done **before** the inlining phase. If inlining is performed after the instrumentation PGO, it can affect negatively PGO profiles precision, that consequently negatively affects PGO optimization efficiency.

I collected some benchmarks regarding applying CSIR PGO in practice: [one](https://issuetracker.google.com/issues/200085858#comment5), [two](https://issues.chromium.org/issues/40147394), [three](https://issuetracker.google.com/issues/223669638). During my PGO benchmarks I tried to use CSIR PGO several times and didn't find the performance difference in practice between IR PGO and CSIR PGO - maybe I was too unlucky, who knows. You also can raise the question: if we have CSIR PGO why do we need IR PGO at all? The answer is still [unclear](https://discourse.llvm.org/t/profile-guided-optimization-pgo-related-questions-and-suggestions/75232/21).

My personal recommendation: start with the regular IR PGO approach first. Only after integrating IR PGO you can try to additionally apply CSIR PGO and measure the performance difference. If it works for you - great, use IR PGO + CSIR PGO. If it doesn't - well, don't waste your time with CSIR PGO.

More about CSIR PGO you can watch at "2020 LLVM Developers’ Meeting: “PGO Instrumentation: Example of CallSite-Aware Profiling”" ([Youtube](https://www.youtube.com/watch?v=GBtQrYx_Jbc)).

### Instrumentation PGO: problems

* Instrumentation PGO: problems

  - Exit-code issues - for some applications in the end the profile is not dumped (like Clangd). You need to tweak the application manually

#### An instrumented binary is slower

An instrumented binary is slower. But how much? Well, as usual - *it depends*. I didn't find such benchmarks for real-life applications so I did them and I'm ready to show you some numbers for several projects:

| Application | Instrumentation to Release slowdown ratio | Benchmark |
|---|---|---|
| HAProxy (Clang) | 1.10x - 1.20x | links ([one](https://github.com/haproxy/haproxy/issues/2047#issuecomment-1729971279), [two](https://github.com/haproxy/haproxy/issues/2047#issuecomment-1728265165)) |
| HAProxy (GCC) | 1.24x | links ([one](https://github.com/haproxy/haproxy/issues/2047#issuecomment-1729971279), [two](https://github.com/haproxy/haproxy/issues/2047#issuecomment-1729606775)) |
| Fluent-Bit | 1.48x | [link](https://github.com/fluent/fluent-bit/discussions/6638#discussioncomment-6419880) |
| ClickHouse | up to **311.0x** (it's not a mistake!) | [link](https://github.com/ClickHouse/ClickHouse/issues/44567#issuecomment-1519136060) |
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
| vtracer | ~2.0x | [link](https://github.com/visioncortex/vtracer/discussions/67#discussion-5889368) |
| Symbolicator | 1.19x | [link](https://github.com/getsentry/symbolicator/issues/1334#issue-2016488888) |
| sage | ~1.8x | [link](https://github.com/adam-mcdaniel/sage/issues/70) |
| lace-cli | ~1.23x | [link](https://github.com/promised-ai/lace/issues/172#issue-2104154605) |
| candy vm | ~1.5x | [link](https://github.com/candy-lang/candy/discussions/953#discussion-6270541) |
| angle-grinder | ~1.6x | [link](https://github.com/rcoh/angle-grinder/discussions/203#discussion-6583612) |
| bbolt-rs | ~1.64x | [link](https://github.com/ambaxter/bbolt-rs/issues/2#issue-2291476885) |
| Bend | ~1.5x | [link](https://github.com/HigherOrderCO/Bend/issues/517#issue-2319505187) |
| resvg | ~1.2x | [link](https://github.com/RazrFalcon/resvg/issues/765#issue-2314946307) |

The same applies to libraries as well:

| Library | Instrumentation to Release slowdown ratio | Benchmark |
|---|---|---|
| tantivy | 1.49x | [link](https://github.com/quickwit-oss/tantivy/issues/2163#issue-1872682955) |
| tonic | up to 1.55x | [link](https://github.com/hyperium/tonic/issues/1486#issue-1871968898) |
| quick-xml | up to 2.5x| [link](https://github.com/tafia/quick-xml/issues/632#issue-1852200475) |
| xml-rs | 1.45x | [link](https://github.com/netvl/xml-rs/issues/228#issue-1852007193) |
| roxmltree | ~2.5x | [link](https://github.com/RazrFalcon/roxmltree/issues/118#issue-2295151269) |
| sxd-document | ~2.0x | [link](https://github.com/RazrFalcon/roxmltree/issues/118#issue-2295151269) |
| xmltree-rs | ~1.4x | [link](https://github.com/RazrFalcon/roxmltree/issues/118#issue-2295151269) |
| tquic | up to 1.3x | [link](https://github.com/Tencent/tquic/issues/19#issue-1972443474) |
| lingua-rs | up to 146x (not an error!) | [link](https://github.com/pemistahl/lingua-rs/discussions/273#discussion-5864708) |
| tokenizers | up to 20x | [link](https://github.com/huggingface/tokenizers/issues/1426#issue-2069318532) |
| zen | up to 4.25x | [link](https://github.com/gorules/zen/discussions/109#discussion-6053432) |
| native_model | up to 95x | [link](https://github.com/vincent-herlemont/native_model/issues/50#issue-2073664067) |
| NativeDB | up to 5.7x | [link](https://github.com/vincent-herlemont/native_db/discussions/92#discussion-6053050) |
| redb | up to 4.5x | [link](https://github.com/vincent-herlemont/native_db/discussions/92#discussion-6053050) |
| pathfinding | up to 3.5x | [link](https://github.com/evenfurther/pathfinding/issues/510#issue-2073786817) |
| minitrace-rust | up to 14x | [link](https://github.com/tikv/minitrace-rust/issues/195#issue-2104485855) |
| needletail | up to 3x | [link](https://github.com/onecodex/needletail/issues/72#issue-2127271081) |
| logos | up to 1.2x | [link](https://github.com/maciejhirsz/logos/issues/374#issue-2127942609) |
| varpro | up to 1.3x | [link](https://github.com/geo-ant/varpro/issues/27#issue-2129147949) |
| prettyplease | 1.3x | [link](https://github.com/dtolnay/prettyplease/issues/74#issue-2292685589) |

I could prepare more advanced analyses like slowdown p50/p95/p99 percentiles, and test across more different configurations (like trying to replicate tests across different hardware/software, etc) but I am a bit lazy about doing it right now. Maybe next time ;)

Generally speaking, there is no way to predict how slow your binary will be after the instrumentation phase since it depends on a myriad of factors: the number of branches in your code, where your program spends the most time, training workload, etc. So in practice the only working solution to predict the slowdown is just benchmarking. Maybe one day such a tooling will be available but not today.

Fun fact: *sometimes* your application **faster** after applying instrumentation! I met such behavior several times during my PGO tests for various applications. I didn't investigate such cases deeply (since it requires digging into assembly code that can be non-trivial for large applications). I can only guess here that such an "improvement" is possible due to initially non-optimal optimization decision in the Release mode. With inserting additional instrumentation routines into the code, optimization decisions made by a compiler can become a bit different, and sometimes you are lucky enough that such decision changes lead to better performance. Of course, you shouldn't rely on it in practice!

#### An instrumented binary is larger

As we've discussed above, Instrumentation PGO works by inserting into your code some counters for tracking the program execution characteristics, so your binary will be larger after the instrumentation. How much binary space does it take in practice? Let's check it (all tests are done on the Linux machine, for C and C++ applications by default Clang is used, for Rust - Rustc):

| Application | Release size | Instrumented size | PGO optimized size | Instrumented to Release size ratio | Language |
|---|---|---|---|---|---|
| ClickHouse | 2.0 Gib | 2.8 Gib | 2.0 Gib | 1.4x | C++ |
| MongoDB | 151 Mib | 255 Mib | 143 Mib | 1.69x | C++ |
| SQLite | 1.8 Mib | 2.7 Mib | 1.5 Mib | 1.5x | C |
| RocksDB (db_bench) | 762 Kib | 2.0 Mib | 715 Kib | 2.62x | C++ |
| Qdrant | 56 Mib | 155 Mib | 54 Mib | ~3.0x | Rust |
| HAProxy | 13 Mib | 17 Mib | 13 Mib | 1.3x | C (Clang) |
| HAProxy | 15 Mib | 19 Mib | 16 Mib | 1.27x | C (GCC) |
| Nginx | 3.8 Mib | 4.3 Mib | 3.8 Mib | 1.13x | C |
| Envoy | 439 Mib | 1800 Mib | 433 Mib | 4.15x | C++ |
| httpd | 2.3 Mib | 2.7 Mib | 2.4 Mib | 1.17x | C |
| Quilkin | 33 Mib | 94 Mib | 30 Mib | 2.84x | Rust |
| Rathole | 3.8 Mib | 11 Mib | 3.5 Mib | 2.89x | Rust |
| bat | 5.4 Mib | 16 Mib | 5.2 Mib | 2.96x | Rust |
| Catboost | 32 Mib | 88 Mib | 30 Mib | 2.75x | C++ |
| curl | 1.1 Mib | 1.4 Mib | 939 Kib | 1.27x | C |
| czkawka | 14 Mib | 38 Mib | 14 Mib | 2.71x | Rust |
| Fluent-Bit | 7.9 Mib | 11 Mib | 6.4 Mib | 1.39x | C++ |
| Vector | 198 Mib | 286 Mib | 124 Mib | 1.44x | Rust |
| htmlq | 6.7 Mib | 11 Mib | 6.8 Mib | 1.64x | Rust |
| ouch | 3.5 Mib | 8.0 Mib | 3.3 Mib | 2.26x | Rust |
| difftastic | 68 Mib | 75 Mib | 68 Mib | 1.10x | Rust |
| slint-fmt | 3.1 Mib | 28 Mib | 3.3 Mib | 9.0x | Rust |
| tokei | 7.5 Mib | 16 Mib | 7.5 Mib | 2.13x | Rust |
| tealdeer | 7.4 Mib | 20 Mib | 7.6 Mib | 2.7x | Rust |
| qsv | 42 Mib | 81 Mib | 39 Mib | 1.93x | Rust |
| vtracer | 6.6 Mib | 13 Mib | 6.4 Mib | 1.97x | Rust |
| Symbolicator (stripped) | 31 Mib | 89 Mib | 27 Mib | 2.87x | Rust |
| HiGHS | 409 Kib | 866 Kib | 408 Kib | 2.12x | C++ |
| lace-cli | ~28 Mib | 111 Mib | ~27.5 Mib | almost the same | Rust |
| llrt | 7.2 Mib | 13 Mib | 6.8 Mib | 1.8x | Rust |
| hydra | 394 Kib | 585 Kib | 354 Kib | 1.48x | C |
| John The Ripper (`john` binary) | 6.0 Mib | 7.5 Mib | 5.6 Mib | 1.25x | C |
| jql | 3.3 Mib | 6.3 Mib | 3.4 Mib | 1.9x | Rust |
| legba | 17 Mib | 53 Mib | 15 Mib | 3.11x | Rust |
| lua | 324 Kib | 548 Kib | 329 Kib | 1.69x | C |
| pylyzer | 36 Mib | 66 Mib | 34 Mib | 1.8x | Rust |
| angle-grinder | 74 Mib | 75 Mib | 61 Mib | 1.01x | Rust |
| CreuSAT (stripped) | 748 Kib | 1.2 Mib | 643 Kib | 1.6x | Rust |
| bbolt-rs | 1.3 Mib | 3.4 Mib | 1.3 Mib | 2.6x | Rust |
| prettyplease | 1.5 Mib | 2.6 Mib | 1.6 Mib | 1.7x | Rust |
| Bend | 2.5 Mib | 6.2 Mib | 2.7 Mib | 2.5x | Rust |
| resvg | 3.1 Mib | 7.8 Mib | 4.8 Mib | 2.5x | Rust |

So in regular scenarios, you should not expect a *huge* binary size increase (however, it depends on your "huge" definition that is context-dependent). For cases like web backends disk space usually is not a problem at all since modern disks are cheap enough. In other domains like embedded with limited storage it can become a problem. Also, you can meet some very niche limitations like a [limit](https://issues.fuchsia.dev/issues/42070037) for the Zircon Boot Image (ZBI) in Fuchsia - just be ready for that.

In general, there is no way to make a *precise* prediction of how large your binary will be after the instrumentation without the actual compilation process. There are so many variables involved in this process (how many branches your application has, do you recompile with PGO your statically-linked dependencies, etc.) that much easier will be just recompile with instrumentation and check it. Maybe one day the compilers (or an ecosystem around the compiler) will provide you some estimations before the actual compilation process but not today.

PGO optimized size hugely depends on your test workflow. If you execute a larger amount of code, with a higher probability the inlining will be triggered during the optimization, and your binary due to this will be larger.

#### Compilation times and memory consumption

Since the instrumentation PGO means a double-compilation model (or even triple, if we are using CSIR PGO). Additionally, linking of an instrumented binary also [becomes](https://issues.chromium.org/issues/40121663) due to addtional amount of work for the linker. As a partial mitigation for that I can recommend to you use faster linkers like [LLD](https://lld.llvm.org/) or [mold](https://github.com/rui314/mold). All these problems create a significant load on your build pipelines - this problem is not theoretical:

* Void Linux maintainer [says](https://github.com/void-linux/void-packages/issues/39652#issuecomment-1265308710) that they do not enable PGO due to the limited build resources
* PGO is not enabled for architectures with limited build resources (like PowerPC, AIX, sometimes ARM, etc.) - as an example [check](https://build.opensuse.org/projects/openSUSE:Factory/packages/gcc12/files/gcc12.spec) (line 2414) the GCC spec file in OpenSUSE: PGO build is enabled only for several hardware arhitectures. In some cases, the only option is to use some kind of hardware emulation with QEMU since native platform simply is not available - imagine the case of using cloud CI systems like GitHub Actions where many architectures simply are not presented (or are pricey).

By the way, the memory usage by a compiler is also [increased](https://issues.chromium.org/issues/40120956) since the compiler must perform additional things. During my PGO benchmarks I met multiple times a situation when my PC was Out-Of-Memory (OOM) during the building instrumented binaries (and successfully compiled a project in a usual Release mode). So be careful: add more RAM to your build machines, reduce number of parallel build jobs. And, of course, properly configure handling the OOM situation on your machine - user-space OOM service like `systemd-oomd` (or any other) can be your friend (or just run build jobs with `systemd-run -p MemoryMax=40G` command as I did multiple times. 40 Gib is just an example!).

#### Target support

For some targets like Webassembly instrumentation PGO [can be unavailable](https://discourse.llvm.org/t/profile-guided-optimization-pgo-and-webassembly/77090) in your compiler (in this case - LLVM). Hopefully the situation [will change](https://discourse.llvm.org/t/profile-guided-optimization-pgo-and-webassembly/77090/4) in 2024. Also, you can [read](https://issues.chromium.org/issues/42203147) about experiments with enabling PGO for WASM in Chromium (with the [Liftoff](https://v8.dev/blog/liftoff) compiler).

If there are some targets without instrumentation PGO support or you know other WebAssembly compilers with PGO support - please let me know!

#### Build issues with instrumentation PGO

Sometimes a project cannot be built with instrumentation PGO due to some build issues. Firstly, I [met](https://github.com/envoyproxy/envoy/issues/25500#issuecomment-1724584679) such an issue with Envoy. When I tried to use a built-in support for PGO in Bazel, Envoy failed to compiler in the instrumentation mode with errors like "FDO instrumnentation is not supported" for the `luajit` dependency. I tried to figure out how to fix it properly but failed. So I just passed manually all required compiler flags directly to Envoy and then recompiled the project with instrumentation successfully.

Another example - a build [issue](https://github.com/rust-lang/rust/issues/119848) with Zen. Here we have many linking errors for a bindgen between Rust and NodeJS API. I didn't dig a lot into the issue - just letting you know that such problem exist. Or [this](https://issuetracker.google.com/issues/187789784#comment6) example where instrumentation PGO didn't work nicely for [libFuzzer](https://llvm.org/docs/LibFuzzer.html). Even LLVM itself sometimes [disables](https://github.com/llvm/llvm-project/commit/e6c3289804a67ea0bb6a86fadbe454dd93b8d855) building with PGO due to some linker errors!

In my experience, you should not expect many build issues with instrumentation PGO in practice but sometimes you will meet them.

#### Runtime issues with instrumentation PGO

In some cases, instrumentation PGO can even change runtime behavior of your program! It's not expected by default, however it can happen. Personally, I met such problem with YDB when during the PGO training phase some internal timeouts were triggered, and training workload failed to complete (since in the instrumentation mode YDB is much slower). More examples: V8 is (was?) [broken](https://issues.chromium.org/issues/40096239) after applying the PGO instrumentation, Chromium [started](https://issues.chromium.org/issues/40280518) flooding errors after the instrumentation, [another](https://issues.chromium.org/issues/40729923#comment6) example with V8 - runtime errors due to lifetime issues and PGO dumping logic. I suppose it's possible to find more examples but I am a bit lazy for that too. So don't be surprised if something goes wrong in runtime after applying instrumentation - shit happens.

#### PGO profiles sensitivity to other compiler switches

Let's imagine that you built your software with `-O2` flag, collected PGO profiles for this build, stored in your VCS and then used this saved PGO profiles in consequent builds. Now you think "Hmmm, let's enable LTO for my app", add `-flto` flag, run your PGO-optimized build pipeline with the already saved PGO profile and ... your application slowed down!

The reason is simple - your saved PGO profile doesn't work anymore for your program. You can wonder about the reason for that since sources are not changed. The problem here is how PGO stores runtime statistics about which parts of your application were executed.

I performed several experiments on my test machine. The idea of the experiments is simple: collect PGO profiles with instrumentation with one flags, then change some compiler switches and collect PGO profiles once again. Then measure the overlap metrics between them (I did it with `llvm-profdata overlap`). Here are the results:

* Enable LTO: **zero** overlap between profiles.
* Change `-O3` flag to `-O2`: **zero** overlap between profiles.

As you see, PGO profiles are **very** sensitive to changing compiler flags. The compiler update also with high probability will invalidate your PGO profiles since between compiler versions inside the compiler can be done many changes: new optimization can be implemented, internal optimization thresholds can be tweaked, optimization passes order can be changed (this thing also influence generated code), etc. Rule of thumb is simple - after changing **anything** related to the code generation don't forget to regenerate PGO profiles.

### Sampling PGO (aka SPGO)

As we see, Instrumentation PGO has so many problems. Some of them like instrumentation runtime overhead are so critical that people decided to implement another approach for doing PGO. Regarding compilers support, it's available [for a while](https://hubicka.blogspot.com/2015/04/GCC5-IPA-LTO-news.html) in C++ compilers: since GCC 5 and LLVM 3.5. The Go compiler also uses Sampling PGO approach (however it's avaialble only since 2023).

TODO: add info about Sampling PGO aka AutoFDO aka AFDO

By the way, Sampling PGO is often called as FDO. From my understanding, this is due to Google AutoFDO project - a special set of tools to implement Sampling PGO for GCC and LLVM-based compilers. We will cover this project a bit later - stay tuned ;)

There are different ways of how the PGO profiles can be collected for Sampling PGO. The most commonly used tool for that is Linux perf. The biggest disadvantage of it - Linux perf works only on Linux. To avoid this limitation, LLVM also [supports](https://clang.llvm.org/docs/UsersManual.html#using-sampling-profilers) reading sampling PGO profiles generated with [Intel SEP](https://www.intel.com/content/www/us/en/content-details/686066/sampling-enabling-product-user-s-guide.html) (Sampling Enabling Product) - this tools works on Windows, Linux, BSD and Android platforms (at least according to the documentation). I tried to use it during my tests but failed - unfortunately the tool [doesn't work](https://community.intel.com/t5/Analyzers/socwatch2-15-module-is-not-loaded-correctly-when-using-kernel-6/td-p/1584234) with the latest Linux kernel versions due to compilation errors and I didn't want to downgrade my Fedora setup. Hopefully, Intel engineers will resolve the issue soon. However, you need to keep this information in mind since the similar issues can appear in the future too. Does PGO profile from Intel SEP work well on all platforms with LLVM? I am not sure at all since I believe this feature was tested only wit Linux so be careful. Android [uses](https://android.googlesource.com/platform/system/extras/+/refs/heads/master/simpleperf/doc/collect_etm_data_for_autofdo.md) the Simpleperf profiler (via the `profcollectd` system [daemon](https://android.googlesource.com/platform/system/extras/+/main/profcollectd/)) for doing the same thing - I didn't test it too. From all the tools above, Linux perf is the most batlle-tested profiler for collecting PGO profiles.

More about AutoFDO you can find here:

* Original AutoFDO paper: [Google research](https://research.google/pubs/pub45290/) (some benches included)
* "Taming Hardware Event Samples for FDO Compilation" [paper](https://storage.googleapis.com/gweb-research2023-media/pubtools/pdf/36358.pdf)
* "Taming Hardware Event Samples for Precise and Versatile Feedback Directed Optimizations" [paper](https://hpctest.cs.tsinghua.edu.cn/papers/tc11.pdf)
* Sampling PGO benchmarks from the Chromium project: [link](https://issuetracker.google.com/issues/259718081)

## PGO caveats

I have read **many** articles about PGO. Unfortunately, I found almost nothing about PGO issues and difficulties in different situations. So I just collected as many as possible traps myself and want to share my experience (not traps) with you. Niels Bohr once said: “An expert is a person who has made all the mistakes that can be made in a very narrow field.”. Am I a PGO expert now, huh?

TODO: insert here a meme with Garold with pain (about all my PGO mistakes)

### Sampling PGO problems

#### Branch Stack Sampling support

Sampling PGO has special relationships with Branch Stack Sampling (BSS) (this thing also commonly called as Last Branch Record (LBR)). According to the [paper](https://hpctest.cs.tsinghua.edu.cn/papers/tc11.pdf), BSS improves sampling PGO quality in practice. However, right now I have no "BSS vs non-BSS" sampling PGO benchmarks for real applications since I have no hardware with BSS support and I am lazy for doing such benchmarks. As a rule of thumb - please use BSS with sampling if it's available in your context.

However, BSS support is still kinda [limited](https://issuetracker.google.com/issues/200184657#comment14) across the ecosystem, and support is not coherent across vendors since every CPU vendor has its own BSS implementation:

* Intel x86-64 - it's called Last Branch Record (LBR). AFAIK, since Nehalem (2008), added support to the Linux kernel - somewhen between 2010-2011.
* AMD x86-64 - it's called BRS (BRanch Sampling). Available since Zen 3 (2020), Linux 5.19 (2022). However, [not all](https://discord.com/channels/636084430946959380/930647188944613406/1175827177405698170) Zen 3 supports BRS. More can be found [here](https://lwn.net/Articles/888996/).
* ARM64 - it's called BRBE (Branch Record Buffer Extension): since ARMv9.2-A (2023), Linux 6.7-rc1 (2024). Related LWN article is [here](https://lwn.net/Articles/951316/). There is a similar umbrella feature - [Coresight](https://www.kernel.org/doc/html/v5.7/trace/coresight/coresight.html). It's available in Linux for a while. I also recommend to check the [ETM](https://developer.arm.com/documentation/ihi0014/q/Introduction/About-Embedded-Trace-Macrocells) (Embedded Trace Macrocells) feature.
* PowerPC - it's called BHRB (Branch History Rolling Buffer): since Power8 (~2013), Linux perf support is also in place. Is this feature supported by your exact CPU or not - you need to check it on your own. By the way, if you are interested in PowerPC, you can try to get remote access to a machine [here](https://osuosl.org/services/powerdev/request_powerci/). I [am not the only](https://stackoverflow.com/questions/48616075/whats-the-equivalent-of-last-branch-record-lbr-on-arm-and-powerpc) who is interested in it!
* RISC-V - it's called Control Transfer Records (CTR). Current status - [under development](https://risc-v-international.slack.com/archives/C017LD0GJ8Z/p1708861135739819?thread_ts=1708839419.127049&cid=C017LD0GJ8Z). More information can be found here: [GitHub](https://github.com/riscv/riscv-control-transfer-records), [JIRA](https://jira.riscv.org/browse/RVG-62). No estimations for when it will be implemented.
* [e2k](https://en.wikipedia.org/wiki/Elbrus_2000) (Elbrus) - an analog to LBR is supported. Unfortunately, there is not support for that in Linux perf but there are plans implement this feature in the future.
* [LoongSon](https://en.wikipedia.org/wiki/Loongson) (Chinese MIPS-based CPUs) - no public-available information at the moment.
* Other architectures (e.g. Linux [supports](https://en.wikipedia.org/wiki/List_of_Linux-supported_computer_architectures) many of them) and operating systems combinations - I don't know (please let me know!). You can try to find corresponding for an architecture person in the Linux [maintainer list](https://www.kernel.org/doc/linux/MAINTAINERS) and ask them directly. I did it multiple times and always got very valuable responses.

It's kinda funny that almost the same feature has different names in different architectures. Does anyone know the reason for that? Are they trademarked or patented? :D

More advanced information about LBR can be found in these articles: [first](https://lwn.net/Articles/680985/), [second](https://lwn.net/Articles/680996/).

As you see, Intel implemented this technology many years ago, but other vendors are trying to fill the gap only during the last years.

To make things even worse - BSS does not work with virtualization for now (LWN [article](https://lwn.net/Articles/680996/), check "Virtualization" section). Maybe some hypervisors already support LBR-related registers, maybe not - deeper investigation is required. So if your environment has a lot of virtual machines - LBR-based Sampling PGO, probably, is not available for you.

#### Tooling

SPGO approach requires a tool to convert profiler report into a compiler-compatible format. Such tools don't appear magically and need to be implemented by someone. Unfortunately, the amount of available tooling is very limited.

The default option for both GCC and Clang is Google AutoFDO ([GitHub](https://github.com/google/autofdo), [paper](https://research.google/pubs/autofdo-automatic-feedback-directed-optimization-for-warehouse-scale-applications/)). This project supports converting only Linux perf profiles into the GCOV format (GCC-compatible format) with `create_gcov` tool or LLVM-compatible format with `create_llvm_prof` tool. Unfortunately, this tool is not ideal, and has the following issues:

* Google engineers don't care much about compatibility with recent LLVM version. It means that if you want to build AutoFDO with latest LLVM version - with high probability you'll get multiple compiler errors (like [this](https://github.com/google/autofdo/issues/179) and [this](https://github.com/google/autofdo/issues/157)). In the issue tracker you will find more build-related errors. Just be ready to fix them locally. Hopefully, the issue will be resolved soon since Google [plans](https://github.com/google/autofdo/issues/140#issuecomment-1986154525) to merge AutoFDO tooling to the LLVM repo.
* AutoFDO tooling consumes ridiculous amount of memory during processing large perf profiles ([issue](https://github.com/google/autofdo/issues/162)). Since the upstream didn't provide a fix - you can try to mitigate it with reducing input profile size. How can you do it? Reduce Linux perf sample rate or just record smaller time frame of your workload - however, it's not always possible to do. Imagine the case, when your application has **really** long process that takes hours/days to complete, this process consists of multiple different steps, and you want to optimize the whole process. In this case, I can suggest to collect N Linux perf profiles, convert them with AutoFDO one by one, and then merge them into with `llvm-profdata merge` utility - it should help with the issue.
* There is a possible issue with AutoFDO profile versions in GCC. GCC [changed](https://github.com/gcc-mirror/gcc/commit/8819c82ce814a6911e2c1bfebd60b1c2366a3805) in the middle of 2021. So if you try to use older AutoFDO profiles or just use old-enough AutoFDO tooling - you'll get profile compatibility errors since GCC [has](https://github.com/gcc-mirror/gcc/blob/master/gcc/auto-profile.cc#L941) an internal check for that. I know at least one person who trapped into this. The obvious recommendation - use recent GCC and AutoFDO tools.

Another option is using [llvm-profgen](https://llvm.org/docs/CommandGuide/llvm-profgen.html) tool. This tool is already a part of the LLVM monorepo so will no be problems with different compilation errors due to unsupported LLVM version. I don't have much experience with this tool but already found at least one issue - the tool [does not support](https://github.com/llvm/llvm-project/issues/82901) Linux perf samples without Branch Stack Sampling. This limitation can be significant for people who don't have BSS support in their environments.

If we are talking about Go, [pprof](https://github.com/google/pprof) tooling is used. And as a usual Google tool, it [has](https://github.com/google/pprof/issues) some bugs too. I don't have enough experience to share with you here since I didn't use PGO in Go a lot (due to its inmatureness) but I'll try fix it in the future.

What if you use another external profiler with custom profile format - you'll need to implement a new profile converter. Find a specification for a profile format of your profiler, read it, read GCC/LLVM profile spec, implement a proper script - can be a time-consuming task to do. E.g. check such a [request](https://github.com/janestreet/magic-trace/discussions/285) for the Jane Street's `magic-trace` profiler.

Almost the same situation is with custom compilers with custom PGO profiles format - you need to implement your own profile converter. So here is the obvious recommendation - try to use more widely-used tooling. In this case, with higher chances something already will be available on the market and won't be alone warrior in the profile converting field.

#### Other PGO types

There are additional PGO ways that are mostly presented only in LLVM infrastructure and the Clang compiler since in this area the most advanced PGO developments are done nowadays (IMO).

* Context-sensitive Sample PGO with Pseudo-Instrumentation (CSS PGO). This PGO type tries to resolve one of the biggest challenges with Samling PGO - improving PGO profiles quality. Here it's done via some kind of pseudo-instrumentation that brings little overhead compared to a "naive" sampling PGO but measurably improves PGO profiles quality and consequently the quality of PGO-driven optimizations (at least in the reported benchmarks). More details: [Google groups](https://groups.google.com/g/llvm-dev/c/1p1rdYbL93s/), [RFC](https://aaupov.github.io/assets/csspgo-rfc-email.html), CSS PGO profile generation [instructions](https://groups.google.com/g/llvm-dev/c/U6zI4M1l1SI), some performance [results](https://groups.google.com/g/llvm-dev/c/-Ao1uXCi8QM). I didn't test it in practice though.
* Control Flow Sensitive AutoFDO (FS-AFDO). Another attempt to improve Sampling PGO quality. In this work, PGO developers try to implement some mitigations against compiler optimizations that can influence Sampling PGO profile precision. More details - [clack](https://lists.llvm.org/pipermail/llvm-dev/2020-November/146694.html).
* Temporal PGO. This kind of PGO tries to optimize another thing - startup time for applications by reshuffling functions inside generated binaries by execution times to reduce the amount of page faults. Startup time is especially important for the mobile domain - that's why this work is done by Meta. More materials about Temporal PGO: [RFC](https://discourse.llvm.org/t/rfc-temporal-profiling-extension-for-irpgo/68068), [Google Groups](https://groups.google.com/a/chromium.org/g/chromium-dev/c/Awq_xBXQgFY/), Chrome on Android [experiments](https://issues.chromium.org/issues/326463850) with Temporal PGO.
* There is a [PR](https://github.com/llvm/llvm-project/pull/69535) (previous version is [here](https://reviews.llvm.org/D63949)) with implementation sampling approach for instrumentation PGO phase. The idea is simple - instead of incrementing each instrumentation counter each time we increment them with some sampling rate. Accoriding to tests, it helps to reduce instrumentation overhead in some cases up to 5x still with pretty good PGO profile overlap compared to default instrumentation - something about 90%. Current status: WIP.
* There is an ongoing [work](https://issues.chromium.org/issues/40242999) about improving Sampling PGO profile error correction efficiency with implementing a different approach aka "the Profi algorithm". Probably you also will be interested in it. I don't know what is the current status about this in the LLVM upstream.

Hard to say, how many things from the topics above are already upstreamed to LLVM, how many things are just temporal (heh) experiments - I didn't check it. At least we see many interesting developments with PGO - it's a good sign!

---

We can conclude that for now there are two the most stable PGO approaches: Instrumentation and Sampling. Instrumentation has far longer history than Sampling so we can *expect* higher availability across industry. Sampling PGO is a younger PGO approach but still pretty tested in the wild (at least by Google). Other PGO types are pretty experimental things and should be considered to use with some higher attention level.

We did a quick overview of different PGO types. What about their practical nuances?

### Common PGO problems

#### Applying PGO for multilingua software

Building software that consists of multiple programming languages was always a problem. Passing values from one programming language to another, writing or generating endless amount of bindings, setuping all required toolchains for eash language, dealing with bugs in each language/toolchain, developing a proper build pipeline in your favourite build system, using "dirty hacks" like Rust's [cc](https://docs.rs/cc/latest/cc/) (inevitable evil, bruh)  - and I can continue this list of horrible things. That's why we all like write software in one language, and if there is a need to use multiple - we try to separate them in a more strict way (like (micro)service approach).

Unfortunately, it's not possible in all cases to avoid such problems, and we still need to do all the things above in many situtations like almost endless amount of libraries like "C kernel + whatever language binding" (e.g. [rust-libxml](https://github.com/KWARC/rust-libxml) and many others). When we try to apply PGO for such projects, we meet additional amount of problems.

Right no there is no a build system that can *automatically* enable building with PGO all dependencies in different languages - we will discuss reasons for that a bit later. Instead, you will need to pass all required compiler flags for each programming language and this experience can be a bit tricky sometimes. At first, you need to figure out that an application consists of multiple languages. Because when you perform a build, usually there is no an indication like "Hey! In your Rust application there is a bunch of C and C++ dependencies too. Don't forget to pass PGO flags to them too. Cheers mate!" - you need to guess on your own. Good indicators to check: requirements to install a C/C++/whatever_lang compiler additionally to the main language of your software. Also, during the build I check with my eyes the executed process in `btop`/`htop` and if I see usually unexpected things like `cc`/`gcc`/`clang` - I start to dig deeper into the build scripts. If you think that it's obvious - nope, it isn't, especially if we are talking about huge software with dozens of dependencies like ClickHouse. When I tried to optimize [ast-grep](https://github.com/ast-grep/ast-grep/discussions/738#discussion-5899721) with PGO, at the beginning I was wondered why PGO didn't help for performance in such so suitable for PGO case. Only a bit later I found that there is [tree-sitter](https://github.com/tree-sitter/tree-sitter) C library as a dependency in `ast-grep`, and this library is responsible for parsing routines so at first I need to apply PGO on it to improve the `ast-grep` performance. Another similar example - Rust [bindings](https://github.com/ada-url/rust/issues/61#issue-2190406874) for `ada-url`. But here the situation was obvious enough since it's the binding project - I knew from the beginning that I need to resolve cross-language PGO question. So, usually, you need to play somehow with `CC`/`CXX` flags (via environment variables or directly in the build scripts) to resolve the issue.

Another issue with multilingua PGO - PGO profile format compatbility. E.g. if an application consists of two different parts in C++ and Rust, and for building each part you use Clang and Rustc compiler that under-the-hood have different LLVM versions (quite a common thing in practice since syncing such implementation details is almost impossible goal to achieve due to multiple reasons like different compiler release cycles, etc.) - you need to use two different `llvm-profdata` versions. Otherwise, you will get incompatibilty [errors](https://issues.fuchsia.dev/issues/42075308). Keep it in mind when you configure your PGO pipelines and always use tools for working with PGO profiles that are usually delivered with your compiler - in this case, you will avoid all the problems. Additionally, there is a similar [problem](https://github.com/astral-sh/ruff/issues/11280#issuecomment-2099121312) with applying LTO (that probably you will try to enable before PGO) for multilingua/multicompiler build pipelines due to incompatibilities between LTO format representation between compilers. Please keep it in mind as well.

#### Third-party and system libraries

One interesting issue is when your application is linked with provided by third-party vendors libraries. Here I mean system libraries from a distribution repository, prebuilt proprietary binaries from a vendor, etc. When you use such libraries, in general there is no way to apply PGO optimization on it since they are already prebuilt by someone else. With huge chances third party builders don't use any PGO-related optimization since at least they don't know your target workload (and because they are busy with other stuff).

Is it critical in practice? If you have hot loops for your applications inside some non-PGO optimized prebuilt library and performance of this library can be improved a lot with PGO - it's critical. Otherwise - probably it isn't. As usual - *it depends*.

What to do here? Well, not so many options:

* You can rebuild these libraries with PGO trained for your workload and link your application with them. In this case, additional maintenance cost are coming. The situation will be a bit simpler if you store 3rd-party libraries as sources together with your application - in this case rebuilding will be much simpler to do.
* You can try to speak with vendors for building these libraries with PGO - you establish with maintainers some training PGO workload, and then they use it during the build process. Honestly, I don't think that in reality anyone will agree to build a library specifically for your use case but who knows - maybe you will be lucky enough!
* Do nothing and accept your not-so-blazing-fast destiny.

However, some ecosystems like Rust partially mitigates this issue by, khm-khm, *motivating* you to building all your dependencies from sources and use static linking. In this way, the problem is *mitigated*. In other ecosystems like C or C++ such behavior is less popular so with C or C++ applications you will struggle with such a dilemma more often.

## PGO support in compilers

Different compilers have different PGO maturity levels. Some of them have supported PGO for decades, some of them added PGO just recently, and some (shame on you) have no PGO support at all! Below I prepared some overview of the PGO state across different compilers for multiple programming languages.

### C and C++

C has a very long history, C++ is a bit younger but still is quite a mature technology. So compilers for C and C++ also evolved **a lot** during the decades from many viewpoints, including multiple optimizations. Regarding PGO for C++ I found the "Infrastructure for Profile Driven Optimizations in GCC Compiler" [document](https://www.ucw.cz/~hubicka/papers/proj/index.html) that have [traces](https://www.ucw.cz/~hubicka/papers/proj/node20.html) up to 199x years - pretty old technology! LLVM infrastructure is much younger but still has PGO support I guess for 15 years or something like that. With such a long history it should be expected to have a lot of developments in this area - and this is what we see nowadays. Let's check PGO support across C++ compilers!

#### Clang

Clang documentation about PGO - [click](https://clang.llvm.org/docs/UsersManual.html#profile-guided-optimization).

With Clang I found the following issues:

* [Missing](https://github.com/llvm/llvm-project/issues/63024) partial training support for instrumentation PGO. Right now, Clang [supports](https://releases.llvm.org/16.0.0/tools/clang/docs/ClangCommandLineReference.html#cmdoption-clang-fprofile-sample-accurate) partial training only for the sampling PGO with `-fno-profile-sample-accurate` flag. However, for instrumentation PGO there is a [hack](https://github.com/llvm/llvm-project/issues/63024#issuecomment-1751625579) with special PGO profile preprocessing with `llvm-profdata` with `--sparse=true` flag - it emulates partial training functionality from GCC.
* By default, Clang [uses](https://clang.llvm.org/docs/UsersManual.html#cmdoption-fprofile-update) non-atomic profile counters. It could be a problem since non-atomic counters can bring some non-determinism into your PGO pipelines. However, by default it's done for better instrumentation PGO performance. More details about it can be found in the corresponding LLVM code [change](https://reviews.llvm.org/D34085) and the mailing list [thread](https://lists.llvm.org/pipermail/llvm-dev/2014-April/072172.html)
* As usual, there are multiple documentation issues (like [one](https://github.com/llvm/llvm-project/issues/74022), [two](https://github.com/llvm/llvm-project/issues/82140)) in different areas. E.g. if you check my one of the earliest PGO benchmarks, you will see `-fprofile-instr-generate` usage (FE PGO) instead of `-fprofile-generate` (IR PGO). As we discussed earlier, IR PGO nowadays should be preffered over FE PGO. But I didn't know that since this information wasn't written in the documentation! If I, a person who spent quite a lot of time with PGO, did this mistake - what about users who only start their PGO trip? Fortunately, right now the issue is resolved, and the Clang's PGO documentation [recommends](https://clang.llvm.org/docs/UsersManual.html#profiling-with-instrumentation) using IR PGO over FE PGO. Despite all of that, I would say Clang has **the best** PGO-related documentation at least among the open-source compilers.
* Sometimes you can [meet](https://issues.fuchsia.dev/issues/42063306) "Unable to track new values: Running out of static counters..." warning. This is due some LLVM internal implementations details about how much memory is allocated during the instrumentation phase for some counters. In most situtations, you can ignore/suppress such warnings. If you care you can try to increase the amout of counters via `-mllvm -vp-counters-per-site` compiler switch. However, I didn't find benchmarks about how it influences on the resulting PGO optimization quality - you can be the first one! More information can be found in LLVM [sources](https://llvm.org/doxygen/InstrProfiling_8cpp_source.html).
* Some smaller undocumented limitations like [impossibility](https://issues.fuchsia.dev/issues/42072804) to write multiple PGO profiles in continous mode or an annoying "function control flow change detected (hash mismatch)" warning ([Fuchsia](https://issues.fuchsia.dev/issues/336358027), [Chromium](https://issues.chromium.org/issues/40272404)) that can be "fixed" with the `-Wno-backend-plugin` warn supressor.
* Quite frequently you can meet an error about empty PGO profiles ([one](https://issues.chromium.org/issues/40277089), [two](https://issues.chromium.org/issues/40219172), [three](https://github.com/zamazan4ik/awesome-pgo/issues/3#issuecomment-2065582161)). Actually, it's not the Clang fault. This error appears then your instrumented application for some reason cannot dump PGO profiles. In my experience, the most common source of such problems are custom logic around the application exit: force killing, some tricky relationships between parent-child processes, etc. When you meet this error, I can recommend just modify your code a bit (if it's possible to do, of course) and manually save PGO profiles via the compiler instrinsics. Later we will cover this question in details.

By the way, if you are interested in some PGO implementation details in Clang, I can recommend to watch [this](https://www.youtube.com/watch?v=CvbIPGSnes8) talk from C++ Russia 2021. The only limitation - the talk is in Russian so prepare your translators.

Apple has a special Clang version - Apple Clang. Unfortunately, Apple Clang [documentation](https://opensource.apple.com/source/clang/clang-23/clang/tools/clang/docs/UsersManual.html) is not as rich as for the "main" Clang. I asked multiple questions about: [supported instrumentation PGO kinds](https://apple.stackexchange.com/questions/467215/profile-guided-optimization-pgo-kinds-in-xcode), [sampling PGO support](https://apple.stackexchange.com/questions/467182/sampling-profile-guided-optimization-pgo-with-xcode-and-clang), [differences between Clang and Apple Clang from PGO perspective](https://apple.stackexchange.com/questions/467218/profile-guided-optimization-pgo-differences-between-clang-and-apple-clang) (additionally [asked](https://developer.apple.com/forums/thread/742840) all these questions on the official Apple developer forum) - no answers. For now I guess a good idea with Apple Clang will be using the command-line reference to track supported compiler options and rely on the "main" Clang documentation in deeper questions. Of course, here there is a risk that Apple ported some features partially or didn't implement them at all in their Clang version. The best idea, however, will be just considering switching to the upstream Clang instead ;) For now I cannot say you why PGO situation with Apple Clang is so questionable. Maybe PGO is not very popular in the Apple ecosystem. Maybe even Apple-oriented users prefer to use other compilers...

#### GCC

GCC doesn't have a dedicated guide about PGO like Clang - only the corresponding compiler switches description: for the [Instrumentation](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html#Instrumentation-Options) and [Optimization](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-fprofile-use) phases. Not the most convenient way to learn how to use PGO with GCC, I would say.

With GCC I found the following issues:

* [No](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=112717) profile compatibility guarantees at all. It means that for every GCC update in theory you need to regenerate your PGO profiles. In practice - who knows, I didn't test it. GL HF!
* PGO profiles' runtime dumping capabilities [are limited](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=112829). Compared to LLVM, there is no easy way to dump PGO profile to a memory instead of a filesystem. GCC developers in this case suggest mitigations like using RAM-disk, NFS and other fancy stuff. If you want to implement it - you need to tweak GCOV implementation on your own.
* It's [not clear](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=112806) how PGO in GCC interacts with user-defined `likely`/`unlikely` hints. It could be a problem when you apply PGO for the (partially) optimized codebase with such user hints. You can get some unexpected results.
* Possible [issues](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=89307) with PGO counters contention in a multithreaded environment. However, I didn't meet such problems in practice. Probably because I mostly use Clang, huh? Additionally, you can [meet](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=68771) broken PGO profiles with multithreading ;)
* GCC matches PGO profiles for object files via storing a full path in the filename. It can bring some problems with `ccache` since it can change paths due to it's caching logic. Simple solution - just disable `ccache` during the PGO optimization phase. I also [found](https://issuetracker.google.com/issues/223669638#comment6) other notes about issues with `ccache` and PGO but without further details.

More issues about PGO in GCC can be found in its Bugzilla with [this](https://gcc.gnu.org/bugzilla/buglist.cgi?quicksearch=PGO&list_id=403385) filter. If you want to learn more about PGO-related GCC internals, I can recommend [this](https://trofi.github.io/posts/243-gcc-profiler-internals.html) article to read. If you are interested in Sampling PGO implementation details in GCC you can start [here](https://github.com/gcc-mirror/gcc/blob/master/gcc/auto-profile.cc#L55). If you are interested in PGO history inside GCC, I highly recommend read articles from [Honza Hubička](https://hubicka.blogspot.com/) about GCC internals: [GCC 4.8](https://hubicka.blogspot.com/2014/04/linktime-optimization-in-gcc-2-firefox.html), [GCC 5](https://hubicka.blogspot.com/2015/04/GCC5-IPA-LTO-news.html), [GCC 6 and Clang 3.9](https://hubicka.blogspot.com/2016/03/building-libreoffice-with-gcc-6-and-lto.html), [GCC8 and Clang 6](https://hubicka.blogspot.com/2018/12/even-more-fun-with-building-and.html), [GCC9](https://hubicka.blogspot.com/2019/05/gcc-9-link-time-and-inter-procedural.html). Don't forget to follow the links inside these articles - they also have many interesting details!

By the way, compared to Clang, GCC [tries](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html#index-fprofile-update) to use an atomic approach for incrementing counters for multithreaded applications (with some limitations but anyway).

#### MSVC

About MSVC I cannot say much - I didn't use MSVC for years and have no enough experience. However, after reading the corresponding PGO [documentation](https://learn.microsoft.com/en-us/cpp/build/profile-guided-optimizations) I can highlight the following things:

* MSVC does not support sampling PGO. Probably there is a possibility to write a conversion tool from a profiler like Intel VTune and convert it to the MSVC-compatible PGO format. But I didn't see anything like this in the wild.
* I didn't find PGO profile compatibility guarantees in the documentation.
* You cannot enable PGO without LTO. It could be a problem since LTO often breaks C and C++ program (due to different hidden UB in them) and LTO bumps requirements for your build machine (it could be a problem for the resource-constrained build environments).
* There is no way to compare two PGO profiles (`llvm-profdata overlap` alternative).
* No alternative to `-fprofile-partial-training` [flag](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-fprofile-partial-training) from GCC.
* You cannot dump PGO profile into a memory. The only way to dump the profile is a save it to a filesystem.

I [asked](https://developercommunity.visualstudio.com/t/Multiple-questions-about-Profile-Guided-/10537677) all these questions on the Microsoft Developer community platform. I hope one day we will get answers there. Also can recommend check `vcruntime` [sources](https://github.com/ojdkbuild/tools_toolchain_vs2017bt_1416/blob/master/VC/Tools/MSVC/14.16.27023/crt/src/vcruntime/exe_common.inl#L204) - some interesting details and PGO nuances can be found there!

#### Intel compilers

At first, we [have](https://community.intel.com/t5/Intel-C-Compiler/What-s-the-difference-between-icx-icl-and-icc-compilers/m-p/1224714) two Intel compiler generations: Intel C++ Compiler Classic aka `icc`/`icl` and Intel oneAPI DPC++/C++ Compiler (a LLVM-based compiler with some proprietary extensions) aka `icx`. ICC is already deprecated however is still used.

`icc` compiler [supports](https://www.intel.com/content/www/us/en/docs/cpp-compiler/developer-guide-reference/2021-10/profile-an-application-with-instrumentation.html) only Instrumentation PGO. Since the compiler uses it's own PGO profile format, it also [has](https://www.intel.com/content/www/us/en/docs/cpp-compiler/developer-guide-reference/2021-10/pgo-tools.html) dedicated tooling to work with them. PGO-related compiler intrinsics are also [supported](https://www.intel.com/content/www/us/en/docs/cpp-compiler/developer-guide-reference/2021-10/pgo-api-support.html).

`icx` compiler has Instrumentation PGO support. Seems like instrumentation PGO in this compiler is fully based on the LLVM implementation - the [documentation](https://www.intel.com/content/www/us/en/docs/dpcpp-cpp-compiler/developer-guide-reference/2024-0/instrumented-profile-guided-optimization.html) even has references to the LLVM documentation about PGO. However, it's unclear about support for additional PGO features like CSIR PGO, which `llvm-profdata` version shall be used with each `icx` version (since otherwise can and will be PGO profile version mismatch errors), PGO profile dumping intrinsics availability - no information for that in the documentation.

`icx` also has Sampling PGO support - it's called "Hardware Profile-Guided Optimization" aka [HWPGO](https://www.intel.com/content/www/us/en/docs/dpcpp-cpp-compiler/developer-guide-reference/2024-0/hardware-profile-guided-optimization.html). According to the description it works pretty like a usual SPGO except an interesting note - it also works with Intel VTune profiles (with enabled [SEP](https://www.intel.com/content/www/us/en/content-details/686066/sampling-enabling-product-user-s-guide.html)). Didn't try it yet but sounds interesting!

Honestly, cannot say many things about PGO quality in Intel compilers in practice - didn't play with them a lot. However, since new Intel C++ compiler is based on LLVM, I can expect quite good PGO performance in practice. And, in my opinion, Intel compilers have the best PGO-related documentation on the market - great job!

#### AMD

According to the [documentation](https://www.amd.com/en/developer/aocc.html#documentation), AOCC compiler (a LLVM-based C++ compiler from AMD) has very limited PGO support. Documentation has only few words about PGO, and even these words are only about FE PGO. No IR PGO support, no sampling PGO support, no more advanced PGO versions or some PGO-related tricky options. I don't know - it is a real compiler limitation or *just* documentation issues. I do not understand why AMD engineers don't care about PGO in their compilers. So at least from the PGO perspective I recommend using Clang instead.

#### MCST and Elbrus

Do you want something more unique? Let's talk about [Elbrus](https://en.wikipedia.org/wiki/Elbrus_2000). Elbrus (aka `e2k`) is a [VLIW](https://en.wikipedia.org/wiki/Very_long_instruction_word)-based architecture. From PGO perspective we are interested only in one specific detail about all VLIW-based things - how much VLIW-based CPUs rely on a compiler. VLIW method depends on the programs providing all the decisions regarding which instructions to execute simultaneously. As a practical matter, this means that the compiler becomes more complex, but the hardware is simpler than in many other means of parallelism. In contrast, our usual x86-64/ARM CPUs do this things on the CPU side.

This decision has an outcome - at least in theory, VLIW can get far more improvements from PGO than other architectures because suboptimal compiler decisions in VLIW have higher performance penalty.

Elbrus engineers (from [MCST](https://en.wikipedia.org/wiki/MCST)) of course understand this thing, and they invest some (quite limited compared to giants like Google) resources into PGO too. MCST has its own compiler, [LCC](http://mcst.ru/LCC) (in Russian). Finding public-available information about the compiler is not an easy task due to closed nature of MCST. However, I had an awesome conversation with an engineer from MCST, and he answered almost all my questions regarding PGO state in the LCC compiler.

The current PGO state in LCC is something like that:

* Instrumentation PGO is supported (in IR PGO flavor).
* No sampling PGO support yet but there are some plans to implement it.
* LCC saves PGO profiles in its own format. There is a tool `eprof` to work with these profiles. This tool can merge profiles and show some statistics about PGO profiles. So in general it looks like a viable alternative to `llvm-profdata`.
* No way to tweak PGO profiles dumping via compiler intrinsics (Clang [has](https://clang.llvm.org/docs/UsersManual.html#fine-tuning-profile-collection) such functionality).

[ALT Linux](https://en.wikipedia.org/wiki/ALT_Linux) (a Russian Linux-based OS that support Elbrus CPUs)LCC [has](http://ftp.altlinux.org/pub/people/mike/elbrus/docs/elbrus_prog/html/chapter1.html#twostage-comp) a dedicated page about PGO. From that page you can find that LCC has a dedicated documentation about PGO. Unfortunately, this documentation is not availabe online - the only way to read it is get access to an Elbrus machine (see the link below). Fun fact: this documentation is so "well-known" that even some LCC engineers don't know about it existance :) I checked LCC's PGO documentation and would say that the documentation is quite detailed. Of course, more practical scenarios could be covered in more details, etc. but in general it's fine.

What is the LCC's PGO implementation quality in practice? There is no publicly-available benchmarks so I performed some for you instead. Since many software for Elbrus need to be patched, and available Elbrus machines are kinda slow (so I need to wait for the benchmark results) I did only one benchmark - for SQLite. The results are available [here](https://github.com/zamazan4ik/awesome-pgo/blob/main/sqlite.md#results). However, nothing too interesting, to be honest - PGO works in LCC as in other compilers, with similar performance improvements. It would be awesome if MCST engineers can provide more benchmarks in the future. Hm... does anyone have PGO benchmarks for Itanium? I am interested in such benchmarks as well (at least from the historical perspective since no one cares about Itanium anymore).

By the way, if you want to play with e2k-based CPUs (because why not?), you can request **free** remote access [here](https://elbrus.kurisa.ch/). I checked - it really works, I got SSH access to a Elbrus-based machine in a 2 days after the request. Nice job!

#### Other C++ compilers

What about other proprietary compilers like [Cray C++ compiler](https://docs.lumi-supercomputer.eu/development/compiling/cce/) and others? I don't know and actually don't care much (because prefer open source compilers instead). If you are interested in these compilers - please check corresponding documentation. If PGO support is missing in them - ask vendors about implementing PGO in their compilers.

---

Conclusion: Clang has the most advanced PGO implementation nowadays, and its implementation still evolving since Google invests money into it. GCC also has a good PGO implementation but without some most advanced features like Temporal PGO. Regarding other compilers (like MSVC) that are not based on GCC or LLVM infrastructure - their PGO implementations stay behind Clang and GCC, and support only "basic" PGO scenarios.

### Rust

Rust has only one major compiler - `rustc`. Rustc is based on LLVM so PGO implementation shares almost all details with Clang.

Some details that I want to highlight about PGO in this compiler:

* An annoying [bug](https://github.com/rust-lang/rust/issues/115344) with enabled LTO and PGO at the same time. Since LTO is widely enabled in the Rust ecosystem, it can become a blocker for Rust projects for adopting PGO.
* Rustc [supports](https://github.com/rust-lang/rust/issues/64892) PGO via sampling. However, official documentation [does not cover](https://github.com/rust-lang/rust/issues/117023) this option.
* [Missing](https://github.com/rust-lang/rust/issues/118562) support for CSIR PGO. Not critical at all but according to Google engineers CSIR PGO can achieve additional few percents in performance compared to usual IR PGO.
* [Missing parts](https://github.com/rust-lang/rust/issues/118561) in the PGO documentation. There a lot of undocumented places in the current documentation: PGO profiles compatibility guarantees, PGO dumping-related compiler intrinsics documentation, PGO counter atomicoty behavior, etc. Rust documentation for further details refers to the Clang documentation (since Rustc from PGO perspective completely relies on LLVM). It's kinda funny because Clang also has [some](https://github.com/llvm/llvm-project/issues/74022) documentation issues in PGO area. From my point of view, it would be much easier for the Rustc users to read all PGO-related information directly from the Rustc documentation, without jumping from time to time into the Clang docs.
* Rarely you can meet [mysterious](https://github.com/rust-lang/rust/issues/123361) issues about SIGSEGV somewhere in the LLVM depths... Or a bug/feature about changing the way how command line arguments are passed from Cargo to Rustc - it [caused](https://github.com/rust-lang/cargo/issues/7416) PGO issues too. You find something similar - GL & HF! Sorry, wanted to say "collect all statistics, core dumps, backtraces, logs, input, environment configuration, MRE (Minimal Running Example) if it possible to prepare - and create the issue about that in the compiler bug tracker". PGO journey sometimes is suffering ;)

Rustc documentation about PGO is available [here](https://doc.rust-lang.org/rustc/profile-guided-optimization.html).

If you are interested in information about applying PGO to Rustc itself I can recommend [this](https://github.com/rust-lang/rust/issues/103595) meta-issue about applying different optimizations to Rustc (includes LTO, PGO, BOLT and other things).

What about other Rust compilers? TBH, a comparatively small amount of people in the Rust community cares about them. Anyway:

* mrustc: [No PGO support](https://github.com/thepowersgang/mrustc/issues/304) for the compiler itself and for compiling Rust programs with the compiler.
* [Ferrocene](https://ferrous-systems.com/ferrocene/): Since it's downstream for the Rustc compiler, it supports PGO. However, the compiler itself [is not built](https://github.com/ferrocene/ferrocene/issues/22) with PGO.
* [gcc-rs](https://rust-gcc.github.io/): [No](https://gcc-rust.zulipchat.com/#narrow/stream/266897-general/topic/Profile-Guided.20Optimization.20.28PGO.29.20support.20with.20.20gcc-rs/near/436664530) support. Honestly, too alpha-quality right now for having such a thing like PGO. One day if the project still will be alive, the developers will be able to add PGO support based on the GCC's PGO infrastructure.

### Go

PGO in Go appeared quite recently: 1.20 in Preview, 1.21 in GA state.

Current PGO implementation in the `go` compiler has the following issues/limitations/inconveniences:

* **Very** limited amount implemented PGO optimizations. It's completely okay for now since this PGO implementation is pretty young. However, for you as for an average PGO enjoyer it means that don't get all possible outcomes from enabling PGO. You can track already implemented optimizations in [this](https://github.com/golang/go/issues/62463) umbrella issue. However, Go dev team invests some money into their PGO implementation, so we can expect improving the sitation with missing PGO-related optimizations in the compiler over time - e.g. in Go 1.22 PGO-based devirtualization was [added](https://tip.golang.org/doc/go1.22#compiler).
* There is [no](https://github.com/golang/go/issues/64487) easy way to perform weighted merge for PGO profiles. It could be important in cases when you have PGO profiles with different importance for your needs - weighted merge solves this problem. There are some ways ([click](https://github.com/golang/go/issues/64487#issue-2019987476) and [clack](https://github.com/golang/go/issues/64487#issuecomment-1843692572)) to resolve the issue but with worse UX that it should be in the production-ready PGO tooling.
* There is [no](https://github.com/golang/go/issues/64394#issuecomment-1830318982) easy way to measure difference between two PGO profiles. One of the common cases for it is tracking how different your PGO profiles are for different training workload. Based on this knowledge you can decide to build two separate binaries for two different target workloads. The only way to achieve it for now is manual implementation or just waiting for the implementation in the upstream - eventually it could implemented (I hope).
* There is [no](https://github.com/golang/go/issues/62444) built-in option to dump the PGO profile after the program exits or trigger PGO dumping via a signal. This scenario is very convenient in cases when you optimize some CLI utilities or one-time jobs. Right now, the only way to get this functionality - implement it manually via patching sources. In other PGO implementations (like LLVM and GCC) such an option is available - and I can in a simple way optimize all my programs without touching the code.
* [Limited](https://github.com/golang/go/issues/64489) integration with existing profiling tooling like Linux' `perf`. It could be an issue if you already have some existing profiling infrastructure - in this case, you need to spend initial efforts for integrating `pprof`-based PGO implementation from Go (if your profiling infra is already `pprof`-based - you are lucky). Not critical at all since it can be easily [fixed](https://github.com/golang/go/issues/64489#issuecomment-1846003368). I think eventually it will be fixed in the upstream. By the way, all currently supported external profilers for Go's PGO are listed [here](https://github.com/golang/go/wiki/PGO-Tools).
* Small documentation issues like profile compatibility [guarantees](https://github.com/golang/go/issues/64394).

My 50 cents about PGO in Go. I am **very** glad to see current PGO movement in the Go ecosystem. We already have a lot software written in Go, many of these applications are running with pretty high loads. Almost all modern "cloud-native" stack is written in Go - it will be nice idea to improve all these applications with PGO. Yeah, for now you need to use pretty new compiler, current implementation has some flaws, etc. But the situation will quickly change so you can start experimenting with PGO in Go applications right now.

For those who want to research the PGO question in Go a bit deeper, I collected some links:

* Original PGO proposal: [link](https://go.googlesource.com/proposal/+/master/design/55022-pgo.md)
* Go official documentation about PGO: [link](https://go.dev/doc/pgo)
* PGO announcement article for Go 1.21: [link](https://go.dev/blog/pgo)
* Some interesting internal details: [link](https://theyahya.com/posts/go-pgo/)
* GopherCon 2023 "Automating Profile-Guided Optimization at Reddit" talk from [Marco Ferrer](https://github.com/marcoferrer) (the recording is not available yet AFAIK)

### Fortran

I didn't use Fortran at all during my engineering career. I have nothing against this technology but I prefer to spend my time with other languages. I understand that Fortran is important in some areas like HPC, so knowing about PGO support will be a good thing. I did some quick research across Fortran compilers:

* [Gfortran](https://gcc.gnu.org/wiki/GFortran) is based on the GCC compiler inftrastructure so as far as I understand PGO support should be similar to GCC.
* There are [several](https://github.com/flang-compiler/flang/wiki) Fortran compilers under the same name - Flang (what a mess). As far as I understand, ["classic" Flang](https://github.com/flang-compiler/flang) doesn't support PGO since I didn't find mentions for it in the repository. The new Flang (aka F18) which is already a part of the LLVM, definetely [doesn't support](https://github.com/llvm/llvm-project/issues/74216) PGO yet.
* As with C++ compilers, for Fortran Intel also [has](https://www.intel.com/content/www/us/en/docs/fortran-compiler/developer-guide-reference/2024-1/overview.html) two compilers: Intel® Fortran Compiler Classic aka `ifort` and Intel® Fortran Compiler aka `ifx` (a new LLVM-based compiler). `ifort` is already deprecated and support for it will be stopped in October 2024. Anyway, `ifort` [has](https://www.intel.com/content/www/us/en/docs/fortran-compiler/developer-guide-reference/2024-1/profile-guided-optimization.html) a similar PGO support as Intel C++ classic compilers: dedicated PGO implementation with instrumentation PGO support, dedicated tools to work with PGO profiles, no sampling PGO support, etc. `ifx` reuses LLVM's PGO infrastructure so I can expect the same PGO level support as Clang but without the most advanced features. However, some differences should be expected in practice since I am sure that `ifx` lags behind at least a bit. 
* IBM Open XL Fortran [supports](https://www.ibm.com/docs/en/openxl-fortran-aix/17.1.2?topic=compatibility-profile-guided-optimization-pgo) instrumentation PGO. Unfortunately, I don't sampling PGO mentions in the documentation.
* AMD's AOCC supports building Fortran application. At least according to the documentation, the compiler [supports](https://www.amd.com/en/developer/aocc.html#documentation) PGO somehow but I didn't check it in practice.

As a small conclusion: as a Fortran developer you already can try to optimize your Fortran applications with PGO.

### C#

Microsoft few years ago started investing in the PGO implementation in C#. Unfortunately, due to lack of my experience with C#, I cannot say much about the quality of this implementation, traps and nuances. Instead, I will share with you some interesting from my point of view articles and links about the PGO state in C# for further reading:

* [Dynamic PGO in .Net 6](https://gist.github.com/EgorBo/dc181796683da3d905a5295bfd3dd95b). By the way, the article author, [EgorBo](https://gist.github.com/EgorBo), is an expert in C# PGO implementation. So probably you can ask him for more details directly!
* [PGO improvements in .Net 7](https://petabridge.com/blog/dotnet7-pgo-performance-improvements/)
* [Conversation about PGO in Microsoft blog](https://devblogs.microsoft.com/dotnet/conversation-about-pgo/)
* Dynamic PGO official design [documentation](https://github.com/dotnet/runtime/blob/main/docs/design/features/DynamicPgo.md)

Anyway, I am happy to see serious investments from Microsoft in PGO for C#. We already have a lot of C# software - having an additional optimization technique would be valuable for the whole industry.

### Java

[GraalVM](https://www.graalvm.org/) implements AoT compilation mode for Java. Unfortunately, I have no experience with GraalVM so I cannot tell you, how GraalVM's PGO implementation works in practice, what caveats you should expect there, etc.
With GraalVM PGO I found the following issues:

* [Official documentation](https://www.graalvm.org/22.0/reference-manual/native-image/PGO/) has so few interesting details so I even hesitate to recommend it as a further reading. Maybe talking directly with the GraalVM engineers would be a good idea?
* GraalVM even [supports](https://github.com/oracle/graal/discussions/7648) sampling PGO but the documentation is a bit outdated. Hopefully, it will be fixed soon.
* [Lack](https://github.com/oracle/graal/discussions/7901) of some PGO-related tooling,
* [Questions](https://github.com/oracle/graal/discussions/7892) about the PGO profile compatibility.
* [Unknown](https://github.com/oracle/graal/discussions/7990) behavior in case of conflict between manually written `likely`/`unlikely` attributes and PGO profiles.
* [Limited](https://github.com/oracle/graal/discussions/7991) PGO profile dumping functionality. Only dumping to a filesystem is officially supported. No dumping to a memory, no PGO-related compiler intrinsics - at least in the documentation.
* It's [unclear](https://github.com/oracle/graal/discussions/7999) how GraalVM optimizes non-executed during the PGO training phase code. Is it considered as a "cold" code or just optimized as a regular Release code? E.g. GCC [has](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-fprofile-partial-training) `-fprofile-partial-training` option for that.
* Not exactly an issue with PGO quality but I wanna mention it anyway. According to the [issue](https://github.com/oracle/graal/discussions/7989), GraalVM has own LLVM distribution. I am almost sure that this distribution is not optimized with PGO. However, according to my [tests](https://github.com/llvm/llvm-project/issues/63486), performance of many LLVM parts can be improved with PGO. So probably PGO usage can be extended in this area too ;)

PGO in GraalVM is available only in GraalVM Enterprise license but don't worry - Enterprise license is free [now](https://blogs.oracle.com/java/post/graalvm-free-license) so you do not need to pay at least for a license if you want to use PGO in Java. Good? Great!

One of the biggest issues with GraalVM right now is its adoption across the Java ecosystem. Even if some large and well-known projects like Kafka try to [adopt](https://cwiki.apache.org/confluence/display/KAFKA/KIP-974%3A+Docker+Image+for+GraalVM+based+Native+Kafka+Broker) GraalVM builds, in general, GraalVM usage is low. AoT GraalVM nature has some difficulties with runtime Java features like reflection. There is a way to mitigate it - using the GraalVM metadata [repository](https://github.com/oracle/graalvm-reachability-metadata).

What about GraalVM PGO efficiency? Honestly, I don't know. I have no hands-on experience with it on real applications. All found public benchmarks for GraalVM PGO also too simple and synthetic to consider them seriously (IMHO). I opened a [request](https://github.com/oracle/graal/discussions/7988) in the upstream for sharing more benchmarks - maybe eventually the collection will be bigger. According to my not-so-deep research, in many cases, GraalVM Native Image + PGO still performs worse than usual JIT from the peak performance perspective.

By the way, GraalVM supports AoT compilation mode for Python, Ruby, R, Javascript (and maybe something). So you can try to apply GraalVM-flavoured PGO on them too (not many people tried to do such things so bugs should be expected).

### Kotlin

[Kotlin Native](https://kotlinlang.org/docs/native-overview.html) has [no](https://youtrack.jetbrains.com/issue/KT-63357/Native-Profile-Guided-Optimization-PGO-support#focus=Comments-27-8353088.0-0) PGO support and Kotlin dev team has no plans to implement it in the near future. So if you want to use PGO with Kotlin - you will need to patch the Kotlin compiler. Another option is using GraalVM since it supports PGO (see Java section for more details). Anyway, using GraalVM instead of Kotlin Native [is not a new thing](https://www.reddit.com/r/Kotlin/comments/172sul9/whats_position_of_kotlin_native_in_the_kotlin/) in the community.

### Scala

[Scala Native](https://scala-native.org/en/stable/) has [no](https://github.com/scala-native/scala-native/issues/1568) PGO support. However, once again, you [can](https://www.reddit.com/r/scala/comments/ju9ty2/best_practices_for_using_graalvm/) use GraalVM with Scala (see Java section for more details).

### Swift

It's a complicated question. From one perspective, in the Swift compiler sources, [there are](https://github.com/apple/swift/blob/main/include/swift/Option/Options.td#L1322) some PGO footprints. From another - even the compiler developer is [not sure](https://github.com/apple/swift/issues/69227) about the current PGO implementation state in the compiler. Anyway, there is an **unanswered** [topic](https://forums.swift.org/t/several-questions-regarding-profile-guided-optimization-pgo-and-llvm-bolt/67963) on the Swift forum. Hopefully, one day it will get an answer. And I am [not alone](https://forums.swift.org/t/does-the-swift-compiler-support-ir-level-llvm-instrumentation-pgo/52666) with questions about PGO and Swift.

What to do? Try to ping Swift developers once again to clarify (and document) the situation or perform your investigation. If you already have some insights about PGO in Swift - please let me know!

### Zig

[Zig](https://ziglang.org/) has [no](https://github.com/ziglang/zig/issues/237) support for PGO. As far as I see, Zig developers don't care much about PGO support for their lang. There was a chance that Zig compiler eventually will reuse PGO infrastructure from LLVM (which is already used by many LLVM-based compilers like Clang and Rustc). However, Zig developers decided to [eliminate](https://github.com/ziglang/zig/issues/16270) LLVM dependency for the Zig compiler. This decision aside from other consequences kills the idea about reusing existing PGO ecosystem. So what you can do? Maybe you can vote for implementing PGO in the Zig compiler. Without first-class PGO support you will need to execute something like "Zig code compile to C code -> C code compile with PGO" compilation pipeline. which can be a bit more difficult to implement in practice.

### D

TODO

D language [has](https://wiki.dlang.org/Compilers) multiple compilers. The major compilers are DMD (default D compiler), GDC (GCC-based compiler), LDC (LLVM-based compiler).

LDC [supports](https://wiki.dlang.org/LDC_LLVM_profiling_instrumentation) PGO. More about PGO in LDC could be found in [this](https://johanengelen.github.io/ldc/2016/07/15/Profile-Guided-Optimization-with-LDC.html) and [this](https://johanengelen.github.io/ldc/2016/04/13/PGO-in-LDC-virtual-calls.html) articles. I also can recommend to watch the "Profile-Guided Optimization in the LDC D compiler" [talk](https://av.tib.eu/media/42272) from FOSDEM 2017.

### Dart

[Dart](https://dart.dev/) does [not](https://github.com/dart-lang/sdk/issues/53855) support PGO in AoT compilation mode (via `dart2native`). I have no idea when it will be implemented. If you are interested in this feature - please ping Dart dev team and somehow vote for the feature. By the way, DartVM itself (which is written in C++) also is [not](https://github.com/dart-lang/sdk/issues/53856) PGO-optimized in the upstream builds. Quite funny to see not implemented PGO in the Google products since Google is the largest PGO user in the world AFAIK. As we see, different parts of Google thinks a bit differently about performance optimization ;)

### Nim

There [was](https://github.com/nim-lang/RFCs/issues/130) a discussion about implementing PGO to Nim. Unfortunately, the RFC was closed due to some implementation concerns from developers perspective. I don't have enough experience with Nim or Nim compiler so cannot discuss it deeper here. If you are interested in increasing performace in the Nim ecosystem - you can try to raise the PGO enablement question once again in the Nim community.

### Pascal

I am not a huge Pascal expert (last time I used it somewhen 10-15 years ago) but according to the [discussion](https://forum.lazarus.freepascal.org/index.php/topic,65162.0.html) on the Lazarus forum, modern Pascal compilers don't support PGO. AFAIK there are no huge investments nowadays into the Pascal ecosystem so chances to get PGO support for Pascal are critically low.

IMHO, the only viable option here is to reuse somehow LLVM's PGO infrastructure and "just" pass all required flags from the Pascal frontend. How difficult is it to implement in practice? I don't know.

### Haskell

Haskell has [many](https://wiki.haskell.org/Implementations) compilers in its ecosystem. Unfortunately, the main compiler - GHC - [doesn't support](https://gitlab.haskell.org/ghc/ghc/-/issues/18393) PGO and no ongoing activities are found in this area. Other Haskell compilers also don't support PGO.

### Julia

According to the [documentation](https://julialang.github.io/PackageCompiler.jl/stable/devdocs/sysimages_part_1.html), Julia by default has a JIT compilation model (this page calls it as "Just-Ahead-of-Time (JAOT)" but I will omit such nuances). Since community [asks](https://discourse.julialang.org/t/ahead-of-time-aot-compiler/14212) for AoT compilation model, [PackageCompiler](https://github.com/JuliaLang/PackageCompiler.jl) was developed. However, as far as I understand, this compiler doesn't support compilation with PGO. I [asked](https://github.com/JuliaLang/PackageCompiler.jl/issues/940) the question in the upstream about plans for that but I guess they don't have enough resources/motivation to implement it. However, the Julia compiler itself [supports](https://github.com/JuliaLang/julia/pull/45641#issue-1268010204) building with PGO.

### Ocaml

Out of the box, the `ocamlopt` compiler has no support for PGO. However, there is a dedicated a bit out-dated tool - [ocamlfdo](https://github.com/gretay-js/ocamlfdo). Jane Street (probably the largest enterprise Ocaml [user](https://www.janestreet.com/technology/), you even can check their [tech talks](https://www.janestreet.com/tech-talks/jane-and-compiler/) about the Ocaml compiler) internally uses "a bit" modified Ocaml compiler - [ocaml-flambda](https://github.com/ocaml-flambda/flambda-backend). If you want to try PGO with Ocaml - please ask Greta Yorsh aka [gretay-js](https://github.com/gretay-js). More details you can find in [this](https://github.com/ocaml/ocaml/issues/12200) discussion on GitHub. Hopefully, one day all this great work will be merged to the upstream and all Ocaml users will be able to use PGO for their applications.

### Cobol

Yeah, I am not joking - Cobol still matters. This technlogy is even (kinda?) alive: [Cobol 2023](https://en.wikipedia.org/wiki/COBOL#COBOL_2023) edition was released! Unfortunately, regarding PGO the situation is sad - I didn't any Cobol compiler with PGO support. There is a [feature request](https://sourceforge.net/p/gnucobol/feature-requests/456/) in GnuCobol,  

There are [many](https://www.infogoal.com/cbd/cbdcmp.htm) other Cobol compilers but I didn't check PGO support in them. I believe that PGO is not supported in them.

### Common Lisp (CL)

Regarding Common Lisp compilers it's simple - there are no compilers with PGO support. As far as I can udnerstand, there are no huge investments into the PGO implementation for them, so... If you want to use PGO with Common Lisp - you need to implement it manually in your favorite CL compiler. Below I collected PGO-related issues in them:

* SBCL: https://bugs.launchpad.net/sbcl/+bug/2045484 (rejected the idea)
* CCL: https://github.com/Clozure/ccl/discussions/467
* Clasp: https://github.com/clasp-developers/clasp/issues/1526
* ECL: https://gitlab.com/embeddable-common-lisp/ecl/-/issues/725
* ABCL: https://github.com/armedbear/abcl/issues/646

### Other programming langs

Excuse me if I didn't mention before Your Favorite Language - I tried to gather information about languages about which I at least heard something and where at least there is a chance to find the PGO support. I also reviewed a lot of less known programming languages and their compilers (like [Circle](https://www.circle-lang.org/), [Odin](https://github.com/odin-lang/Odin/discussions/3081), and many-many others) and can conclude that (almost) non of them support PGO in its compiler. Yes, I understand the reasons - some languages don't care much about the target performance, some of them are too young to implement PGO (there are more important features to implement), some projects simply do not have enough human resources to implement PGO in their compilers (and LLVM is not a universal answer in many situations. Even with LLVM the Rustc compiler [had](https://rust-lang.github.io/compiler-team/working-groups/pgo/) a dedicated working group for implementing PGO). But hey - if you develop a **performance-oriented** language with an AoT compilation model - please consider adding PGO into your compiler. Without it, you lose too many optimization opportunities.

If you are motivated-enough to research the question about non-mentioned above compilers you can try to use [this](https://en.wikipedia.org/wiki/List_of_compilers) list of compilers and contact corresponding developers!

---

As a small conclusion about PGO support in different programming languages. C and C++ nowadays have the most advanced PGO implementations (especially if we are talking about Clang). Other languages that have LLVM and GCC-based compilers have almost the same PGO capabilities as C++ but without some the most recent developments in this area - but they can be "backported" in the future. The Go compiler quickly improves its PGO infrastructure and in the future I expect it will be at the same level with other "mature" PGO implementations. C# has very promising PGO movements. For Java world GraalVM brings PGO but for now there are a lot of questions to answer. Regarding other languages... for now I am quite pessimistic.

## PGO profile processing

TODO: Write more about `llvm-profdata` tool usage. Check the documentation for the tool and find missing parts in it

Let's talk a bit about operations with PGO profiles. In practice, you want to perform at least the following things with profiles:

* Merge multiple PGO profiles into one. Useful when you gather profiles from multiple workloads and want to prepare one PGO build to rule all of them.
* Compare different PGO profiles with each other. Useful for understanding how different are execution paths of your application in different workloads or how many differences in PGO profiles are for different application versions.
* Get some statistics about a PGO profile. How many functions/branches are executed, find most commonly used functions, etc.

Usually, all these operations are done not with compilers but with dedicated tools. Each PGO ecosystem brings different tools for that. Let's do a quick overview for them.

In the LLVM ecosystem, the main tool to work with PGO profiles is [llvm-profdata](https://llvm.org/docs/CommandGuide/llvm-profdata.html). It supports all mentioned above scenarios like merging, comparing and showing summary and much-much more - just check the amount of different switches in it! It even supports SOTA (State-of-the-Art) scenarios like [merging](https://issuetracker.google.com/issues/259718081#comment18) Sampling and Instrumentation PGO profiles together with a goal to get the best from the both worlds: precision of instrumentation profiles (that can be collected rarely) and lightness of sampling profiles (that can be collected as frequent as we want due to low runtime overhead during the collection process). Implementation details for that can be found [here](https://reviews.llvm.org/D81981). Also, in the documentation can be found [unsupported](https://llvm.org/docs/CommandGuide/llvm-profdata.html#cmdoption-llvm-profdata-merge-gcc) options - the reason for its existence is still [unknown](https://discourse.llvm.org/t/llvm-profdata-merge-and-gcc-emit-format-support/79207).

The GCC's alternative to `llvm-profdata` is `gcov-tool` ([docs](https://gcc.gnu.org/onlinedocs/gcc/Gcov-tool.html)). This tool also allows merging multiple PGO profiles, comparing them and showing some statistics - basic scenarios are covered well. However, it supports far less "advanced" options compared to `llvm-profdata` so this tool provides you less flexibility. Since some scenarios like merging multiple PGO profiles at once [are not supported](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=47618#c29), community already developed extensions like [gcov-tool-many](https://github.com/yugr/maintainer-scripts/blob/master/gcov-tool-many). However, such extensions can be less maintained than the "official" GCOV tools so be keep it in mind.

TODO: finish about pprof

The official Go compiler [supports](https://go.dev/wiki/PGO-Tools) multiple tool. The main tool for work with the profiles is [pprof](https://github.com/google/pprof).

If you know other dedicated PGO-related tools - please let me know. I am especially interested if they support some non-usual scenarios for processing PGO profiles.

### Proprietary tooling

If you use a proprietary compiler with custom PGO implementation without reusing LLVM/GCC infrastructure, highly-likely you will need to use a custom PGO tooling. Some examples of such tools:

* `profmerge`, `proforder` [tools](https://www.intel.com/content/www/us/en/docs/cpp-compiler/developer-guide-reference/2021-10/pgo-tools.html) for Intel Classic Compiler.
* `pgomgr`, `pgosweep` [tools](https://learn.microsoft.com/en-us/cpp/build/pgomgr?view=msvc-170) for MSVC.
* `cleanpdf`, `showpdf`, and `mergepdf` [commands](https://www.ibm.com/docs/en/openxl-c-and-cpp-aix/17.1.2?topic=compatibility-profile-guided-optimization-pgo) for the older IBM XL C/C++ compiler (replaced by `ibm-llvm-profdata` tool in the newer IBM Open XL C/C++ compiler that is probably a fork from LLVM's `llvm-profdata`).
* `eprof` tool for the Elbrus's `LCC` compiler.

Cannot say much about these tools since I didn't try them in practice. In practice, one of the biggest limitation with them is their proprietarity. Since current PGO is not widely used nowadays, there is a (small) chance that your PGO scenario will not be covered by existing tools. With open-source, you are able to patch the tools according your needs,least check implementation details (since documentation in this area is not great) or even extract some helpful information about the PGO file format (could be the case if you implement your own PGO-related tool). You cannot do all these things with propritary stuff - you are tied to the provided by the compiler developers scenarios. My recommendation - prefer to use open ecosystems.

## PGO support in build systems

In 99.9(9)% cases, we do not compile our applications via direct compiler invocations - we use build systems. What kind of PGO support could we expect from a build system? I have the following wishes:

* I want to have the possibility to enable PGO for my application via *a build system flag* rather than *a compiler flag*. Why? Because I can use multiple compilers, each compiler can have different flags for enabling PGO (e.g. check the difference in PGO in MSVC and Clang). Writing such logic for each compiler is not what I want to do for each project where I want to enable PGO.
* Optimizing with PGO whole dependency tree, not only my project. Almost any modern application uses dependencies. And for making things even more complicated, these dependencies can be written in different languages, that are compiled by different compilers (with different PGO flags, as we know). In this case, enabling PGO build for the whole dependency tree is quickly becomes a non-trivial task.
* It should work.

What do we have now in the ecosystem?

### Cargo

Cargo (the default build system for Rust) has no built-in support for PGO. However, the community (particularly [Jakub "Kobzol" Beranek](https://github.com/kobzol)) developed an extension - [cargo-pgo](https://github.com/Kobzol/cargo-pgo). I highly recommend you using this Cargo extension if you are going to start optimizing Rust projects with PGO. I used for **every** Rust project that I PGOed (and I did it for a lot of them) - it always worked like a charm. It even supports optimizing with LLVM BOLT (we will talk about it later) as an additional post-PGO optimization step! I wish every other ecosystem eventually will get something similar.

Rust build ecosystem has a standard way to run benchmarks - `cargo bench`. One of the most common algorithm to perform PGO benchmark for a Rust project is:

* Find a Rust project with built-in benchmarks.
* Run `cargo bench` to get the benchmark result in the Release mode.
* Run `cargo pgo bench` to collect PGO profiles from the benchmarks with Instrumentation PGO.
* Run `cargo pgo optmize bench` to perform PGO optimization and compare PGO-optimized results with the Release results.

That's it - it's really simple to do. `cargo-pgo` supports collecting PGO profiles not only from benchmarks; it even supports optimizing your software with LLVM BOLT (covered later in this article).

Of course, even with such a handy tool, there are some nuances:

* [No](https://github.com/Kobzol/cargo-pgo/issues/33) sampling PGO support. Not a big deal if you are going to use instrumentation PGO. However, if you want to collect PGO profiles directly from production (as Google does) - sampling PGO is the only viable option. In this case, you need to patch `cargo-pgo` or just pass all required Rustc flags around manually without `cargo-pgo`. Sampling not supported also for PLO tools like LLVM BOLT - we will about them a bit later.
* If a Rust application has some non-Rust dependencies (like C or C++ "native" dependency - quite a common thing yet in the Rust ecosystem because RIIR movement is not powerful enough, lol), `cargo-pgo` [does not](https://github.com/Kobzol/cargo-pgo/issues/38) optimize these C or C++ dependencies with PGO. This is due to the difficult nature of building an application with different programming languages - in this case, `cargo-pgo` needs to detect the C compiler, depending on its vendor/version choose proper PGO-realted compiler flags, pass them properly, and other similar boring and error-prone to implement things. I completely understand why the `cargo-pgo` author doesn't want to implement it. But if you have such a case (as I [had](https://github.com/Kobzol/cargo-pgo/issues/38#issue-1878263921) with TiKV) - you need to resolve it manually. Some hacking around manually passing proper C/C++ flags in general should be enough.

Actually, `cargo-pgo` is the reason why most of my PGO benchmarks are performed for Rust projects. When I look at a random C or C+ project, my thoughts are somehting like "Ehh, at first I need to figure out how to build it properly (with installing all required dependencies in a *right* way). Then I need to figure out how to run benchmarks... Nah, I am too lazy, let's try to find a Rust alternative in the same application domain and perform PGO benches on it instead". Hey, C++ [committee](https://isocpp.org/std/the-committee) SG15 ("Tooling" study group) - what about having something like this for C++ too? Because, well, you know - tooling matters **a lot**.

### Bazel

Bazel [has](https://bazel.build/reference/command-line-reference#flag--fdo_instrument) built-in PGO support with `--fdo_instrument`/`--fdo_optimize` options. With provided by Bazel command-line options, you will be able to optimize your program with PGO without needing to know how to properly invoke PGO stuff in your favorite compiler. Unfortunately, there is no built-in support for using sampling PGO or more advanced instrumentation PGO modes like CSIR PGO. If you want to use them - pass corresponding flags manually.

But even such a "native" PGO integration into the build system could have some problems. I [found](https://github.com/envoyproxy/envoy/issues/25500#issuecomment-1724584679) one of them during the PGO testing for Envoy. For some unknown reason, Bazel denied compiling with PGO one of Envoy's dependencies when I tried to use the PGO option. But if I just pass the required compiler flags recursively for all dependencies - it works fine! I don't know what is the root cause for such behavior in this case (honestly, I don't care much since I got my job done there)... Anyway, now you know about this possible caveat and how to avoid the problem :)

### Meson

Meson [supports](https://mesonbuild.com/Builtin-options.html) building with instrumentation PGO via `b_pgo` option, more precisely - IR PGO. Similarly to Bazel, there is no built-in support for using sampling PGO or more advanced instrumentation PGO modes like CSIR PGO. So if you want to do something more advanced than simple instrumentation PGO - be ready to pass all required compiler switches manually.

### CMake

CMake (one of the most [popular](https://www.jetbrains.com/lp/devecosystem-2022/cpp/#Which-project-models-or-build-systems-do-you-regularly-use) build systems for C++ nowadays) has no built-in PGO support - only a [request](https://gitlab.kitware.com/cmake/cmake/-/issues/19273) for it. So for CMake-based projects passing required PGO flags via `CMAKE_C_FLAGS` and/or `CMAKE_CXX_FLAGS` (or via env `CFLAGS` and/or `CXXFLAGS`) are the only options.

### SCons

Not so widely used nowadays build system (is it good or bad is out of the topic) but still - it [doesn't support](https://github.com/SCons/scons/discussions/4437) building an application with PGO via built-in routines. If you use this build system - be ready to tweak compiler options manually.

### Other build systems

With some probability, you use other build systems like [Build2](https://build2.org/), [WAF](https://waf.io/) or any other build system (or even you wrote a new one). For such systems general estimation is only one - with a huge probability you won't find PGO support in them. So once again - be ready to play around with PGO-related compiler flags manually correspondingly to your build system.

## Package managers

TODO: Check Conan and Vcpkg support for PGO and how it could be done.
TODO: discuss about prebuilt binaries in package managers and PGO - it can be a tricky task to do such things
TODO: discuss language-specific and OS dependency managers
TODO: source-based package managers vs prebuilt binaries

Modern software in most cases consists of not only its source code - it also uses dependencies. Some technologies use them less due to various reasons (say "Hi!" to the  C world), some - *maybe* even too much (Javascript's [is-odd](https://www.npmjs.com/package/is-odd) vibe intensifies). Here we interested in it because we want to optimize our dependencies PGO too since a dependency's performance can influence **a lot** actual application performance.

There are multiple way to manage dependencies. The most barbaric one (that is still popular in the C ecosystem nowadays, btw) just copy dependencies sources into your application's repository, and then compile dependencies as a part of the application. From the PGO perspective, the case is completely the same as enabling PGO for your application - just use corresponding flags for your program, and dependencies will be PGO-optimized too. However, adding/removing/updating dependencies *frequently* in this way can be too painful. For resolving the issue humankind invented package (or dependency) managers.

There are viewpoints on the package managers: language-specific or OS-specific, with support for multiple languages or just tied to a one, source-based or binary-based, integrated into a build system or a dedicated tool. I think classification can be continued here. For us is important only one thing - how to enable PGO in all of them? Let's discuss this question.

Source-based package managers don't provide prebuilt binaries. Instead, a package manager delivers sources for all specified by you dependencies, and then they are build during the compilation process. From the developer perspective, it often look like pretty the same as we save these sources directly to a repo but here there are notable differences - how these dependencies are compiled. Usually, for compilation dependency sources a different set of compiler flags is used. These flags can be specified by a maintainer in a package recipe. E.g. imagine the case that for some unknown yet reason a library doesn't work with `-fstrict-aliasing` (hehe) but your application does. That's why you may want to have an ability to specify dependency-specific compiler switches. From PGO perspective it brings additional problems since we need somehow pass correspodning PGO flags to the dependency (especially if we are talking about Instrumentation PGO). In some package managers pass corresponding flags is pretty easy (like Rust's Cargo and `cargo-pgo`), in some - a bit more manual work is required (like Conan).

Next station - binary package managers. They can deliver already precompiled dependencies for you. In that case, you don't need to spend your or CI's time on building dependencies once again - someone already did it for you. From the PGO point of view, it can be a problem - what if precompiled library uses not good enough for your use case PGO profile? Different package managers solves it in different ways. Some of them like Cargo just doesn't support prebuilt dependencies. You won't need to resolve issues with prebuild binaries if you don't have prebuilt binaries at all - clever enough! Some of them like Conan allow you in a one click switch between source/binary library versions: if you are ok with the prebuilt version - use it and save your CI time, if you are not - use source version and recompile it locally (probably with PGO).

How many dependencies do support PGO nowadays? I did a small research, and the results are not fascinating: almost all libraries don't have a *dedicated* support for building with PGO. It's true for both source- and binary-based things. A pleasant exclusions are [pydantic-core](https://github.com/pydantic/pydantic-core/pull/741) and [python-libipld](https://github.com/MarshalX/python-libipld/pull/30) libraries: they optimize prebuilt binaries (a Rust library + Python bindings packed into a Python wheel) with some predefined PGO training scenario. Unfortunately, these examples are just exclusions from the overall state across package managers.

In theory, it's possible to optimize prebuilt binary libraries on the package manager side, and then deliver preoptimized library.

Why do we have this situation? Here we have a "standard" set of problems:

* Many maintainers don't know/don't care about PGO or don't believe in the PGO efficiency.
* Even if they know/care/believe, enabling PGO will require tweaking package recipe, implementing somehow 2/3-stage build, etc. Maintainers don't want to spend time on it - they have a lot of other work to do.
* Even if they care and ready to spend some time on enabling it - where do they need to collect PGO profiles? What if a user use case is different? Is it ok 

### Language-specific package managers

### OS-specific package managers

### Several pieces of advice

#### Missing/outdated profiles

Sometimes you pass PGO profiles to the compiler, and get some additional amount of PGO-related warnings. It can be a good idea to look at such warnings and evaluate them. If you see that too many (how many - it depends on the case) are without profiles - *probably* your training workload is not good enough. If you get too many "profile data may be out of date" - *probably* it's a good sign to update/regenerate your PGO profiles. Or you just made a mistake with providing the right filepath for profiles and some build script tweaks are required for the build scripts - I trapped into this several times too. For Clang, warnings like [-Wprofile-instr-out-of-date](https://clang.llvm.org/docs/DiagnosticsReference.html#wprofile-instr-missing), [-Wprofile-instr-out-of-date](https://clang.llvm.org/docs/DiagnosticsReference.html#wprofile-instr-out-of-date), [-Wprofile-instr-unprofiled](https://clang.llvm.org/docs/DiagnosticsReference.html#wprofile-instr-unprofiled) can help you. For other compilers - please check the corresponding documentation. If your compiler lacks such warnings - it can be a good idea to add a feature request for them.

By the way, if you have enabled by default `-Werror` flag, it can be a good idea to make exclusion for PGO-related warnings since they create to much noise - for almost any build you will get "profile data may be incomplete" warnings since achieving 100% PGO profile coverage is impossible in practice.

#### PGO instrumentation is not enabled

After enabling instrumentation PGO for your application, before running the actual training phase, please perform preliminary checks. Firstly, check the binary size. If after enabling instrumentation the binary size was not increased - you did something wrong and instrumentation didn't apply during the build phase. Highly-likely you did something wrong in your build scripts and somewhere the corresponding instrumentation flags were not applied. Also, I highly recommend before running the long training phase run something short like `your_app_name --help` and check that PGO profiles are successfully created. I made such a dumb mistake so many times then I started a long training script and after that profiles were not saved due to some stupid mistakes from my side or some trickier details in the project. So don't repeat my mistakes and PGO saves in early stages ;) I guess some more clever checker can be implemented like checking for the corresponding symbols in the binary but I didn't see something like that nor developed my version of it. Contributions are welcomed!

However, be cautious with this way - CMake build scripts can prepare many surprises for you! Sometimes your flags are overriden, sometimes they are not properly passed to vendored dependencies, etc. Each time you will need to carefully debug it.

---

Why do we have so poor PGO support in build systems? Firstly, PGO is not popular nowadays so there were no many requests about adding first-class PGO support to build systems. Secondly, it [can be a difficult task](https://github.com/SCons/scons/discussions/4437#discussioncomment-7361016) to do since it requires considering a lot of boring questions:

* Corresponding PGO-related compiler switches. For each supported compiler you need to gather all PGO-related switches, and integrate them into your build system interfaces. And resolve a bunch of additional issues like "PGO doesn't work without LTO" in MSVC case, "What should I do with non-supported yet compilers? Users can (and will) complain about it", "Different compilers support different PGO approaches (e.g. Clang has many of them) - how should I implement it in a generic-enough way on the build system side?", and many other similar questions.
* Different PGO kinds: Instrumentation, Sampling, FE and IR PGO, and other PGO modifications. Here also comes more advanced use-case like using CSIR PGO after the usual IR PGO - Clang documentation claims that it's a good way to improve PGO efficiency in practice. Do you need to implement such a scenario on a build system side too?
* Different PGO phases: training, PGO profile preprocessing, optimization. PGO profile processing is the most tricky part here since it requires additional tools like `llvm-profdata` or Google AutoFDO tools so here we create an additional amount of external dependencies. Or do we need to built-in such tools into the build systems? In that case, we increase maintenance costs...
* Post-Link Optimization phase (LLVM BOLT and others) with its details: different phases, different ways to do, etc. It's another topic to discuss but PLO also has some interleaving with PGO so users may want to see support for that too.

All these questions quickly become far worse when we start talking about cross-language PGO since a number of compilers quickly increases. Probably I forgot other questions but I hope you got the point - it's not an easy task. Even if we will look at build systems like Meson or Bazel - they support only basic PGO modes. During my experiments, such support level wasn't enough to fulfill my PGO practical needs.

Generally speaking, with the current PGO state across build systems manual compiler options tweaking will be required for enabling PGO. It will be the most reliable way to enable PGO. If you have an application with some dependencies in multiple languages (like a common C + Rust situation) - be ready to tweak build scripts more carefully and probably pass PGO-related flags via environment variables. And don't forget to ensure that your flags are not flushed somewhere in the middle of the build scripts - in this case, it will be pretty annoying to debug!

## PGO profile collection

TODO: Some work about PGO builds automation for LLVM builds: https://issuetracker.google.com/issues/334876457

Okay, you decided to give PGO a try in your application. But where can you collect a profile? And, what is more important, where can you collect *a good* profile? There are multiple approaches - let's discuss all of them!

### Tests

The first idea that can appear in your mind - "We have unit/functional/integration/e2e-tests - let's use them as a sample workload for PGO". Please don't do that.

In most cases, the main aim of your tests is testing (!) your application in all possible situations (including rare and almost impossible scenarios aka corner cases). Instead, PGO optimizes not for all cases but for the most common from your **actual** workload. If you try to use tests as a sample workload for training PGO, you optimize your program not for the usual workload but for the corner cases. That's definitely not what you want to do. However, if you have "real-life" tests like user-scenario-based e2e tests - it's fine to use them as a PGO training workload.

### Benchmarks

Using benchmarks for training is a bit more complicated topic because the answer here is "It depends". It depends on how well your bench suite represents your actual workload. Because if you have a bench suite that measures something not so important in your application - you get completely the same problem as we have with tests.

From my experience, I met the following problems with using benchmarks as a PGO training workload:

* A bench suite measures unimportant things (TODO: add an example here from a real project).
* A bench suite does not provide good coverage for the optimized application.
* A bench suite is broken/not maintained well due to some reasons (TODO: add Quilkin and grcov and hickory-dns examples at least)

TODO: insert here a meme about a parrot who learned to say "It depends" and became an architect
TODO: add project examples where such an approach is used
TODO: write about benchmark infra standardization in some ecosystems as Rust with `cargo bench` and an effect
TODO: benchmark run fail can lead to PGO performance decrease: https://issues.chromium.org/issues/41491803#comment17
TODO: an interesting discussions about multiple PGO profiles, memory degradation and performance boost due to better PGO training set: https://issues.chromium.org/issues/41490637
TODO: Add more benchmark numbers when a PGO training workload and target workload are not completely the same

Some examples of using benchmarks as a PGO training workload:

* Firefox: [Uses](https://github.com/mozilla/gecko-dev/blob/master/build/pgo/profileserver.py#L25) WebKit performance test suite.
* Pydantic-core: [Uses](https://github.com/pydantic/pydantic-core/blob/main/Makefile#L74) own benchmarks.

Some ecosystems (like Rust) allows you to use benchmarks as a PGO training scenario in an easier way. In Rust you can use the benchmarks for PGO training phase with a simple `cargo pgo bench` command. In Rust it's possible to achieve since the community "standardized" not only how the Rust projects are built but other related routines like unit-tests, documentation, benchmarking.

It's an another example where tooling matters - if you make something convenient enough to use without (huge) pain - people will use it more frequently. I can confirm it since I used the `cargo-pgo` approach for many projects specifically for the "PGO training on benchmarks" use case - it worked flawlessly. It would be nice to see similar tools for other PGO-enabled ecosystems like C++, Fortran, etc.

Several times I heard something like "Hey, training PGO on benchmarks and then testing PGO efficiency on the same benchmark is not fair - of course, it will work! What about real-world cases?". I cannot understand such a point of view. If your benchmark represents "live" workload well - it's fine to use it for the PGO training phase. If it doesn't represent real world well - why do you have such benchmarks? :)

### Manually crafted workload

Here I collected some examples of real-life projects that use this approach for doing PGO:

* ISPC: [Has](https://github.com/ispc/ispc/tree/main/superbuild#build-process) special real-life `ispc-corpus` for PGO training purposes.
* Clang: [Uses](https://llvm.org/docs/HowToBuildWithPGO.html#selecting-benchmarks) the instrumented Clang to build Clang, LLVM, and all of the other LLVM subprojects available to it.
* Rustc: [Compiles](https://github.com/rust-lang/rust/blob/master/src/tools/opt-dist/src/training.rs#L19) a set of sample open-source crates as a training workload
* Windows Terminal: [Uses](https://github.com/microsoft/terminal/pull/10071) special PGO-oriented user test scenarios.
* Zstd: [Uses](https://github.com/facebook/zstd/blob/dev/programs/Makefile#L240) several Zstd CLI invocations with different parameters
* FoundationDB: [Uses](https://github.com/apple/foundationdb/blob/1a6114a66f3de508c0cf0a45f72f3687ba05750c/contrib/generate_profile.sh) custom workload
* Go compiler: [Compiles](https://github.com/golang/go/blob/master/src/cmd/compile/profile.sh) all targets in `std` and `cmd`.
* Foot: [Has](https://codeberg.org/dnkl/foot/src/branch/master/pgo) multiple predefined scenarios to choose from.
* gawk: Serpent OS maintainers [use](https://github.com/serpent-os/recipes/commit/b62808b8659f2a2747711550d5d72c72f623fc07) simple test workload.


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
TODO: Do not forget to add a mention about monitoring actual workload and tracking - does it match your current PGO-profile or not

### Production environment

Advantages:

* Production is the most representative environment to get the actual workload profile.
* There is no need to maintain custom near real-life workloads, sync them with the actual production workload, monitor a skew between the sample workload and the production, etc.

Disadvantages:

* It can be difficult to collect PGO profiles from external environments. E.g. let's imagine you have a mobile application. It's much more difficult to collect PGO profiles from consumer devices and transfer them to your CI environment for PGO optimization.
* Getting PGO profiles from production affects performance of your production. So do it carefully. It can be partially mitigated by using sampling PGO approach since with sampling you have an option to tweak the performance overhead via increasing/decreasing the sampling frequency ("Sampling overhead <-> Profile precision" tradeoff). Actually, instrumented PGO also can be used in production but before this, you need to test/estimate the performance penalty and decide - is it acceptable in your case or not.

Especially for this case, Google created [AutoFDO](https://github.com/google/autofdo) project. Google has a corresponding internal infrastructure for collecting PGO profiles directly from the production environment with almost no overhead. We will discuss Google's approach in-depth a bit later, no worries.

TODO: Write about the caveat of how to collect PGO profiles from the customer devices, the overhead of taking a profile from the production, etc.

---

TODO: write a short conclusion about PGO profiles collection

### PGO tips

TODO: rework this chapter

* Merging multiple profiles - works well.
* PGO works well with libraries. However, it will be a question - how to collect and distribute a profile for them? Especially if we are talking about general-purpose builds like in OS distros. Also, it's not easy to collect a profile from instrumented but non-instrumented binary (e.g. instrumented .so library which is used from Python software). It needs to be clarified but writing an instrumented wrapper right now is a recommended option.
* PGO profiles do not depend on time! However, if your code has time-dependent paths, PGO profiles could differ due to "time stuff" like time issues on a build machine, different speeds of different build machines, etc - be careful with that

## Post-Link time Optimization (PLO)

What if PGO is not enough for us and we want to squeeze even more performance "for free" with no manual optimizations? Well, in this case, the industry already has something to offer. These kinds of tools are called Post-Link Time Optimizers or simply PLO.

TL;DR, the idea behind PLO is simple - try to reduce [CPU instruction cache (I-cache)](https://en.wikipedia.org/wiki/CPU_cache#Overview) and [Instruction Translation Lookaside Buffer (iTLB)](https://en.wikipedia.org/wiki/Translation_lookaside_buffer#Overview) misses for applications. We can achieve this goal by rearranging code in our binary according to our execution profile: hot code executed together we place together in the binary, so these pieces of code highly likely will be uploaded to a CPU at the same time with reducing corresponding instruction cache misses.

Right now, there are three of the most mature tools in this area: [LLVM BOLT](https://github.com/llvm/llvm-project/blob/main/bolt/README.md) (from Facebook/Meta), [Propeller](https://github.com/google/llvm-propeller) (from Google) and Intel Thin Layout Optimizer (from Intel, obviously). Let's discuss each of them.

### LLVM BOLT

TODO: BOLT atomically updates its counters: https://discord.com/channels/636084430946959380/930647188944613406/1239768541729918976
TODO: BOLT's BAT: <https://github.com/llvm/llvm-project/blob/main/bolt/docs/BAT.md> + <https://github.com/llvm/llvm-project/blob/main/bolt/docs/Heatmaps.md>

According to its [README](https://github.com/llvm/llvm-project/blob/main/bolt/README.md), BOLT is a post-link optimizer developed to speed up large applications. It achieves the improvements by optimizing application's code layout based on execution profile gathered by sampling profiler, such as Linux `perf` tool.

TODO: write about CSIR PGO and LLVM BOLT - https://github.com/rust-lang/rust/issues/118562#issuecomment-1842738590

Please do not mess LLVM BOLT with another [bolt](https://gitlab.freedesktop.org/bolt/bolt) project - a Thunderbolt devices manager. Naming collision happened even here. Anyway, it wouldn't be a huge problem in practice - just always search for "LLVM BOLT" and you will be fine.

BOLT supports two modes: instrumentation and sampling. Instrumentation mode works semantically in the same way as instrumentation PGO - some counters are inserted by BOLT (via a dedicated BOLT instrumentation phase). When your application is executed, this counters are updated and dumped at the exit. Sampling mode collects runtime execution statistics via Linux perf, then BOLT converts the profile to a compatible format and works with it. Advantages of sampling compared to instrumentation are also pretty the same as with PGO - lower runtime overhead and no binary size increase.

From my perspective, BOLT has the following problems/caveats/nuances:

* BOLT [does not](https://github.com/llvm/llvm-project/issues/72205) support many major operating systems like Windows, macOS, *BSD. The only supported platform is Linux (ELF), even 32 bit ELF [is not supported](https://github.com/llvm/llvm-project/issues/70328). If Linux is the only target platform for you - it's fine. For many existing applications it's not the case - we need to support multiple operating systems. Unfortuantely, BOLT developers are not very interested in supporting other OS in BOLT - Facebook (the major contributor to BOLT) cares only about Linux platform because Facebook's main case for BOLT - optimizing their servers fleet, not consumer applications. Chances for contributing support for operating systems are pretty low, IMHO.
* BOLT [supports](https://github.com/llvm/llvm-project/blob/main/bolt/README.md#input-binary-requirements) only x86-64 and AArch64 architectures, AArch64 implementation is less tested than x86-64. Since BOLT works as a disassembler it means if you want to apply BOLT to other architectures like RISC-V or something like that - you need to carefully patch BOLT for that. Chances for the upstream support for other architectures are also quite low - Facebook dev team does not care much about it.
* Limited support for languages. For C, C++ and Rust binaries BOLT works well. However, for other languages you can meet different problems. Let's take a look on Golang support for LLVM BOLT: [GitHub issue](https://github.com/golang/go/issues/49031#issuecomment-1837418788) - unfortunately for now there is no way to use BOLT with Go binaries. And once again - upstream developers are not very interested in such functionality. This is quite strage because performance results are promising: [YouTube timestamp](https://www.youtube.com/watch?v=AT-ttb6VwRA&t=402s).
* Memory consumption. During the profile conversion via `perf2bolt` or the instrumentation phase - in [both](https://github.com/llvm/llvm-project/issues/62320) [cases](https://github.com/llvm/llvm-project/issues/61711) BOLT can be memory hungry. This problem can be only partially mitigated: the case with PGO profile conversion can be resolved with using smaller profiles (but it's not suitable option if you have no ability to collect a representative small profile - completely real-life case if you are trying to optimize a long-running process with multiple steps inside); instrumentation phase can be "patched" with [using](https://github.com/llvm/llvm-project/issues/61711#issuecomment-1484246795) `-strict=0` option (but it disables some optimizations in BOLT). During my BOLT tests I did another trick - just threw money into my PC and got from a friend an additional 16 Gib RAM :)
* Broken binaries - sometimes after BOLTing an application crashes. There are multiple SIGSEGV-flavoured issues for [MySQL](https://github.com/llvm/llvm-project/issues/62307), [Clang](https://github.com/llvm/llvm-project/issues/60045), [ananicy-cpp](https://github.com/llvm/llvm-project/issues/73992), [Julia](https://github.com/llvm/llvm-project/issues/89117), a shared library [issue](https://github.com/llvm/llvm-project/issues/64634). I am not saying that your binary will be broken as well, I just warned you about the issue - be ready for that. Anyway, the BOLT dev team tries to fix such errors.
* According to the [README](https://github.com/llvm/llvm-project/blob/main/bolt/README.md#for-services) file, when non-LBR profiles will be used for optimization with BOLT, the optimization result will be less efficient. However, no information is available about how much performance loss will be in practice. I understand that it highly depends on many variables like application type, target workload, etc. but as a user I need at least some initial numbers for some applications to understand the difference in practice. [Here](https://github.com/llvm/llvm-project/issues/61902) is the corresponding issue for that question in the upstream. If someone wants to perform such experiments - please ping me in the issue!
* Sampling mode works only with Linux perf profiles. If you want to use other profilers - you will need to implement some kind of profile conversion routine or tweak BOLT sources. However, since BOLT supports only the Linux platform, it shouldn't be a big limitation.
* Lack of documentation. Even if basic usage is described in the README [file](https://github.com/llvm/llvm-project/blob/main/bolt/README.md), there are many-MANY options from `llvm-bolt --help`,  that are not even mentioned somewhere else. `llvm-bolt --help | rg '\-\-' | wc -l` gives me 113, and it isn't the end since we have *hidden* options :D So `llvm-bolt --help-list-hidden | rg '\-\-' | wc -l` gives me 247 options! Don't forget that many options have multiple parameters, there different switch combinations, etc. If you to figure out what an option does **in practice** - better just ask on LLVM Discord, Discourse platform, etc. Even if from the "man page" you are able to understand an idea in general (spoiler: you don't for all of them since we are not all compiler engineers, huh), possibly important details beyond the "beginner" stuff will be unclear. Unfortunately, we don't have good BOLT user guides for now.

Quite a long list of disadvantages, isn't it? However, the biggest advantage of BOLT (IMO) is that this tool is the most used PLO tool nowadays not only inside Meta - open-source developers also try to integrate it (and some of them already added BOLT to their optimization pipelines). As a huge advantage, I want to highlight responsive developers in the LLVM Discord channel - it's a big advantage if you are hacking such things.

I collected some numbers about binary slowdown from the BOLT instrumentation:

| Application | BOLT Instrumentation to Release slowdown ratio | Benchmark |
|---|---|---|
| czkawka | 2.66x | [link](https://github.com/qarmin/czkawka/issues/1099#issue-1946612100) |
| Symbolicator | 1.64x | [link](https://github.com/getsentry/symbolicator/issues/1334#issue-2016488888) |
| typos | ~3.1x | [link](https://github.com/crate-ci/typos/issues/827#issue-1888263250) |
| pylyzer | **32x** | [link](https://github.com/mtshiba/pylyzer/discussions/80#discussion-6500001) |
| bbolt-rs | 1.33x | [link](https://github.com/ambaxter/bbolt-rs/issues/2#issue-2291476885) |
| Bend | **35x** | [link](https://github.com/HigherOrderCO/Bend/issues/517#issue-2319505187) |
| resvg | ~1.5x | [link](https://github.com/RazrFalcon/resvg/issues/765#issue-2314946307) |

At least in the cases above, nothing too critical except the `pylyzer` case. For some unknown yet reason, the actual slowdown from the BOLT instrumentation is a quite big. I [asked](https://discord.com/channels/636084430946959380/930647188944613406/1233792727498363020) about it in the `#bolt` channel at LLVM Discord - let's wait for the answer.

What about binary size increase from instrumentation?

| Application | Release size | Instrumented size | BOLT optimized size | Instrumented to Release ratio | Language |
|---|---|---|---|---|
| Symbolicator | 27 Mib | 136 Mib | 33 Mib | ~5x | Rust |
| pylyzer | 27 Mib | 126 Mib | 37 Mib | ~4.7x | Rust |
| angle-grinder | 45 Mib | 69 Mib | 47 Mib | 1.53x | Rust |
| prettyplease | 1.6 Mib | 15 Mib | 4.3 Mib | 9.4x | Rust |
| bbolt-rs | 1.3 Mib | 14 Mib | 4.3 Mib | 10.8x | Rust |
| CreuSAT | 623 Kib | 9.2 Mib | 4.1 Mib | 14.7x | Rust |
| Bend | 4.6 Mib | 20 Mib | 8.6 Mib | Rust |
| resvg | 4.8 Mib | 20 Mib | 8.7 Mib | ~4.1x | Rust |

The binary size increase is bigger compared to PGO instrumentation. Mostly this is due to a [limitation](https://github.com/Kobzol/cargo-pgo?tab=readme-ov-file#bolt--pgo) with stripping binary - you can get some linking errors if you strip your binary during the BOLT instrumentation phase. Except that, nothing criminal. Remember one more thing - you can try to avoid the slowdown and binary size increase with using BOLT with Sampling instead of Instrumentation.

I know, I know - not a huge benchmark list (compared to the PGO lists above). That's only because I mostly spent my time with more stable and battle-tested PGO approach! You also may noticed that all my numbers are for Rust applications - it's only because I started performing BOLT benchmarks together with PGO for applications (and one of the reasons - *very* convenient [way](https://github.com/Kobzol/cargo-pgo?tab=readme-ov-file#bolt--pgo) to use BOLT together with PGO). Later more numbers for more programs and more programming languages can (not a promise!) be provided. At least having small amount of benchmarks is better than nothing ;)

BOLT is already integrated into the optimization pipelines in several projects:

* Rustc: [GitHub PR](https://github.com/rust-lang/rust/pull/116352). The `rustc` compiler is already PGO + BOLT optimized for Linux platforms.
* CPython: [GitHub PR](https://github.com/python/cpython/pull/95908)
* Pyston: [README](https://github.com/pyston/pyston#building)
* Clang: [CMake script](https://github.com/llvm/llvm-project/blob/main/clang/cmake/caches/BOLT.cmake)

Wanna more materials about BOLT? Of course, I have something to share with you:

* BOLT original paper: [Facebook engineering blog](https://research.facebook.com/publications/bolt-a-practical-binary-optimizer-for-data-centers-and-beyond/)
* 2016 EuroLLVM Developers' Meeting: Maksim Panchenko "Building a binary optimizer with LLVM": [Youtube](https://www.youtube.com/watch?v=gw3iDO3By5Y)
* Facebook Research paper "Lightning BOLT: Powerful, Fast, and Scalable Binary Optimization": [Facebook engineering blog](https://research.facebook.com/publications/lightning-bolt-powerful-fast-and-scalable-binary-optimization/)
* Optimizing Linux kernel with BOLT: [Youtube](https://www.youtube.com/watch?v=ivTCCTSMGZg)
* 2023 EuroLLVM Tutorial: Maksim Panchenko and Amir Ayupov "Developing BOLT pass": [Youtube](https://www.youtube.com/watch?v=kPKPkg5ugMg)
* Some additional [materials](https://discourse.llvm.org/t/practical-compiler-optimizations-for-wsc-workshop-us-llvm-dev-meeting-2023/73998) about BOLT and similar things
* [Ideas](https://discourse.llvm.org/t/bolt-open-projects/61857) about future BOLT improvements

### Propeller

TODO: Do I have any benchmarks? I guess putting from the official Google papers is fine here
TODO: interest in Propeller: https://issues.fuchsia.dev/issues/42162813 + https://issuetracker.google.com/issues/200184657
TODO: LBR limitation: https://issuetracker.google.com/issues/200184657#comment14

Additional links to read:

* Original Propeller repo: [GitHub](https://github.com/google/llvm-propeller)
* 2019 LLVM Developers’ Meeting: S. Tallam “Propeller: Profile Guided Large Scale Performance...”: [Youtube](https://www.youtube.com/watch?v=DySuXFGmB40)
* Some discussions about BOLT vs Propeller: [Google groups](https://groups.google.com/g/llvm-dev/c/ef3mKzAdJ7U/). Some people already [asks](https://github.com/google/llvm-propeller/issues/10) about unifying PLO efforts between BOLT and Propeller dev teams. However, I don't see such movements at least yet. Probably, in the future we can get something similar if BOLT and Propeller will be a part of LLVM: BOLT is already [merged](https://www.phoronix.com/news/LLVM-Lands-BOLT) into LLVM, I guess Propeller also will be fully merged into LLVM one day.

### Intel Thin Layout Optimizer (TLO)

Intel recently [open-sourced](https://github.com/intel/thin-layout-optimizer) its vision about how PLO should be implemented - Intel Thin Layout Optimizer (or simply TLO because we love abbreviations).

For now, I see the following limitations/risks in this tool:

* Doesn't work without LBR. It can be a huge limitation in practice.
* Too young tool -> not so much experience in the industry with it + possible many uncovered yet issues.
* Works only with Linux perf profiles. So no support for other platforms. It's especially interesting detail since Intel has a Sampling Enabling Product (SEP) technology that also works on other platforms except Linux.
* According to the TLO's [wiki](https://github.com/intel/thin-layout-optimizer/wiki#5-performance-comparison), there are questionable performance results for now compared to BOLT.

For better understanding design choices, why the tool was implemented, etc. I recommend you to read the official Wiki [page](https://github.com/intel/thin-layout-optimizer/wiki). I [asked](https://github.com/intel/thin-layout-optimizer/issues/2) many questions about several TLO implementation details from different perspectives - you can check the answers from maintainers (and ask your questions ;)

I don't have enough experience with the tool since my CPU doesn't support LBR or similar technology. My current conclusion - the tool is too inmature and there are no serious reasons to use it instead of BOLT. However, the situation can be changed in the future.

### Other tools

Binary optimization is not a novel technique - this idea is quite old. Since this, [there are](https://en.wikipedia.org/wiki/Object_code_optimizer) other binary optimization tools like [IBM z/OS Binary Optimizer](https://www.ibm.com/products/automatic-binary-optimizer-zos). Cannot say much about them since I didn't try them. If you have some practice in this field - please share your experience! I can predict that many other tools are too experimental (read it as "cannot be used in a serious production") or too niche to be widely-adopted across the industry.

### What to choose?

Since we have so many PLO tools, the obvious question appears - what should I use? From my experience, right now BOLT will be the best option to start with if we are talking about PLO usage. Start by default with BOLT. If it works for you - great, stay with it. If you meet some limitations and they cannot be easily mitigated - consider switching to another tools like Propeller and TLO.

Someone may ask - why do we have PGO and PLO at the same time? Why do need two dedicated approaches that do the same things - optimize an application based on runtime statistics. That's a very good question.

TODO: add thoughts about unification efforts between BOLT and Propeller: https://github.com/google/llvm-propeller/issues/10
TODO: Android's toolchain uses BOLT instead of CSFDO: https://issuetracker.google.com/issues/223669638#comment16
TODO: Some notes about BOLT vs Propeller for Chromium: https://issues.chromium.org/issues/40740472

## LTO, PGO, PLO state in the ecosystem

TODO: PLO has no integration in the build systems

In this chapter I want to discuss PGO adoption across different parts of our ecosystem: applications (open-source and proprietary), packaged software in different OS distributions (and not only in them), and cloud compute providers.

### Open-source projects

I decided to start my PGO enablement investigation from the most available for research purposes place - open source software. Since I am a regular open source tools user, it's also nice to contribute something back to the community.

TODO: describe more about PGO integration into the build scripts
TODO: write some statistics about how projects reacted to PGO reports to them
TODO: Get more information about PGO states in other browsers like Safari, Brave ([GitHub comment](https://github.com/brave/brave-browser/issues/20560#issuecomment-1658782341)), Yandex.Browser and others . It can be expected that usually for modern major web browsers PGO is enabled. However, not for all of them - put here a link to the PGO issue in Ladybird browser.
TODO: PGO is not enabled for an LLVM fork: https://github.com/Xilinx/llvm-aie Even if upstream supports PGO, derivatives can just skip enabling it

PGO usually is **not** enabled by the upstream developers due to a lack of support for sample load or a lack of resources for the multi-stage build. So please ask maintainers explicitly about PGO support addition.

#### Desktop environments (DE)

One of my ideas for improving PGO coverage across the ecosystem was proposing PGO enabling for desktop environments (DE) like Gnome and KDE. These "umbrella" projects have many different software and trying to enable PGO across one DE will cover a lot of projects. Sounds great, doesn't it? And of course, the idea failed.

At FOSDEM 2024 I had several conversations about enabling PGO with Gnome and KDE developers (their stands were placed together, huh). Opinions about PGO were almost identical - "Yes, it would be useful to give it a try. However, we need to figure out how we can collect good PGO profiles. And we need more human resources to implement it. Btw, contributions are welcomed!". I interpret it as "If you are interested in it - you need to implement it.". Honestly, pretty fair point of view - that's how open-source works. At least the developers suggested me to create some sort of discussions on the corresponding platforms. I did it for [Gnome](https://discourse.gnome.org/t/evaluate-using-profile-guided-optimization-pgo-and-post-link-optimization-plo-for-gnome-projects/19341) and [KDE](https://discuss.kde.org/t/evaluate-using-profile-guided-optimization-pgo-and-post-link-optimization-plo-for-kde-projects/10290) - unfortunately, not so much activity. Do I need to start a conversation somewhere else? However, at least Gnome recently posted several articles ([one](https://blogs.gnome.org/chergert/2024/03/21/bolting-libraries/), [two](https://blogs.gnome.org/alatiera/2024/03/26/thoughts-on-employing-pgo-and-bolt-on-the-gnome-stack/)) about PGO and PLO so probably at least *some* interest exists. I just recommend starting with regular PGO, and only after that start thinking about BOLT integration - PGO is a much more stable technology with fewer limitations.

By the way, discussions at FOSDEM were pretty interesting - I learnt a lot of stuff about internal benchmark systems in Gnome and KDE.

For anyone who wants to complain about "Where is my favorite X DE???" - I just started from the most widely-used DE in the Linux ecosystem. If you are interested in bringing PGO to other great projects like LXDE, Cinnamon, etc. - you are welcome! I will be happy to see PGO adoption in as many projects as possible! I didn't create corresponding discussions into all DEs only because my resources are kinda limited. Maybe in the future, my backlog will be a bit shorter (it won't) and I'll create more PGO discussions for more DEs.

#### Documentation

TODO: Lady Dreidra's documentation about PGO - <https://github.com/Eliah-Lakhin/lady-deirdre/commit/63b54ec04b2bac26812f066737f807f6a5b21ebf> - is not visible for users of the library. It should be placed in a more visible place like the official documentation (the place which users read) - <https://lady-deirdre.lakhin.com/>

Some projects already have dedicated pages about PGO:

* ClickHouse: [Profile-Guided Optimization](https://clickhouse.com/docs/en/operations/optimizing-performance/profile-guided-optimization)
* Databend: [PGO](https://databend.rs/doc/contributing/pgo)
* Vector: [PGO](https://vector.dev/docs/administration/tuning/pgo/)
* Nebula: [AutoFDO](https://docs.nebula-graph.io/3.5.0/8.service-tuning/enable_autofdo_for_nebulagraph/)
* GCC: Official [Docs](https://gcc.gnu.org/install/build.html), section "Building with profile feedback"
* Clang: [How to build with PGO](https://llvm.org/docs/HowToBuildWithPGO.html) and [Advanced builds](https://llvm.org/docs/AdvancedBuilds.html) instructions
* Rustc: [Optimized build](https://rustc-dev-guide.rust-lang.org/building/optimized-build.html#profile-guided-optimization)
* tsv-utils: [Building with Link Time Optimization and Profile Guided Optimization](https://github.com/eBay/tsv-utils/blob/master/docs/BuildingWithLTO.md)

I hope having PGO-related documentation in the project helps users in the following ways:

* Find information about PGO usage with the project. It also improves PGO visibility overall. Without corresponding documentation users with high probability will not find the possibility to optimize your project with PGO even if you invested a lot of resources into the actual PGO integration.
* Find information about how much performance improvement can be expected from enabling PGO for the project at least in some cases. Of course, a user can have different cases and can get in reality worse (or better - who knows) results with PGO for the project - but having some kind of baseline is important during the idea evaluation stage.
* Find information about how the project can be built with PGO. Sometimes it's *just* a build system switch, sometimes it's a more sophisticated process with direct playing around compiler switches, PGO profiles, etc. Having this process documented lowers **a lot** PGO entrance level for the project because fighting with unfamiliar (to the project user) build scripts is a **very** demotivating task. Developers' UX matters here too, trust me.

One more wish - please, place PGO-related documentation somewhere in the visible place. It can be just a README file note (for smaller projects) or a dedicated page (for larger projects with a dedicated website). It would be awesome if you have some kind of search over the documentation where I can type "PGO" and go to the corresponding piece of documentation. Otherwise finding the corresponding page can be a difficult task. E.g. let's check MariaDB case. MariaDB [has](https://mariadb.com/files/MariaDBEnteprise-Profile-GuidedOptimization-20150401_0.pdf) PGO documentation but, unfortunately, it's in the form of PDF file. I even don't know how to find this file via the official [documentation](https://mariadb.com/kb/en/documentation/) - I found this guide only via Google search! MariaDB's built-in search engine on its website cannot find anything related to PGO. I discussed this topic with MariaDB's devs at FOSDEM 2024 and they promised me take a look at it. I remember your promise ;)

All information above targets mostly PGO documentation for applications. What about libraries? Since libraries are *usually* smaller than an application, possibly leaving some PGO benchmarks in the README file will be enough (something like [this](https://github.com/lovasoa/serde-sqlite-jsonb/pull/2) one). If your library supports building artifacts like dynamic, static libraries or some packages like Python wheels and you support building them with PGO (like `pydantic` project [does](https://github.com/pydantic/pydantic-core/pull/678)) - document this process as well.

#### Prebuilt binaries

Even if PGO is somehow implemented in the project, it doesn't mean at all that binaries provided by the project will be optimized with PGO! E.g. ISPC [supports](https://github.com/ispc/ispc/blob/main/superbuild/README.md) building with PGO but provided binaries [don't](https://github.com/ispc/ispc/issues/2687) use this optimization. I am pretty sure that for other projects like ClickHouse and YugabyteDB situation is the same - they support somehow (via a dedicated build switch or at least describe building with PGO in the documentation) PGO build but don't use for their binaries. The applies to their derivatives like Altinity [builds](https://altinity.com/altinity-stable/) for ClickHouse.

Why? In many cases developers cannot predict target workload for their users - it's especially true for performance-critical things like databases. Access patterns can be too different, and upstream developers can be just too conservative to preoptimize their binaries with some reference workload. Maintaining several binary versions? Yeah, it's possible but additional maintenance cost. Collect actual workload from customers? Yeah, it's possible to do but requires additional efforts from developers and establishing this process + additional maintenance costs once again.

### OS distributions

TODO: finish the chapter
TODO: Some improvements around PGO generation in CrOS (ChromeOS): https://issuetracker.google.com/issues/298247335

Even if PGO is supported by a project, it does not mean that your favorite Linux distro builds this project with PGO enabled. For this there are a lot of reasons: maintainer burden (because we are humans (yet)), build machines burden (in general you need to compile twice), reproducibility issues (like profile is an additional input to the build process and you need to make it reproducible), a maintainer just don't know about PGO, etc.

So here I will try to collect information about the PGO status across the Linux distros for the projects that support PGO in the upstream. If you didn't find your distro - don't worry! Just check it somehow (probably in some chats/distros' build systems, etc.) and report it here (e.g. via Issues) - I will add it to the list.

* GCC:
  - Note: PGO for GCC usually is not enabled for all architectures since it requires too much from the build systems
  - Debian: yes
  - Ubuntu: same as Debian
  - RedHat: Yes. And that is the reason why PGO is enabled for GCC in all RedHat-based distros.
  - Fedora: [yes](https://src.fedoraproject.org/rpms/gcc/blob/rawhide/f/gcc.spec#_1182)
  - Rocky Linux: [yes](https://git.rockylinux.org/staging/rpms/gcc-toolset-11-gcc/-/blob/r8/SPECS/gcc.spec#L1130)
  - Alma Linux: [yes](https://git.almalinux.org/rpms/gcc/src/branch/c8/SPECS/gcc.spec#L1164)
  - NixOS: [no](https://github.com/NixOS/nixpkgs/pull/112928)
  - OpenSUSE: [yes](https://build.opensuse.org/package/view_file/openSUSE:Factory/gcc12/gcc12.spec), see line `2414`
* Clang:
  - Binaries from LLVM are already PGO-optimized (according to the [note](https://apt.llvm.org/) about using "stage2" build - it's a PGO optimized build)
  - RedHat (CentOS Stream): [no](https://bugzilla.redhat.com/show_bug.cgi?id=2224925)
  - Fedora: [no](https://bugzilla.redhat.com/show_bug.cgi?id=2156679)
  - AlmaLinux: [no](https://bugs.almalinux.org/view.php?id=350)
  - Rocky Linux: [no](https://bugs.rockylinux.org/view.php?id=1618)
  - NixOS: [no](https://github.com/NixOS/nixpkgs/issues/208188)
  - Arch Linux: sent an email to the Clang maintainer in Arch Linux - no response yet
* Rustc:
  - Fedora: [yes](https://src.fedoraproject.org/rpms/rust/blob/rawhide/f/rust.spec#_773)
* CPython:
  - Fedora: [yes](https://src.fedoraproject.org/rpms/python3.11/blob/f38/f/python3.11.spec#_73). Also, check [this](https://lists.fedoraproject.org/archives/list/devel@lists.fedoraproject.org/thread/2H4ZDRR326XAZ2EPCQKTNRMQYG5YZQ2K/) discussion. I guess other RedHat-based distro builds are the same for this package (however I didn't check it but Rocky Linux is the [same](https://git.rockylinux.org/staging/rpms/python3.11/-/blame/r8/SPECS/python3.11.spec#L70)).

Sometimes PGO is not supported in the upstream for some reason. In this case, a package manager/OS maintainers can decide to enable PGO on their own and patch the application correspondingly. Here are some examples:

* libtre in FreeBSD: [Makefile](https://cgit.freebsd.org/ports/tree/textproc/libtre/Makefile)

TODO: Request for enabling PGO for the Rustc compiler in Fuchsia: https://issues.fuchsia.dev/issues/42083760
TODO: write about Propeller adoption - nowhere adopted

What about PLO adoption?

TODO: what about Windows package managers? BOLT is unavailable for Windows right now but PGO can be used too.

Here we track LLVM BOLT enablement across various projects in various OS-specific build scripts:

* Clang:
  - [Gentoo bugtracker](https://bugs.gentoo.org/907931)
* GCC: TODO
* Rustc:
  - Fedora: no
  - RedHat: no
* CPython: TODO
* Pyston: TODO  

Meta-issues about PGO and LLVM BOLT usage in different OSs and package managers:

* Fedora: [Bugzilla](https://bugzilla.redhat.com/show_bug.cgi?id=2249353)
* RedHat: [JIRA](https://issues.redhat.com/browse/RHEL-16308)
* ClearLinux: [GitHub issue](https://github.com/clearlinux/distribution/issues/2996)
* CachyOS ([Website](https://cachyos.org/)): according to the search over its GitHub repositories - they are trying to integrate BOLT as much as possible
* OpenSUSE: Cannot create an account to create a corresponding issue
* Ubuntu: [Ubuntu forums](https://ubuntuforums.org/showthread.php?t=2492480&p=14165131#post14165131)
* Alpine Linux: [Gitlab issue](https://gitlab.alpinelinux.org/alpine/aports/-/issues/15465)
* Mageia: [Bugzilla](https://bugs.mageia.org/show_bug.cgi?id=32511)
* Void Linux: [GitHub issue](https://github.com/void-linux/void-packages/issues/47215)
* Chromebrew: [GitHub discussion](https://github.com/chromebrew/chromebrew/discussions/8941)
* Homebrew: [GitHub discussion](https://github.com/orgs/Homebrew/discussions/4902)
* Spack: [GitHub discussion](https://github.com/spack/spack/discussions/41090)
* Vcpkg: [GitHub discussion](https://github.com/microsoft/vcpkg/discussions/35190)
* FreeBSD: [FreeBSD forum](https://forums.freebsd.org/threads/expand-profile-guided-optimization-pgo-usage-across-freebsd-packages.91034/)
* Conan: [GitHub issue](https://github.com/conan-io/conan-center-index/issues/21245)
* MacPorts: [Ticket](https://trac.macports.org/ticket/68746#ticket)
  - They said this question should be discussed in mailing lists
* LLVM-mingw: [GitHub issue](https://github.com/mstorsjo/llvm-mingw/issues/383)
* MinGW repo: [GitHub issue](https://github.com/msys2/MINGW-packages/issues/19273)
* CBL-Mariner: [GitHub discussion](https://github.com/microsoft/CBL-Mariner/discussions/8072)

How PGO can be integrated on the OS level? One of the ideas can be creating an additional version of the package like "PGO-optimized packages"

TODO: what about distribution-wide performance changes like https://www.phoronix.com/news/Ubuntu-x86-64-v3-Experiment ? PGO can be in the same field

---

TODO: merge with previous one

## PGO state across Linux distributions - TEMPORAL

TODO: finish the chapter
TODO: some distributions tend to enable PGO and BOLT in more cases like Gentoo, ClearLinux, CachyOS, SerpentOS. Clear Linux PGO packages: https://github.com/search?q=org%3Aclearlinux-pkgs+%22pgo+%3D+true%22&type=code + PGO integration as a USE flag
TODO: someone even proposed me to become a maintainer to push PGO activities: https://bugs.mageia.org/show_bug.cgi?id=32511#c1
TODO: Write about an observation that if in the upstream there is no PGO support - maintainers almost always do not enable PGO in their packages
TODO: PLO has no integration in the Linux distros yet
TODO: sometimes PGO PRs just dying - https://github.com/void-linux/void-packages/issues/29526

In many cases, we don't use binaries directly from developers or we don't rebuild all the things in our perimeter (despite knowing such security requirements in some areas!) - we use *prebuilt* binaries from our favorite OS distribution. But if the binaries are prebuilt by someone, there is a chance that PGO can be disabled even if the PGO-optimized build is supported by the upstream. How does it work in practice?

 Let's take a look at some examples:

Unfortunately, PGO is rarely enabled in the provided by OS repositories. I see the following reasons for that:

* Sometimes PGO is disabled on some platforms due to a lack of build resources
* Sometimes PGO is disabled due to reproducible builds concerns. Concern about reproducing the profile. Bugs: https://bugzilla.opensuse.org/show_bug.cgi?id=1040589, NixOS comment (find and link it here)
* Could we trust the profiles from the upstream to use them instead of our own? Good question. That’s why is important to commit scripts for reproducing the profile (and still can differ due to time-based things like I had in YDB)

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


#### PGO integration into OS-specific build systems

TODO: SerpentOS example - https://github.com/serpent-os/boulder/blob/main/data/macros/actions/pgo.yml

What about PLO integration into operating systems? Since the approach is much younger than PGO you can expect that adoption is lower compared to PGO - and you will be right.

From the PLO perspective, right now only LLVM BOLT is considered *sometimes* as a tool for that kind of optimization - almost nobody talks about Google's Propeller.

TODO: finish chapter

https://github.com/getsolus/packages/blob/main/packages/l/llvm/package.yml#L116 - example of BOLT integration in the upstream

#### PGO / PLO state across package managers

### SaaS platforms

Nowadays many people are crazy about clouds. So in my mind raised an idea - what is the current state of PGO usage in different SaaS environments? Quick googling gave me nothing so I just asked major public clouds via the official channels about their PGO state. Especially I was interested in the open-source projects like databases that are provided via SaaS model by public cloud since for such projects I already had a lot of benchmarks and was ready to demonstrate them (they are all public anyway). I asked multiple public cloud about that, and here we go:

* AWS: no answer ([Repost AWS post](https://repost.aws/questions/QUzEamhSqBRS-gkAbvCjchPQ/profile-guided-optimization-pgo-and-post-link-optimization-plo-for-aws-products)).
* Azure: no answer ([Feedback Azure post](https://feedback.azure.com/d365community/idea/da9e736e-0bb1-ee11-92bc-000d3a037f01)).
* GCP: no answer ([GCP community post](https://www.googlecloudcommunity.com/gc/Databases/Profile-Guided-Optimization-PGO-and-Post-Link-Optimization-PLO/m-p/697535)).
* Cloudflare: no answer ([a topic on the Cloudflare community platform](https://community.cloudflare.com/t/using-profile-guided-optimization-pgo-for-cloudflare-products/587534).
* OVH: no answer yet (waits for moderation).
* Yandex and VK clouds: no answer. Waits for moderation or just silently rejected - I don't know.

Well... Bad results but expected tbh. However, since the PGO support is missing in many of these projects (just guessing!) their customers get worse experience that it could be in practice: higher resource consumption, slower query execution, etc.

From chatting with my friends and asking proper questions at the [Highload](https://highload.ru/) conference I can say that local public clouds like Russian-based Yandex Cloud or VK Cloud also don't use PGO internally for their products. I am pretty sure that the situation with other clouds is pretty much the same.

In general, communication with corporations sucks a lot. Yeah, I know that it's obvious, they are big companies with multi-level support lines, a lot of bureacracy inside and bla-bla-bla. But I needed to highlight it once again. Even developers from these companies confirm (in private conversations) that this communication way sucks (ofc no names here)! Probably here there is some work for community managers.

From my experience, it's much better to find the right person from interesting for you corp via mailing lists/Twitter/HackerNews/etc. and write directly to them. In this case, with a higher probability, you will get an answer (hopefully without NDA violation, haha) from a knowledgable about your question person and not from a first-line "support" or even an LLM-based operator. However, no guarantees here either. Sometimes your requests will become forgotten, and sometimes people just don't check their email or simply ignore you. E.g. I sent an email to Brendan Gregg (I hope you know who is this performance semi-god. If don't - [you are welcome](https://www.brendangregg.com/)) about using PGO for the Intel products - no response.

I hope that someone from such corporation will read this article and pass it to the right people inside a company. Probability isn't huge, I know, but at least I tried to do it.

### Proprietary software

Everyday we use not only open-source software, that can be (simply) optimized with LTO, PGO, PLO and other fancy stuff by our own hands in a Gentoo-style but also some proprietary software where we have no such an option. And still proprietary software performance can be valuable for us: CI performance with closed-source compilers, IDE performance, closed-source database performance (in both on-premise and SaaS versions) - there are many of them. So what could we do instead?

I have tried to test a silly thing: write directly to the companies with the idea of optimizing their software with PGO. Here are some results:

* Sent an email to <opensource@apple.com> about enabling PGO for Apple products like Apple Clang, prebuilt FoundationDB, etc - no response.
* Created the [topic](https://forums.percona.com/t/profile-guided-optimization-pgo-for-databases/24035) on the official Percona forum about optimizing its products with PGO - no response.
* Sent an email to Nvidia via Developer Contact email about enabling PGO for their HPC compilers with all required information about PGO. Only got a suggestion about creating a topic on their HPC forum. I have tried multiple times to create an account on this resource but with no success due to unknown reasons. No more responses from them by email, btw.
* Sent an email about PGO to https://soqol.ru/ via the official contact form on the website - no response about PGO.
* Sent an email to [NauEngine](https://nauengine.org/) devs about evaluating PGO for this game engine - and **got a response** that PGO information was sent to development teams. Great! I am waiting for the actual release to check PGO availability in the engine.
* Sent a request to [Arenadata DB](https://arenadata.tech/en/products/arenadata-db/) via the feedback form on the website - but no response. Maybe my email is not enterpris-ish enough?
* Sent an email to Cloudflare via https://sinkingpoint.com/ - sent 09.01.2024, got a response in a few hours! Cloudflare already uses PGO internally. For which projects, and what are the performance gains from it - still unknown. It needs to be clarified later.

As expected - in many cases didn't get a valuable response. Proprietary. Proprietary never changes.

So I don't have success stories to share with you but I have several ideas. Maybe you will be motivated enough to try them!

* If you have an established communication channel with a company, you can, *just in case*, discuss with them enabling PGO for their products. We are all humans and share experience with each other. So maybe developers on the other side will be interested enough in enabling PGO for used by you products.
* Some companies have a collaboration model via money *donation*: if you are interested in having some feature in their product - pay them and they will try to implement it (hopefully not in a Q6 since as usual Q5 is already fully loaded!). So if you are interested in improving performance - you can try to pay for that performance. However, I completely understand that defending such a budget on your side can be a problem. As usual - it depends on your context.
* If you a developer of such a product and reading this article AND you are interested in PGO - just bring it to your daily meeting, internal mailing list, etc. That's how at least an internal conversation can be started.

I think we can try to change the situation since we have so many suitable for PGO software (like [HighTec compilers](https://hightec-rt.com/en/products/development-platform)). The situation is similar to the SaaS section above - please send the article to the right people inside your company if you can. Thank you in advance.

## Why am I writing this?

TODO: finish

* I like performant applications
There are multiple reasons for that:
* For me, it would be easier to work in the IT industry, where we have "PGO by default" mindset. Because with faster software it's easier to achieve the required NFR (Non-Functional Requirements) before the horizontal scaling questions.
* Because I can
* Just for lulz

* [Awesome PGO](https://github.com/zamazan4ik/awesome-pgo). It's my repository, where I collect **everything** about PGO: benchmarks for different software, bugs in compilers, tracking PGO issues in multiple projects, etc. If anyone asks you something like "Hi ${PERSON_NAME}! Could you please send me a link to dive into PGO?" - it should the first link appeared in your mind to share! From my side, I will try to maintain it on the corresponding quality level (at least for some time).

## Test environment

### Hardware and OS

TODO: add a photo of my Mac with my PC test setup (add a screen with Yorha units with the question "Are Yorha units PGO-optimized or not?")

During the tests, I used two test setups: my Windows-Linux dual-boot PC and Macbook with macOS.

PC:

* CPU: AMD Ryzen 9 5900x
* RAM: 48 Gib of some default non-RGB RAM
* SSD: Samsung 980 Pro 2 Tib
* OS: Fedora 38/39
* Kernel: Linux kernel 6.* (mostly 6.2.* - 6.8.*)

Macbook M1 Pro:

* CPU: Apple M1 Pro 6+2
* RAM: 16 Gib
* SSD: 512 Gib
* OS: macOS (mostly 13.* and 14.* versions aka Ventura and Sonoma)

In the article, my PC setup is called "Linux", and my Macbook as "macOS". Most of the tests are done with Linux-based since this OSes are mostly used on servers (at least where I prefer to work), they are highly-tweakable, and I have a lot of practice with them. Sorry Windows people, I have no desire to perform all the tests on Windows because I am not interested enough in this platform from the PGO perspective yet (but I fully understand the importance of Windows nowadays - no worries!). But anyway - it should work (and works) on this OS completely in the same way (at least from the instrumentation PGO perspective).

### Compilers

For all Rust projects, I use the `rustc` compiler with LLVM backend, versions somewhere in the 1.68 - 1.73 range. For C and C++ I prefer using Clang 16, sometimes I use GCC 13 as an additional compiler. I have nothing against GCC and GPL - I just prefer LLVM and the ecosystem around LLVM due to better flexibility, from my point of view.

All these compilers are in their "default" state with no custom patches from my side: `rustc` from the official website, Clang and GCC from the official Fedora repository.

## Stories

During my PGO tests across the ecosystem, I collected some interesting PGO-related non-technical stories. I think it would be an interesting thing to share them with you and learn some lessons. Some of them are funny too!

### How to find a PGO prey

For performing PGO benchmarks at first I need to find suitable projects for the tests. When I was trying to find suitable projects for PGO, I found the following sources the most helpful:

* Reddit. The [Rust subreddit](https://www.reddit.com/r/rust/) was the most helpful. Some PGO issues were created after several **minutes** after the publication (the fastest "Evaluate using Profile-Guided Optimization (PGO)" issue generator in the world!)
* Hackernews. That's a great place to learn about something new and hyping (and create a PGO issue for them of course!)
* GitHub [trending](https://github.com/trending/rust?spoken_language_code=). I was using it to scrape the most starred applications in C, C++, and Rust domains. Scrape, evaluate PGO applicability, create an issue - a simple algorithm, isn't it?
* Different "Awesome-X" repos. Oh, these sources are my favorite! In one place I get a lot of applications for the same domain. E.g. we see that PGO shines in the compilers domain. Just use [awesome-compilers](https://github.com/aalhour/awesome-compilers), evaluate **all** compilers (okay, not all compilers - all alive compilers), and create for them a PGO issue. As far as I remember, I used the following "awesome-*" repos: [awesome-compilers](https://github.com/aalhour/awesome-compilers), [(awesome-)static-analysis](https://github.com/analysis-tools-dev/static-analysis), [awesome-llvm](https://github.com/learn-llvm/awesome-llvm), [awesome-rust](https://github.com/rust-unofficial/awesome-rust), [awesome-cpp](https://github.com/fffaraz/awesome-cpp), [awesome-game-engine](https://github.com/stevinz/awesome-game-engine-dev), [awesome-rust-formalized-reasoning](https://github.com/newca12/awesome-rust-formalized-reasoning). Or even a simple [list](https://gist.github.com/sts10/daadbc2f403bdffad1b6d33aff016c0a) of Rust command-line tools.
* Other domain-specific software registries. For databases, I used [Database of Databases](dbdb.io) (nice registry btw),for cloud-native apps (whatever it means) I used [CNCF landscape](https://landscape.cncf.io/), for ML stuff - [AI & Data landscape](https://landscape.lfai.foundation/).
* Categories in package managers. Like a "Lang" group in FreeBSD [ports](https://cgit.freebsd.org/ports/tree/lang).
* Sometimes recommendations from the github.com main page were quite interesting to investigate as well!

In each project I quickly searched for PGO existence with the following "algorithm":

* Search over issues/discussions for "PGO", "Profile guided", and "FDO" keywords.
* (rip)grep sources for PGO-related compiler flags like `-fprofile-generate`, `-fprofile-use`, `-fprofile-instr-generate`, etc.
* Checking for words like "performance", "blazing", "benchmark", etc. - it increases chances that the project cares about performance so my PGO suggestions less likely will be rejected. So if you don't want to see my PGO issues - just don't claim that you are blazing fast :D

If I found something related - good catch! It's a chance to learn something new about PGO in another project. If I found nothing - well, the issue about PGO can (and will) be created.

### Pastebin vs GitHub

If you check the PGO benchmark reports, you can find one detail - early PGO results are saved to Pastebin but later reports use gist.github.com service. There is a simple reason for that change. One day I did so many benchmarks in one day for different projects that reached the daily limit for pastes! It wasn't a big issue - I quickly switched to the GitHub service that has no such low limits. Seems like my PGO productivity was too high for Pastebin service :)

### ClickHouse

PGO-optimizing ClickHouse was fun for several reasons. At first, I was excited to get any positive results since ClickHouse has a reputation of heavily-optimized software, so achieving even few percent is a great result.

For the PGO training phase, I decided to use [ClickBench](https://github.com/ClickHouse/ClickBench) - the bench suite from the ClickHouse dev team that simulates typical OLAP queries. Here I met the first problem - due to some implementation details, ClickHouse in Instrumentation mode works ridiculously slow - the actual slowdown is more than 300 times (it's not a mistake - I rechecked the results multiple times)! So to collect PGO profiles from the whole ClickBench, I was waiting for more than 30 hours! However, it was not so critical since I (ab)used beer-based time travel mechanics ;)

Another trap was found while applying PLO optimization with LLVM BOLT to ClickHouse. At first, I tried to use LLVM BOLT in the instrumentation mode (since my CPU does not support LBR (BRS in my case since I have AMD) but BOLT recommends using `perf` profiles with LBR enabled). But quickly met a limitation - [awfully huge](https://github.com/llvm/llvm-project/issues/62320) memory consumption during the instrumentation ClickHouse binary phase. Since the issue was not resolved, I gave up on the idea of using BOLT instrumentation and went with BOLT via sampling instead (even without LBR, yeah).

We discussed [Alexey Milovidov](https://github.com/alexey-milovidov) (ClickHouse co-founder and CTO) benchmark results and the slowdown issue. It was a really nice and helpful conversation. Alexey said that such a slowdown could come from the ClickHouse implementation details: there are several **very** hot loops inside ClickHouse. If these loops are affected by instrumentation (they certainly are affected because that's how PGO instrumentation works) - there will be a huge runtime slowdown. However, Alexey was quite impressed with the results so he suggested contributing PGO-related documentation to ClickHouse. I agreed with the idea, and we prepared [this](https://clickhouse.com/docs/en/operations/optimizing-performance/profile-guided-optimization) page in the official documentation. Overall, communicating with the ClickHouse dev team was a good example of nice collaboration between project devs and one-time external contributors - nice job!

### YDB

When I was working on testing PGO with different databases, I found [YDB](https://ydb.tech/) - an interesting database from Yandex. I decided to optimize it with PGO. YDB is written in C++ so PGO can be applied, and [has](https://ydb.tech/en/docs/reference/ydb-cli/commands/workload/) built-in benchmarks so PGO can be easily trained at least on these benchmarks.

I recompiled YDB with instrumentation PGO and started the benchmark to collect PGO profiles. But for some reason, the benchmark failed. After a deeper investigation, I found an interesting behavior - I was hitting internal YDB timeout deadlines. Since the instrumented binary was running slower, internal deadlines started to interrupt the PGO training phase. I increased [the limit](https://github.com/ydb-platform/ydb/blob/main/ydb/core/load_test/kqp.cpp#L321) (since I had no clue how to fix it properly) and the issue was fixed.

However, since the upstream has no proper fix yet, and you decide to optimize YDB with PGO - just be aware of the issue! Maybe there are the same places in other YDB parts - who knows? By the way, I talked a bit about PGO with YDB developers at TechInternals 2024 - they successfully reproduced my results internally ;)

Another interesting issue was met when I tried to apply LLVM BOLT to YDB. The resulting Linux `perf` profiles were quite big (more than 200 Mib), and `perf2bolt` tool from LLVM BOLT consumed [too much](https://github.com/llvm/llvm-project/issues/61711) RAM during the conversion phase. The solution/hack was to pass an additional `-strict=0` option. However, before getting a response I got an additional 16 Gib RAM from my friend and upgraded my setup from 32 Gib to 48 Gib. Not only Chrome and Java eat all your memory ;)

Lessons learned:

* Having built-in benchmarks is a useful thing!
* Timeouts are not jokes and actually can hurt your PGO trip

### HAProxy

Once I [created](https://github.com/haproxy/haproxy/issues/2047) an issue about PGO in the HAProxy repo, and [Willy Tarreau](https://github.com/wtarreau) (the HAProxy maintainer) said that there would be no effects from PGO on HAProxy. I argued a little bit about PGO, CPU consumption, HAProxy actual bottlenecks, and I was not able to win this dispute without an actual benchmark. So I decided to prove my point of view.

I crafted a benchmark and showed the positive results to Willy, he asked me to improve my benchmark scenario, demonstrate additional information like binary sizes, and perform PGO tests on GCC too (I use Clang as a default C and C++ compiler). After several iterations, Willy [concluded](https://github.com/haproxy/haproxy/issues/2047#issuecomment-1729980766) that PGO shows measurable improvements. It was a huge win for me!

I **really** appreciate such attention to the details and wish to understand as many details as possible about PGO from Willy Tarreu. Even if it required performing multiple tests, I still think it was worth it.

Lessons learned:

* Maintainers know the domain much better than you and can help you with crafting the "right" benchmark
* Disputing about PGO is much easier with the actual benchmark results

### SQLite

SQLite is one of the most famous databases for good reasons. It's easy to use, performant, and well-supported - sounds like a good candidate for PGO optimization, right? I thought in the same way and decided to optimize SQLite with PGO. I quickly googled  "SQLite PGO" and... found a note in the official SQLite documentation! It said (approximately) - "PGO doesn't bring measurable effects on SQLite performance". Challenge accepted.

At first, I used [Clickbench](https://github.com/ClickHouse/ClickBench) (btw, nice OLAP benchmark!) for PGO experiments with SQLite, performed all required test routines, and [reported](https://sqlite.org/forum/forumpost/19870fae957d8c1a) the results back on the SQLite forum. After some discussions and performing another round of tests (since developers asked me to test additionally with a built-in SQLite benchmark instead of ClickBench), a PGO note was [removed](https://www.sqlite.org/docsrc/info/43c1154ec6af8b4dbbde3fe64b6c1887868c3f43cb72b1bfaa5821082cc1099c) from the documentation.

I still wanted a bit more - explicitly mention in the SQLite documentation that PGO works, and PGO should be used by SQLite users if they want to achieve better performance with the database. That's why I created [another](https://sqlite.org/forum/forumpost/7fd3c9a43a) forum topic to add a note about **positive** PGO effects on SQLite. And... didn't get any answer at all. I even sent an email to "drh at sqlite.org" and "ggw at sqlite.org" (found these emails somewhere on the official SQLite website) - also no response. For now, it's the end of the story. If someone has good relationships with SQLite devs - please ask them about PGO documentation for the database. If you are a SQLite developer - you don't need even to ask, you can commit to the documentation directly ;)

Some conclusions: doesn't matter how advanced is your benchmark. At first, you need to use the benchmarks from the project or recommended by the project since maintainers believe in them more compared to any other external bench suite.

* TODO: insert here a meme with VittorioRomeo and C compiler warning

### Lapce and an "interesting" approach to measure performance improvements

I [had](https://github.com/lapce/lapce/discussions/2817) a quite funny conversation about enabling PGO for Lapce. As usual, the maintainer doesn't believe in PGO efficiency in practice (that's one of the reason why I wrote this article). I started to explain that probably we need to perform some benchmarks before concluding that PGO doesn't bring much value for the project. Instead, I [got](https://github.com/lapce/lapce/discussions/2817#discussioncomment-7674615) an interesting answer "If the improvement is not visible without benchmark, is it really an improvement? As far as this goes, it's only different numbers without real impact on users.".

Well, I can imagine the source of such a point of view. If you write the user-facing application, you *can* think about all under-the-hood optimizations as non-important. However, the "visibility" topic is complicated. Some people even notice the latency between different terminals (one of the reason why we still have ongoing optimizations in this area), some people definetely can see battery usage improvements due to more efficient application implementation (especially if such users use CPU-heavy features from an application). That's why by default I suggest to talk about something measurable like benchmarks - it's much easier to work with numbers instead of feelings, you know. Performing huge A/B tests on different focus groups is a nice thing actually but this option is not available for almost all open source projects yet.

### Spamming and LLM

As you see in the "Are we PGO yet?" list, I created a lot of different PGO-related issues in different repositories. My intention was simple: if I see that a project's performance can be improved with PGO - I create an issue about it. Creating an issue or discussion is a natural way to collaborate on GitHub.

In most cases, the issues are left unanswered, in some cases maintainers are interested in PGO, and in some cases they even start PGO evaluation - that's really great! However, in very rare cases, people just call me a spammer and close the issue automatically. I met such behavior in [Shady](https://github.com/Hugobros3/shady/issues/18), [ECL](https://gitlab.com/embeddable-common-lisp/ecl/-/issues/725#note_1678404112), and [HEIR](https://github.com/google/heir/discussions/613#discussioncomment-9602431). Especially funny was the ECL case when I was called a LLM :D I wanted to start my answer something like "As a brain-driven not only language model" but decided to spend my time in a more useful way - play Devil May Cry 3 :D

Yes, I created *hundreds* of issues about PGO since this is the only viable option to start a conversation about PGO with project authors/maintainers, talk with them about the major pain points of PGO, discuss some performance improvement numbers and possible ways to integrate PGO into their projects. The most interesting insights about PGO I got from long and sometimes flamy conversations about PGO. So if some people that I am "just spamming" - well, I don't care *much* about that, not a first day in the Internet. You can always just ignore my issues - it's up to you. I will do the same thing - spread the word about PGO for various projects. For the projects where I see potential benefits from PGO.

When the discussion starts from the first comment in some rude tone, you know, I have far less desire to continue conversation with you on the topic, and I will just ignore you since I have many other things to do on my list. So if you want to get valuable feedback about my motivation, and more details about PGO and similar things - just ask me, I would be happy to chat with you about the topic. If you are not interested in it - just kindly say it in the issue and close it, there is no reason to be harsh in this case. Maybe leaving the issue without an answer will be better than that.

### PGO and GPT/LLM/AI/any other buzzword

Kinda funny answers about PGO I got from Bard/Gemini. This awesome piece of software on the question "Can I use PGO profiles from Clang with Audi A6?" (Volkswagen Group didn't pay me for such questions) quickly responded something like "Yes, sure - go ahead!". Don't know - is it possible to reuse PGO profiles from BMW with Clang or they use GCC-compatible format instead - needs to be investigated deeply! The latest version is a bit better but still gives me non-existing compiler options in the generated command lines - shame on you.

Both ChatGPT 3.5 and Bing (in a precise mode) generated almost sane content about PGO. However, do not expect from them much above the official documentation to the compiler. They still cannot apply PGO properly to any non-trivial application, don't know much about PGO-related intrinsics, and from time to time invent non-existing compiler switches.

As a small conclusion - LLM is not a good PGO friend yet. Maybe with this article and wider PGO adoption in the community, the situation will change. As for now - reading the old but gold documentation is still a better option to start with PGO.

And every time when I hear things like "all we need here is AI", my brain automatically plays [this](https://www.youtube.com/watch?v=D0q0QeQbw9U) video. Don't be a **blind** Ayaya!

### Issue close policy

In several projects I found a strange (IMO) policy about auto-closing issues without activity (like [this](https://github.com/kcl-lang/kcl/issues/647#issuecomment-1972561465) or [this](https://github.com/SerenityOS/serenity/issues/19549#issuecomment-1603415990)). Sometimes it's done via GitHub stale bot, sometimes - manually. I don't understand this strategy. If a project maintainers are not interested in some features - just write about it and close the issue with a commment like "Thanks for the suggestion but we don't care about PGO.". This is completely fine since I see that project maintainers don't care about this opportunity.

However, auto-closing issues is a different thing. If someone from the community will find the issue and decide to work on it, it's not clear - is the issue still relevant for project or not since it's closed. New comers are going to work on **open** issues, not closed. If you are a maintainer and see an issue on that you don't have enough time to work with - just add a "help wanted" label and leave it as open. There is nothing bad about having open issues - just use proper tools to manage them properly ;)

A bit similar situation when a project [closes](https://github.com/root-project/root/issues/15778) (and a [similar](https://github.com/root-project/cling/issues/508#issuecomment-2203623528) one) PGO request due to "lack of action points". I don't understand that either - if you are not against the idea just leave the issue open. That's how open source works - people see open issues and can start to work on them. Even if I cannot right now start working on it (due to my insanely huge PGO backlog with other projects) - someone from the Internet can find it. Just mark the issue like an "Idea" and that's it ;) Another example - Q# [issue](https://github.com/microsoft/qsharp/issues/752#issuecomment-1773194243). The PGO issue was closed with "This does not seem to be relevant to our ongoing work." statement. I tried to find the project roadmap (no success) and kindly asked to explain a bit more about the ongoing priorities - no success either. Q# happens!

### Enemy language

This funny story requires some knowledge of Russian language or knowing how to use a Russian->whatever translator.

One day I found another database - `libmdbx`. I already performed a lot of PGO tests on databases with very positive results so I decided to create a PGO request for this database too. Since the GitHub [repository](https://github.com/erthink/libmdbx) is archived, I quickly found a live [mirror](https://gitflic.ru/project/erthink/libmdbx) on GitFlic (a Russian-based "alternative" to GitHub), and created the corresponding [issue](https://gitflic.ru/project/erthink/libmdbx/issue/14) about PGO.

However, the answer from the developer was a bit unusual - in the first paragraph he kindly asked not to use the "enemy language" (Enlgish, in this case). Well, I confess - most of my PGO issues are copy-pasted with some project-specific tweaks - I was a bit lazy to rewrite all this text in Russian (probably my bad - sorry for that!). Also, on GitHub I saw English text and didn't know how many libmdbx contributors know Russian language - by default I am trying to use English for IT stuff. Additionally, I checked the author's [profile](https://gitflic.ru/user/erthink) and found wonderful note - "Please don’t use my work, if you are associated with Adolf Hitler, Stepan Bandera, George Soros, Michael Hodorkovsky, either support an actions of these felons.". I swear that my work is not associated with these persons. However, if Soros or Hodorkovsky want to support my PGO activity - I'll be happy for that (especially if this support has a form of many Benjamin Franklin's portraits ;). I suppose other two persons from the list cannot support me anymore.

Besides that politic-dependent detail, the answer was very clear and precise with technical details about manually-written compiler instrinsics, some PGO benchmarks for specific cases, etc. I would say, one of the most technically-full answers during my PGO journey.

Conclusion? No conclusion here. I don't like when politics is mixed with technical things, and propose to other people don't do that. Since 2022 I met multiple similar politically-driven requests for [OpenBLAS](https://github.com/OpenMathLib/OpenBLAS/issues/3551), [Conan](https://github.com/conan-io/conan/issues/15134), [gperftools](https://github.com/gperftools/gperftools/issues/1345#issuecomment-1165155423). In gperftools repo I was banned for [proposing](https://github.com/gperftools/gperftools/issues/1346#issuecomment-1165535381) a help to remove the x86-64 support since this architecture also does many evil things :D Yep, I like trolling and I don't care at all about politics when I do my favorite thing - playing with technologies. Unfortunately, the reality and people are a bit more difficult...

### Barriers to creating issues about PGO

During my PGO voyage I found several barriers for talking with projects developers about PGO. At first, several projects don't any kind of SSO with Google, GitHub, etc. I understand that sometimes it requires additional resources for configuration, sometimes maintainers simply don't want to enable such functionality for several reasons. However, it creates additional barriers for contributing (in my case - talking with you via issue/forum topics/etc.). As a result, I created many accounts just for posting an issue - not a big deal but anyway.

Unfortunately, sometimes a new account simply cannot be created. For [Picodata](https://git.picodata.io/picodata/picodata/picodata) my request for a new account simply left unanswered. [RISC OS Open](https://www.riscosopen.org/) simply [disabled](https://www.riscosopen.org/hub/account/signup) new accounts registration due to spammers - I wanted to start a discussion on their forums about enabling PGO for different pieces of the project: the OS itself, infrastructure around it, etc. Instead of a convenient online registration form they propose to send an email with a registration request, wait for approval, etc - not a quick process. I simply decided to skip it for now. If anyone from RISC Open is reading that - please consider to look at PGO (and at least thing about some better way to handle spammers). I just wanna say that having such quirks in the communication can hurt new comers' experience a little bit.

TODO: SerenityOS (and Ladybird) policy, 8th one - "Talk is cheap. Don't waste other people's time by talking about your great ideas if you don't also spend time implementing your ideas. We have no need for "idea guys"". LOL, how do I need to propose an idea?
TODO: Write about missing Discussion in many repos on GitHub - that's why I created many Issues for PGO even if they are not the issues. Try to find a user request for enabling the Discussions by default on GitHub (or create such a request)

### CPU overheating

When the summer 2024 arrived, I noticed that during my most CPU-intensive PGO benchmarks my CPU in PC was overheating. Consequently, throttling quickly arrived. Not a significant performance penaly but anyway it was enough to choose more powerful CPU Cooler. So I quickly bought be quiet! Dark Rock PRO 5 and forgot about the CPU temperature completely. PGO can help you not only to achieve better software performance but also spent some amount of bucks on upgrading your potato PC!

### Other

Here I collected some smaller observations that don't fit into a dedicated section but still worth to be mentioned. [Sometimes](https://issues.chromium.org/issues/325103518#comment8) people claims about PGO efficiency for a project but for some reason don't attach the benchmark reesults. Please don't do that since such a way lacks of transparency and it's much easier to believe in PGO efficiency with some benchmarks numbers. Especially great if your PGO results can be easily reproduced. As an example you can look at my PGO benchmarks for different projects - they consists of test environment, benchmark setup, how the tests were done, performance improvement results and some additional information.

For some projects I met build issues in a bit specific way - the project itself can be built successfully but the built-in benchmarks for some reason fails to build (like [one](https://github.com/dalance/amber/issues/335), [two](https://github.com/tokio-rs/prost/issues/1005)) or [fails](https://github.com/unicode-org/icu4x/issues/5105) to run due to bugs. I guess the reason for that because people don't integrate building (at least building!) the benchmarks to their CI pipelines. It leads to an obvious outcome - at some point, your benchmarks will be broken. The solution is simple - please, at least build your benchmarks as a part of your regular build pipeline. If you don't care much about this code - write a note somewhere in a README file with something like "Benchmarks are written once, we don't maintain them as carefully as we do for the main code. Be ready to fix some bugs". For PGO it's important since benchmarks can be used (in some cases) as a PGO training workload. Having issues with building and running benchmarks can affect negatively the PGO integration path into a project.

In several projects I found that PGO was integrated without benchmarks at all! Even if I like PGO, I don't like such approach since for some projects PGO won't bring improvements. In this case, you simply slowdown the build process without valuable outcome. Even if you are lucky and PGO made some improvements - it would be nice to document them since this makes the discussion about enabling PGO with downstream projects (like package maintainers) much easier because engineers like numbers ;)

By the way, I highly recommend you to look through the PGO-related issues in the public [Google issue tracker](https://issuetracker.google.com/issues?q=PGO). Google through many years gathered a lot of experience with PGO. When the official documentation lacks of many important practical details, they can be found in this tracker discussed in the context of real applications like Fuchsia, Android, Chromium. I found many interesting insights about PGO here (e.g. the first mention of Temporal PGO was found in this tracker). Don't forget to check "Similar issues" during your investigations - quite often they suggest other PGO-relevant topics!

I am almost sure that I forgot mention many other smaller details but who cares? Anyway, I had a lot of fun during all of these interactions so I don't care much about small nuances!

## PGO tips

TODO: Add to the article info about link errors and passing PGO flags to LD too

## FAQ

### Is PGO a silver bullet? :D

TODO: Write a note about reporting problems with PGO to the upstream - how to do it properly? Especially, if we are talking about debuggability of the performance regression
TODO: Add a note to the article about debugging performance regressions with PGO - how you can do it
TODO: What to do with performance regressions after PGO? Rustc question: https://users.rust-lang.org/t/how-to-report-performance-regressions-with-profile-guided-optimization-pgo/98225
TODO: doesn't work for IO-bound things - PGO doesn't help for IO-bound workloads: https://github.com/near/nearcore/issues/10916#issuecomment-2041807052
TODO: doesn't work well for already manually heavily-optimized projects like FFmpeg

### Can I use PGO without LTO?

TODO: add info that ThinLTO is not so slow compared to FatLTO like it was proven in https://github.com/ldc-developers/ldc/issues/2168#issuecomment-313969487

Yes! Link-Time Optimization (LTO) and PGO are completely independent and can be used without each other with almost all compilers. AFAIK, the only exception is MSVC - with this compiler, you cannot enable PGO without LTO.

Usually, LTO is enabled before PGO. Why it happens? Because both LTO and PGO are optimization techniques with an aim to optimize your program. In most cases, enabling LTO is much easier than PGO because enabling one compiler/linker switch with LTO is an easier task than introducing 2-stage build pipelines with PGO. So please - before trying to add PGO to your program try to use LTO at first.

There are some situations, when you may want to avoid using LTO with PGO:

* Weak build machines. LTO (even in ThinLTO mode) consumes a large amount of RAM on your build machines. That means if your build environment is highly memory-constrained - you may want to use PGO without LTO since PGO usually has lighter RAM requirements for your CI. (TODO: add Linux distribution examples here in the build scripts)
* Compiler bugs. Sometimes PGO does not work for some reason with LTO (like [this](https://github.com/rust-lang/rust/issues/115344) and [this](https://github.com/rust-lang/rust/issues/117220) bugs in the Rustc compiler). Even without PGO enabling LTO can bring multiple bugs - e.g. check YugabyteDB LLVM [fix](https://github.com/yugabyte/llvm-project/commit/64d871949eb23145af7b97cb13feaeeeee7ab39a), `resvg` [experience](https://github.com/RazrFalcon/resvg/commit/dc6182f32fec2ba0d1c58ccaf42eee271f0dc633) or Cern's Root [issue](https://github.com/root-project/root/issues/11657) with LTO.

Besides that, you can meet other smaller inconveniences with LTO like [thread contention](https://issues.chromium.org/issues/40942246#comment4) during the build process that also can be mitigated with some efforts.

I have several examples of how LTO improves performance:

| Application | Release | Release + LTO | Link |
|---|---|---|---|
| legba |  |  |  |

Some Linux distributions a few years ago started to integrate LTO as a default compiler flag for building their software. Here are some examples:

* Debian: [link](https://wiki.debian.org/ToolChain/LTO)
* Ubuntu: [link](https://wiki.ubuntu.com/ToolChain/LTO)
* Fedora: [link](https://fedoraproject.org/wiki/LTOByDefault)
* OpenSUSE: [link](https://en.opensuse.org/openSUSE:LTO)

Enabling LTO by default is a huge step that highlights dozens of bugs in your software (especially C and C++). You see it on a Gentoo example - they already collected many LTO-related issues (links to [Gentoo bugzilla](https://bugs.gentoo.org/618550), [gentooLTO repository](https://github.com/InBetweenNames/gentooLTO/issues), [Windows-specific LTO bug in Rustc](https://github.com/rust-lang/rust/pull/113433)). I am sure you can find similar issues in other bug trackers as well.

However, not all distributions consider enabling LTO by default due to the problems discussed above. One of the examples is NixOS (the discussion can be found on [Reddit](https://www.reddit.com/r/NixOS/comments/146wdfk/lto_by_default/)). Hopefully, over time more and more issues with LTO will be resolved and we will see the "LTO by default" policy enabled in more build guidelines.

If you are interested in more details about LTO, I collected several links for you:

* ThinLTO: [Google Research](https://research.google/pubs/pub47584/), [Youtube](https://www.youtube.com/watch?v=9OIEZAj243g)

By the way, ThinLTO works [better](https://discourse.llvm.org/t/thinlto-and-profile-based-module-groups/77120/10) with PGO according to ThinLTO developers due to better module grouping decisions. More information can be found in [these](https://llvm.org/devmtg/2015-04/slides/ThinLTO_EuroLLVM2015.pdf) slides and in [this](https://ieeexplore.ieee.org/document/7863733) publication. GCC's LTO implementation also has [benefits](https://gcc.gnu.org/legacy-ml/gcc-help/2019-05/msg00032.html) from PGO.

TODO:
- add more information about Fat and Thin LTO
- add about LTO state across Linux distributions
- add LTO performance improvement result examples
- add more LTO links to articles and talks

### What about PGO profile compatibility between compilers?

**Awful**. PGO profiles are incompatible between compilers. So if you have PGO profiles generated by GCC, it's impossible to use them in Clang or MSVC. In theory, it's possible to write a converter between profile formats but I don't know such projects (of course you can try writing your own. If you do - please let me know!). However, due to differences between PGO implementations in compilers, it could be [difficult/impossible (check the third answer)](https://discourse.llvm.org/t/profile-guided-optimization-pgo-related-questions-and-suggestions/75232/2?u=zamazan4ik) to implement.

Compatibility is not guaranteed even between versions of the compiler! So if you upgraded from Clang X to Clang Y - probably you need to regenerate your PGO profiles as well. And if you want to support multiple compilers - you need to maintain multiple PGO profile versions (or just generate PGO profiles on-demand for each compiler).

TODO: update this section with information for Clang, GCC, MSVC with links

I tried to find strict profile format definition, forward/backward compatibility guarantees, migration guides, and built-in versioning support but with no success. If you are a compiler engineer and you can fix it somehow - please do it! It would be helpful for the whole community!

Another idea is converting PGO profile information directly to the sources. In theory, we can try to extract all this information about likely/unlikely branches from a profile and insert required [[[likely]]/[[unlikely]]](https://en.cppreference.com/w/cpp/language/attributes/likely) attributes (or corresponding compiler intrinsics like [__builtin_expect](https://gcc.gnu.org/onlinedocs/gcc/Other-Builtins.html#index-_005f_005fbuiltin_005fexpect) in GCC). It will resolve PGO profile compatibility issues. However, there are a lot of open questions in this case:

* Is it possible to codify all information from PGO profiles via compilers attributes/intrinsics?
* Putting all this information directly to the sources can reduce the "signal-to-noise" ratio in the sources. It's important since sources are still mainly edited by humans. Maybe some IDE clever folding functionality will be required in this case.
* How to track multiple PGO profiles for different workloads in the same codebase? Different code branches? :)

And many other [known unknowns](https://www.theuncertaintyproject.org/tools/rumsfeld-matrix) that should be carefully clarified.

### PGO profiles compatibility between OS

Another possible caveat - different mangling rules between OS. I [met](https://github.com/mstorsjo/llvm-mingw/issues/383#issuecomment-1838585818) such an issue at least once and confirmed it with my local tests, the Rustc project also [found](https://discord.com/channels/636084430946959380/1041903397873406012/1177366981368877127) the same issue. For fixing this there is an idea to write a PGO profile converter Linux <-> macOS for changing OS-specific symbol name details. Current status - just an idea, nothing more. If you want to implement it - you are welcome (pls don't forget to open-source it!).

So by default, I recommend regenerating the PGO profile for each operating system. If you cannot do it for some reason (e.g. there is no possibility to collect PGO profiles on some target OS due to various reasons like [lack of](https://github.com/pydantic/pydantic-core/issues/732) corresponding runners in a CI pipeline) - you could try to apply a PGO profile from one OS to PGO-optimized build for another OS but you need to track somehow that this PGO profile has good coverage for the target OS too. Practical [example](https://issues.chromium.org/issues/40281799) of such reusage - Chrome on Android.

### Does PGO/PLO depend on a CPU architecture?

No, it does not.

However, still there are cases when the PGO profile can be affected by CPU-intensive code. Imagine a program with CPU runtime dispatch logic (detects CPU in runtime and based on the detected CPU executes different code branches). In this case, PGO will be affected by the CPU in runtime - because depending on the CPU different code branches will be executed, so different PGO profiles will be generated. Although, this is true not only for CPU but for any other parameter that can trigger executing different code branches in your code.

In general, if your program doesn't have **a lot** of CPU-architecture-specific code - you should be fine with using one workload scenario for all architectures. If it's not the case for you - just prepare N different workloads, one per target CPU architecture.

### Is it possible to use PGO with cross-compilation?

Disclaimer: I don't have enough experience with cross-compiling - I always struggled a lot with installation corresponding cross-compiling toolchains properly.

As far as I understand, there are no reasons why PGO will not work in cross-compiling scenarios. If you already collected PGO profiles from the target platform - it must be completely fine. If you collect PGO profiles during the cross-compilation process - beware of differences in PGO profiles between platforms (which were discussed a bit earlier).

The only limitation I found is a [nuance](https://github.com/yamafaktory/jql/discussions/244#discussioncomment-7665597) in `cross-rs` with subcommand invocation. Nothing too critical but anyway. Other projects sometimes also [disable](https://github.com/LadybirdBrowser/ladybird/commit/3ff7b76502ff2b7492660475a881befe2cda42f9) PGO for cross-compilation scenarios - the actual problem should be investigated.

### PGO and user hints

In many programming languages and compilers, it's possible to insert different hints into the source code which can help a compiler to optimize your application better. Some examples are `[[likely]]`/`[[unlikely]]` attributes from C++, different kinds of `__always_inline`, `no_inline`, `go::noinline` and similar things about inlining, etc. An obvious question that appears in my mind - how does PGO interact with user-provided hints? What if a user hint says that a function should be inlined but according to the PGO profile the function shouldn't be inlined?

The question is complicated. On the one hand, If a PGO-based decision is preferred, then for some users such behavior can be surprising since they explicitly added the hint to tweak the compiler behavior. On the other hand, if the user hint is chosen, we can miss some optimization opportunity since the PGO profile is a data-driven decision and our hint can be just outdated due to various reasons. Maybe try to "merge" PGO and user hint with some weights? Maybe just raise a mismatch warning about a conflict between user hint and the PGO profile? So many options.

Unfortunately, it's a dark area in the compilers. I didn't find **any** compiler that documents such behavior well. There are such questions for [LLVM](https://github.com/llvm/llvm-project/issues/58189), [GCC](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=112806), [GraalVM](https://github.com/oracle/graal/discussions/7990). There is an [answer](https://github.com/golang/go/issues/64460#issuecomment-1834294715) for the Go compiler - PGO should respect the user's hints - but it should be tested in practice. For other compilers (especially proprietary ones) you need to do additional research. And I already made some experiements too ;)

For the following code:

```cpp
[[clang::noinline]]
bool some_top_secret_checker(int var)
{
    if (var == 42) [[unlikely]] return true;
    if (var == 322) [[unlikely]] return true;
    if (var == 1337) [[likely]] return true;

    return false;
}

int main()
{
    int var;
    std::scanf("%d", &var);
    std::printf("%d\n", some_top_secret_checker(var));
    return 0;
}
```

and training workload with 322 values we get the following assembly (compiled with Clang 17: `clang++ -O3 -fprofile-use=likely.profdata -S -masm=intel main_likely.cpp`):

```
	mov	al, 1
	cmp	edi, 322
	jne	.LBB1_1
.LBB1_4:
	ret
.LBB1_1:
	cmp	edi, 42
	je	.LBB1_4
# %bb.2:
	cmp	edi, 1337
	je	.LBB1_4
# %bb.3:
	xor	eax, eax
	jmp	.LBB1_4
```

At least in this case with Clang PGO profile has higher priority than a user-specified attribute - the branch with 322 is moved to the beginning of the function even if it's marked as `unlikely` in the source code. It's not a guarantee of course - it's just an example, and probably in other cases Clang can make different optimization decisions. However, it's better than nothing. Also, I did the same experiment with GCC. For some reasons, GCC [ignores](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=114761) `likely`/`__builtin_expect` attributes and PGO profiles for this code, so we need to prepare something else for the test.

Why do the compilers hesitate to document such behavior? At first, there was no interest in such information before (at least I didn't find it in the public field) :) If no one is interested in it - there is no reason to document it. At second, internally compilers are pretty complicated things, with multiple optimization passes, etc. Tracking interaction between user hints and PGO can be a tricky task that depends on multiple compiler internal details. I am pretty sure that without additional semiresearch/semidebug compiler engineers also don't know the exact behavior in all cases. And even if they know - guaranteeing it can be a limitation since if you guarantee something you need to test it, make regression tests, etc. But we are humans == we are lazy by design so we don't like to do extra things. So actually I don't expect that such things will be documented in the near future.

What about the mentioned above "clever" strategies for handling PGO and user-hints at the same time like warnings, merging, etc.? Clang has the `-Wmisexpect` warning ([reference](https://clang.llvm.org/docs/DiagnosticsReference.html#wmisexpect), detailed [description](https://llvm.org/docs/MisExpect.html), usage [example](https://issues.chromium.org/issues/40694104) in Chromium) - nice quality of life feature. GCC [doesn't have](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=114851#c1) such a thing (yet). Unfortunately, I didn't find anyhting like that in other compilers for different programming languages. Possibly a good improvement point for current PGO implementations.

### Regenerating vs caching PGO profiles

TODO: finish section

Usually, there are two ways to work with PGO profiles:

* Generate PGO profiles again for each build
* Cache PGO profiles and reuse them for builds

Here are some examples of how it's done in some projects:

* Android Open Source Project (AOSP): [repository](https://android.googlesource.com/toolchain/pgo-profiles/+/refs/heads/main)
* file.d: [GitHub file](https://github.com/ozontech/file.d/blob/master/default.pgo)

### Algorithm optimizations always outperform machine optimizations

Several times I seen an interesting thought. TL;DR - "Algorithms always beat optimizers". I met such a viewpoint at least twice ([one](https://github.com/rsyslog/rsyslog/issues/5048#issuecomment-1611665495), [two](https://github.com/KhronosGroup/glslang/issues/3400#issuecomment-1817277433)). I agree and disagree with them at the same time.

I agree that it's better to spend time on algorithm-based optimizations instead of nitpicking `always_inline` attributes here and there because, probably, from algorithmic optimizations will be more performance outcomes (however, it's not always true. it depends on a case, you know it). Also, doing algorithmic optimizations is a funnier thing to do, isn't it? We are all engineers, after all, and we like to spend/waste our time on fixing our algos from O(N^2) to O(NlogN) (unfortunately many of us forget about the hidden constant but that's another story).

However, I disagree that "algorithms beat optimizers". Because they work at the same time! Your super-duper-blazing-fast algorithm from your usually pretty high-level language is then compiled and optimized by a compiler. During the compilation process, a lot of things happen: inlining, loop rolling/unrolling, [peephole optimizations](https://en.wikipedia.org/wiki/Peephole_optimization) and other things. And you don't complain about them, right? PGO is exactly the same thing as other compiler optimizations - it helps to tweak compiler optimizations for specific target workloads, consequently helping to optimize better your algorithms.

So PGO doesn't replace algorithmic stuff. Let your compiler do "dirty" things when you spend your time on more high-level topics like algorithms.

### Do I need to integrate LTO, PGO, PLO or any other optimization without benchmarks?

TL;DR: No, you should not.

Any of these optimizations can (and with a high chance will) improve application performance. However, in every case, there are billion of reasons why it can be worse or better: application-specific bugs triggered by LTO, too small performance wins from PGO, too huge build times with PGO and PLO enabled, etc. Every optimization has its pros and cons. Enabling all these optimizations blindly for the whole software on your favorite server is not the smartest idea.

I hope one day the ecosystem will be ready, and using PGO for release builds will be as easy and common as using `-O3` now is. Now it's not the case, so please - in each case compare the benefits from enabling PGO with the costs for enabling PGO for a project. If the expected costs are bigger - just do nothing (and wait for a bit more for a better PGO future).

### How does PGO and/or PLO change a binary size?

TODO: PGO helps with optimizing binary size since we can inline less for actually cold paths of our programs (and it can help with performance as well since our program will be smaller and more friendly for CPU I-cache). Write that binary can become larger (due to more aggressive inlining) or smaller (if inlining is not required in some places) - it depends on the software kind and actual profiles.

TODO: Do I need this chapter since this topic was discussed above with actual numbers?

### PGO, PLO and inline assembly

TODO: add here info about other PLO tools

Compilers are modest from the optimizations perspective when they see inline assembly. So don't expect PGO optimizations **inside** the inline assembly block. Around the block, all should work the same as for the usual code.

However, the situation with BOLT is different. Since BOLT works on already-compiled binaries, it doesn't care about your inline assembler and can perform optimizations over it. However the inline assembly can be written in a bunch of strange ways: with `rodata` mixed with the text, or without CFIs (breaking ABI unwinding requirements). BOLT can handle that and has safeguards skipping over functions it doesn't fully understand, but in the worst case, it's possible to skip such functions manually using `-skip-funcs` BOLT parameter (LLVM Discord [discussion](https://discord.com/channels/636084430946959380/930647188944613406/1183110709865881701)).

More about handling inline assembly in LLVM you can find in the "2021 LLVM Dev Mtg “Handling inline assembly in Clang and LLVM" talk ([YouTube](https://www.youtube.com/watch?v=MeB7Dp3G2UE)).

### How does PGO/PLO help with binary size optimization?

Sometimes you want to optimize not for speed but for size. In most cases, such requests I heard from the embedded world. For PGO I didn't find good investigations in this area. The only finding is [this](https://issues.fuchsia.dev/issues/42159696) issue in the Fuchsia issue tracker with some measurements. However, the dataset is small - we need more evaluations in this area. If you have something - please share it!

BOLT [cannot](https://discord.com/channels/636084430946959380/930647188944613406/1183491352382689432) reduce the binary size. There was some ongoing work on it but the original author [lost interest](https://discord.com/channels/636084430946959380/930647188944613406/1183493533894725663) in the project.

By the way, there is a project about optimizing binary size on the linker step from ByteDance with some promising results: [Slides](https://llvm.org/devmtg/2022-11/slides/TechTalk11-LinkerCodeSizeOptimization.pdf), [Paper](https://arxiv.org/abs/2210.07311v1). However, I don't know the current status of this work.

### Does PGO/PLO work with stripped binaries?

Instrumentation PGO *should* work with stripped binaries. So after the instrumentation phase, it's ok to strip the binary. Sampling PGO **does not work** with stripping. Debug symbols or at least some other samples-to-source mapping is required. Clang [says](https://clang.llvm.org/docs/UsersManual.html#using-sampling-profilers) about it in the SPGO documentation via enabling `-gline-tables-only` or `-g` option. GCC [has](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-fauto-profile) the same note in the `-fauto-profile` documentation. LLVM BOLT also [requires](https://github.com/llvm/llvm-project/blob/main/bolt/README.md#input-binary-requirements) non-stripped binaries.

### PGO **decreases** performance - what can I do?

TODO: write about reporting such problems to upstream - https://users.rust-lang.org/t/how-to-report-performance-regressions-with-profile-guided-optimization-pgo/98225
TODO: write about how can you try to understand the root cause of the problem
TODO: write about conflicting workloads and you can avoid it
TODO: discuss topic about compiling multiple versions of the same library with optimization to different workloads in the same binary

Imagine the case, when you applied PGO to an application, ran benchmarks, and oh no - performance is **decreased**! Before starting raging with thoughts like "PGO is a lie" or smth like that, let's try to figure out what could go wrong here. There are several reasons why PGO can decrease performance:

* Your training workload is not representative enough.
* There conflicting workloads in your training scenario.
* A compiler bug.

If after all previous steps, you believe that it's a compiler bug... Well, in this case, if you want a fix you need to report the problem to the compiler developers (of course if you are not a compiler engineer and can try to fix the compiler locally (doesn't work with proprietary compilers, though)). Debugging such problems could be a tough job for compiler developers, so be ready to prepare a Minimal Running Example (MRE) with your problem. As an example of non-trivial PGO performance regression debug, you can take a look at [this](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=90364) one in GCC. Here we can check how tough it could be to debug PGO-related problem.

Narrowing the place where the performance is decreased is another topic to discuss. Here you will need to profile your application before and after PGO optimization, and then compare them to find a diff. Tools like [Differential Flame Graphs](https://www.brendangregg.com/blog/2014-11-09/differential-flame-graphs.html) can help here. If you prefer to use other profiling tools - please check the corresponding documentation for them.

### How long should I collect PGO profiles?

TODO: Add to the article a comment from RustFest Zurich about PGO profile collection time, possible strategies for reduction it and hot/cold ratio for different PGO traing phase durations
TODO: Write about a stupid idea about waiting for the loooong benchmarks during the training phase (ClickHouse and faer-rs cases). Add a warning for using benchmarks as a PGO training workload. PGO instrumentation phase for faer-rs was finished in 40 hours (!!!)

Actually, compiler doesn't care about how long you collect your PGO profile. The only thing that the compiler cares is how much of your code is *covered* in your PGO profile. If ~100% percent of your code is executed during the 100ms of execution and this execution completely represents your target workload - there is no need to spend more time with PGO profile collection. In many real scenarios, even if your training workload takes **hours** to complete, you spend most of the time in the same places of your code, so there is no critical need to wait for the end. As an example, you can check my [experiment](https://github.com/llvm/llvm-project/issues/63486#issuecomment-1607953028) with LLD and ClickHouse. However, if your workload takes a long period of time to complete, and during the execution it touches different places of code at different time points, and you want to optimize your application as much as possible - in ideal world, you need to collect the PGO profile from the whole training script. If some places are [not covered](https://issues.chromium.org/issues/40244486) - they will not be optimized. So statements like "You don't need to large PGO profiles for optimizing your software" ([click](https://github.com/llvm/llvm-project/issues/61711#issuecomment-1497184801)) are only partially true. In practice, the answer is, again - "It depends".

One possible caveat that you can find with large profiles is that PGO-related tooling is not optimized for such cases. It could crash or just [get OOMed](https://github.com/llvm/llvm-project/issues/61711). In this case, you can try to do a trick. Instead of collecting one big profile and processing it via a tool, collect N smaller profiles, process them with a tool, and then try to merge processed PGO profiles into one once again. it works since during the conversion phase the tool can perform some heavy and not-so-memory-optimized operations but during the merge phase performed routines are quite simple - "just" merge counters from multiple files into a big one.

### PGO/PLO makes my build non-reproducible! What can I do?

Reproducible builds are a hot topic nowadays for various reasons: security, build cross-validation, etc. More about it you can read [here](https://reproducible-builds.org/). From the PGO perspective we have a problem here since PGO introduces another source of non-determinism to your build that consequently sacrifices your reproducibility efforts. Since some distributions like Rocky Linux care a lot about reproducibility (I had a great conversation about it with a Rocky Linux maintainer at FOSDEM 2024), expanding PGO usage across software can become a problem.

Fortunately, the reproducibility issue with PGO can be mitigated. The most obvious way - collect PGO profiles once, commit them to a VCS, and use these profiles for every build. As an example check how the Android project [stores](https://cs.android.com/android/platform/superproject/main/+/main:toolchain/pgo-profiles/sampling/) PGO profiles. Since your PGO profile is static, you will get the same binary each time. Also, it's possible to generate PGO workloads based on some random inputs - just be sure that you use a fixed seed. E.g. this way was [used](https://github.com/void-linux/void-packages/issues/29526#issuecomment-872803985) for Foot's PGO optimization in Void Linux and [proposed](https://issues.chromium.org/issues/41438361#comment10) in Chromium. Easy-peasy! However, when you commit a profile to a repo, and then users try to use this profile during the PGO build, they will ask you a question - "How did you collect this profile? What was the training workload?". They care because if the training workload differs a lot from their target workload - they simply cannot use your PGO profile! I [met](https://github.com/ozontech/file.d/issues/468#issuecomment-1703866524) this problem with the `file.d` project: they committed a profile, and there is no way to understand from which workload the profile was collected. How can you fix it? At the very least, you can describe somehow your training workload. Even a small piece of text like "For File.d the PGO training workload was reading logs from a file, transforming them with regular expressions and sending them to a ClickHouse cluster" (just an example, not an actual description). Providing training workload Bash scripts (or whatever similar) improves the situation even more since in this case a user can run your scripts in their environment and recreate the PGO profile. Or tweak these scripts a bit and adapt to their specific workloads. As a rule of thumb: if you don't understand how a PGO profile was collected - just regenerate it. And please do not forget one more thing - if your applications evolve and/or your target workload is changed, you need to regenerate your PGO profile and commit it to the repository once again. If you don't do it, your PGO optimization becomes less and less efficient over time since less and less actual source code is covered by the saved PGO profile. In advanced PGO pipelines the profiles efficiency is [measured](https://issuetracker.google.com/issues/301994149) on a regular basis.

One step beyond this idea, we come to an interesting topic - making a PGO profile itself reproducible, not a PGO-optimized build. Here things quickly become more complicated. At first, if your instrumented application has some non-deterministic logic inside (like time-based things or relies on any other external non-reproducible source like hardware RNG), different code paths may be executed -> you get different PGO profiles. These things usually can be fixed manually in your software somehow (e.g. mocking all sources of non-determinism or reworking the source code). Another thing to consider is how your compiler creates PGO profiles. Some compiler like Clang by default uses [non-atomic counter updates](https://clang.llvm.org/docs/UsersManual.html#cmdoption-fprofile-update). So if your application is multi-threaded, with a huge chance PGO profile will differ. Fortunately, this behavior can be changed via the `-fprofile-update` compiler switch. As a downside, instrumentation overhead will increase. The overhead shouldn't be too big in practice but anyway. If you use another compiler - please check the corresponding documentation. According to my experiments, usually Instrumentation PGO produces reproducible PGO profiles if you have deterministic logic inside your application, the same training input, etc. However, this is not always the case. E.g. let's check the reproducibility [issue](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=93398) for the PGO-optimized GCC build. According to the issue, things like Address Space Layout Randomization ([ASLR](https://en.wikipedia.org/wiki/Address_space_layout_randomization)) and "randomness" in places like hash-tables can introduce problems. Generally speaking, we need to perform more tests across different compilers (including proprietary) and software to understand better the common practical problems with PGO in reproducible build pipelines.

All information above is written about instrumentation PGO. What about sampling PGO? Well, here the answer is simple - there is no working way in practice to create a reproducible sampling-based PGO profile. Sampling PGO, by its nature, samples your application with some frequency. In practice, it's almost impossible to sample your application **exactly** at the same execution points so the collected stacks will be just a bit different. Yeah, in theory, you can implement some clever post-processing equalizer tool to reduce the influence of this "noise" or implement manually some deterministic sample approach. However, I never heard about such things.

### How should I dump PGO profiles?

By default, all major compilers save instrumentation PGO profiles at the exit of your program. You may ask - what does it mean, "at exit"? LLVM [means](https://github.com/llvm/llvm-project/blob/main/compiler-rt/lib/profile/InstrProfilingFile.c#L1216 ) an [atexit](https://en.cppreference.com/w/cpp/utility/program/atexit) handler. I am pretty sure that other compilers have the same behavior (but I didn't check it). From such a definition we can get a helpful inside - if our application crashes or is killed, PGO profiles will not be saved.

Here we can start asking about possibilities to tweak PGO profile dumping behavior that we can use in different scenarios. What if we want to dump PGO profiles with some (tweakable by us in runtime) frequency? Or trigger PGO dumping via a signal or even an HTTP handler? Compilers have something for that. LLVM can [suggest](https://github.com/llvm/llvm-project/blob/main/compiler-rt/lib/profile/InstrProfiling.h) you to use several `compiler-rt` intrinsics - they are pretty useful in practice. E.g. YugabyteDB developers [implemented](https://github.com/yugabyte/yugabyte-db/blob/master/src/yb/common/llvm_profile_dumper.cc) a dedicated thread that dumps PGO profiles periodically. During my PGO benchmarks then an application didn't exit normally, I many times tweaked an application and inserted PGO profiles dump capability with `__llvm_dump_profile()` somewhere near the program exit flow or just added a corresponding signal handler with PGO profile dump functionality - it saved me a lot of time! With LLVM it's even possible to write a PGO profile to a memory if you don't have an RW access to a filesystem or simply don't have a filesystem.

What about other compilers? Go compiler has similar functionality via `pprof` infrastructure. GCC, unfortunately, has [quite limited](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=112829) dumping capabilities compared to Clang. MSVC is a bit better but also cannot save a PGO profile to a memory. What about other compilers? I don't know, you need to check the corresponding documentation... Of course, I am joking here - I am almost sure that it will not documented in your case so you need to ask your compiler developers about such functionality ;)

### Can I use PGO/PLO for library optimization?

TODO: finish the chapter

Yes, you can! PGO and PLO can be applied to libraries as well. There are no big differences between a library and an application - here and there it's simply a code. However, there are *some* differences in use cases that can affect PGO implementation for them.

### Can I use PGO/PLO for applications with UI?

Interesting question was [raised](https://github.com/slint-ui/slint/issues/3909#issuecomment-1816054164) by the Slint project - the difficulty of optimizing UI applications with PGO. You want to optimize rendering-related code too. To do that you need to somehow execute rendering code during the PGO training phase. Scripting such things is a bit more difficult task compared to simple execution of several CLI commands.

Here you can use different strategies. You can try to automate UI scenarios during the PGO trainig phase via tools like Selenium or any other UI test framework. You can collect PGO profiles from manual tests where a human can be involved as a part of your testing pipeline. You can try to collect rendering-related PGO profiles directly from production where users interact with your application and do this job for you for free (they even pay for your applications sometimes!) - in this scenario, probably, Sampling PGO should be considered at first. Also, I can suggest you to check Chromium project experience - maybe they have some established practices in this field.

### Is it difficult to extend PGO-related functionality?

TODO: Add to the article information about PGO profile file formats. Discuss at least LLVM-compatible format. Sampling file format is described here - https://clang.llvm.org/docs/UsersManual.html#sample-profile-formats . But not instrumentation formats - is it possible to find it? Is it required since we have documented sampling format?
TODO: Write about multiple instrumentation and sampling tooling and their compatibility with PGO infrastructure: https://thume.ca/2023/12/02/tracing-methods/

### PGO and constant-time calculations

At TechInternals 2024 after the [talk](https://internals.tech/2024/abstracts/9746) I was asked by [Ignat Korchagin](https://twitter.com/ignatkn) about the PGO influence on constant-time calculations. Constant time computation is an important topic in very specific domains like cryptography. If you never heard about that I can recommend to start from the following points: a beginner-friendly [intro](https://www.chosenplaintext.ca/articles/beginners-guide-constant-time-cryptography.html), the BearSSL [article](https://www.bearssl.org/constanttime.html), more advanced things from the "Robust Constant-Time Cryptography" [talk](https://www.youtube.com/watch?v=OZEM_ZAPi5M); if you want to check something more practical - check the [code](https://github.com/RustCrypto/crypto-bigint?tab=readme-ov-file#security-notes). The topic is not that easy and has a lot of hidden "gems".

There are different strategies about how the constant time computation can be implemented. Some of them are more robust, some them less. Good news that all known to me ways to implement it "properly" (quotes here because I am not an expert in this field) shouldn't be affected by PGO. If you have some `sleep`-based logic with some jittering - PGO shouldn't break the things here. "Useful" part of the computation probably will be optimized and become more CPU-efficient but timers will remain in your code. If you use more sophisticated techniques like using proper assembly instructions per architecture, etc. (the rabbit hole here is too deep to discuss in depth - I will leave it for crypto (not currency!) professionals) - highly-likely it also won't be broken. In this case, anyway, you need somehow to check the generated assembly - with or without PGO doesn't matter since even the compiler update can ruin all your efforts via [too clever](https://pqshield.com/pqshield-plugs-timing-leaks-in-kyber-ml-kem-to-improve-pqc-implementation-maturity/) optimizations. If you use inline assembly for that - good news, since all known to me modern compilers are pretty conservative about inline assembly and won't touch it. However, this trick can be broken by some PLO tools like BOLT since it disassembles your program and can try to be too intelligent and optimize-out your constant-related tricks. More details see about that see in "PGO, PLO and inline assembly" section. As a small conclusion - it should be pretty safe to use at least PGO in this domain.

By the way, Ignat at Cloudflare does pretty funny things with the Linux kernel - you can check his [blog](https://blog.cloudflare.com/author/ignat) (more precisely - part of the Cloudflare engineering materials but I don't care), you could find interesting things. Unfortunately, without PGO mentioning (at least yet) :)

## Related projects

TODO: write about improved PGO workflow: https://github.com/llvm/llvm-project/issues/58422

Here I want to show you some PGO-related ongoing research or similar ideas that possibly you find at least interesting.

### Application Specific Operating Systems (ASOS)

TODO: add info about ASOS
TODO: add a diagram about how our applications are deployed on OS

Before we talked about optimizing with LTO/PGO/PLO our applications. But in 99.99(9)% cases, we run software on operating systems. But what if we try to PGO optimize an operating system too?

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
  - Also highly recommend to read [this](https://groups.google.com/g/clang-built-linux/c/85eKdM9_Jeg/) thread about enabling PGO for the Linux kernel with Clang. A lot of interesting insights!
* Windows: 5-20% improvement according to the [presentation](https://lpc.events/event/7/contributions/771/attachments/630/1193/Exploring_Profile_Guided_Optimization_of_the_Linux_Kernel.pdf)

Unfortunately, I have no PGO results for other operating systems like macOS (almost impossible to get insights from Apple, you know) or *BSD (e.g. see [this](https://www.reddit.com/r/freebsd/comments/150codp/profileguided_optimization_pgo_on_freebsd_kernel/) Reddit post). If you can perform tests for other operating systems or already know some PGO results - please let me know!

TODO: Enable PGO, LTO for Fuchsia: https://issues.fuchsia.dev/issues/335919004 + https://issues.fuchsia.dev/issues/331995728

Operating systems can be optimized with PGO in the same as we do for usual applications: compile with instrumentation (if we choose instrumentation PGO), run target workload on the operating system, collect PGO profiles, and recompile the OS once again.

More information can be found in the original paper [here](http://scis.scichina.com/en/2018/092102.pdf).

This idea can go even further! What about building Application Specific Interpreters? The idea sounds reasonable since different applications can have different execution profiles in their interpreters. We can try to optimize interpreters per application, why not? I didn't find papers about this idea, so probably there could be room for future investigations. I already have [some](https://www.reddit.com/r/lua/comments/151dtyu/comment/jsedtcs/?utm_source=share&utm_medium=web2x&context=3) preliminary results about this idea for the Lua interpreter. What about Application Specific Virtual Machines? If you know similar works - please let me know!

### Machine learning-based compilers

Today machine-learning (ML) things are popular, and they are popular for a reason - in many cases, they work great! So it's natural to have an idea about using ML somehow in the compilers.

One of the ideas is MLGO: a Machine Learning Guided Compiler Optimizations Framework ([paper](https://arxiv.org/pdf/2101.04808.pdf), [article](https://research.google/blog/mlgo-a-machine-learning-framework-for-compiler-optimization/), [GitHub](https://github.com/google/ml-compiler-opt), some [benchmarks](https://chromium-review.googlesource.com/c/chromium/src/+/3829793) for Chrome). The idea is simple - try to use machine-learning-driven decisions in some optimizations in the compiler. Every modern compiler already has some kind of "implicit performance model" about the code: a bunch of heuristics for splitting hot/cold code, multiple semi-hidden inline thresholds (like [this](https://github.com/llvm/llvm-project/blob/main/llvm/lib/Analysis/InlineCost.cpp#L82) from LLVM), etc. All these numbers have appeared in compiler through years of compiler optimizations - but this doesn't make all these hardcoded constants the best possible values in for every program! MLGO project tries to replace such hardcoded places in LLVM with something more flexible and twekable - with a ML model. E.g. MLGO uses a Tensorflow-based neural network to perform better inlining decisions (see paper for more details). The work is still in the early stages but looks promising. LLVM is not alone here - Intel® oneAPI DPC++/C++ Compiler has similar `-fprofile-ml-use` [option](https://www.intel.com/content/www/us/en/docs/dpcpp-cpp-compiler/developer-guide-reference/2023-2/fprofile-ml-use.html).

Another idea - try to use ML for [searching](https://github.com/google/heir/discussions/613#discussioncomment-9606880) over compiler options to find the best compiler switches suite for your application. We can imagine that our compiler is a black box, and one of the inputs for this magic box is a compiler options set. Let's try to throw into the compiler different compiler switches for the same source files, compile a program and measure the resuling performance. Compilers have an insane amount of optimizations inside (check [this](https://llvm.org/docs/Passes.html) for LLVM), different optimizations have different efficiency (from the impact on the resulting application performance perspective) for different applications. Even an order in which optimizations are applied can influence in the optimization efficiency. That's why the idea works - there are many compiler-oriented papers with similar ideas.

Even if right now the options above look quite unstable (they are) I still think we will get more and more ML-driven tools around us, and compilers are not something special here. PGO and ML-based compilers have a common thing - they both need some kind of runtime profile of the application to perform better optimization decisions. I can expect that in the future we will see some hybrids between PGO and ML approaches. There are semi-academic projects like [CompilerGym](https://github.com/facebookresearch/CompilerGym) with an aim to help with implementing and testing ML-based approaches for compiler problems. Already [there are](https://llvm.org/devmtg/2024-03/#talk4) some talks from LLVM dev meetings about expanding ML usage.

### OCOLOS: Online COde Layout OptimizationS

PGO and PLO still have one limitation - there is a lag between profile collection and applying optimizations based on this profile. Usually, we need to collect a profile, transfer it to a build machine, enrich the profile with symbols, recompile our application with the profile, and deploy new build to a production environment... sounds pretty tough (especially if it's not CI-automated).

What if we have a tool, that rebuilds our native application on the fly based on the current runtime? Just like JIT but for our usual AoT application. It's exactly what OCOLOS ([paper](https://people.ucsc.edu/~hlitz/papers/ocolos.pdf)) does: collect profiles directly on the production environment, and just in place rebuild your software in a hot-reload manner. Sounds pretty interesting but do not forget - it's just a paper! Every paper is far away from the production-ready quality. So if you think about deploying this tool to something critical - be ready for bugs and other academic stuff.

### PGO for algorithms and data structures

What if we expand the idea of tracking the runtime profile for an application to its other parts? With more advanced profiling, we can try to gather the information about most common access patterns for containers inside the application, the most common memory allocation patterns, and based on this information we can give at least recommendations to developers about the most efficient data structures and memory allocators in certain situations. Right now similar optimizations can be done only manually. However, at least for me expanding PGO in this direction sounds completely sane.

I understand that implementing such an approach on the production-grade quality level will not be an easy task due to various problems - but we can dream at least. Similar works are already available in some ecosystems - check [Chameleon](https://dl.acm.org/doi/10.1145/1543135.1542522) project for Java. We can get something similar for other ecosystems as well.

### PGO for memory allocations

What if we try to optimize memory allocation patterns based on runtime information? Sounds like a PGO for memory allocators! Exactly this thing is described in the "HALO: Post-Link Heap-Layout Optimisation" [paper](https://dl.acm.org/doi/pdf/10.1145/3368826.3377914). I didn't try it yet in practice since it's not implemented in any major compiler. However, I hope that such technologies one day will be implemented - I had exactly the same ideas about applying PGO for allocators for many years but was lazy enough to not share them with the community - sorry for that!

## Awesome PGO people

If you decide to integrate PGO, at some point you will meet some problems, a corner case or find a nice way to improve PGO for your specific workload. So here I want to share the PGO experts list (not complete!). I believe these people could help you in your PGO journey:

* Jakub Beranek aka [Kobzol](https://github.com/Kobzol) - main PGO wizard in the Rust community.
* Maxim Panchenko aka [maksfb](https://github.com/maksfb) - BOLT developer
* Amir Aupov aka [aaupov](https://github.com/aaupov) - BOLT developer
* [ptr1337](https://github.com/ptr1337) (excuse me - I don't know your IRL name) - CachyOS founder and developer
* Xinliang Li aka [davidxl](https://discourse.llvm.org/u/davidxl/summary) - [answered](https://discourse.llvm.org/t/profile-guided-optimization-pgo-related-questions-and-suggestions/75232) **a lot** of my questions regarding the PGO state in LLVM
* Andrew Pinski - an active GCC developer, [gave](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=112717) me many answers about PGO details in GCC.
* Michael Pratt aka [prattmic](https://github.com/prattmic) - a PGO expert in Go. I guess in almost any material about PGO in Go you can find his name.

Additionally, I recommend you [join](https://discord.gg/xS7Z362) the LLVM Discord server. It has many kind and clever people who can help you with your questions regarding LTO, PGO, PLO and other compiler-like fancy stuff. There is a dedicated channel for BOLT too.

Another awesome place to ask questions - compiler communities. Here I can recommend joining LLVM Discourse [forum](https://discourse.llvm.org/).

One more hint to find PGO-related persons in project. If you see if in a project there is already PGO support in some way, `git blame` can help you to find a right person to ask PGO-related questions. Obvious tool but helpful enough to be mentioned here.

## Future plans and ideas

I have several ideas about PGO and PLO's future improvements across the industry. Maybe something from my list will be so interesting to someone, that they eventually will be implemented.

### Explore PGO for gamedev domain

For now, PGO is rarely mentioned with games. It's a bit sad to see since performance (even if we are talking only about CPU part of it) is still important for games: better battery life (smartphones and consoles like Steam Deck), lowering CPU system requirements, higher FPS (where the CPU part is a bottleneck), increased game developer productivity during the game development process - I can imagine many things! The only known to me case is the talk from GDC 2023 "Squeezing out the last 5% of performance: AGDE, profile-guided optimization, and automation" ([Youtube](https://www.youtube.com/watch?v=CNbpFTyHOe8)) - it mentions [this](https://developer.android.com/games/agde/pgo-overview) PGO guide in the Android Game Development Extension (AGDE) documentation. It's great to see PGO mentions in such places! Fun fact: even such proficient in PGO companies like Google sometimes [have](https://issuetracker.google.com/issues/315487999) mistakes in their PGO-related documentation!

One of the interesting areas here - game engine optimization with PGO.

Unreal Engine (UE) supports building with PGO since 4.27 ([release notes](https://docs.unrealengine.com/4.27/en-US/WhatsNew/Builds/ReleaseNotes/4_27/)). According to the documentation, PGO allows achieving +10% on some CPU-heavy scenarios with UE.

Unity is an interesting beast here. Unity has a long-standing process about [Burst](https://docs.unity3d.com/Packages/com.unity.burst@0.2/manual/index.html) - a compiler for translating from IL/.NET bytecode to highly optimized native code using LLVM. Burst aims to be a compiler for "high-performance C# subset" (or something like that). Unfortunately, this high-performance toy [doesn't support](https://forum.unity.com/threads/profile-guided-optimisation-for-burst.931419/) PGO - Burst developers don't think that PGO will work well with Burst. From my perspective - all the mentioned problems from the forum post above are solvable in practice, and outcomes from PGO are far better than problems. Anyway, it's only my opinion, and I have no power to forcefully change Unity developers' minds (Illithid Powers are not allowed here). If we are talking about more "classic" Unity, we have [IL2CPP](https://docs.unity3d.com/Manual/IL2CPP.html) - a tool for converting C# code into C++, and then compiling C++ code with usual C++ compilers. Here there is a possibility to apply PGO after IL2CPP: compile your game into C++ code, then run PGO for C++ code - C++ compilers have one of the best PGO implementations nowadays - it should work at least in theory. In practice - I never heard about using such an optimization pipeline for real games.

In other game engines, the situation is even worse. For [Godot](https://godotengine.org/) we only have [this](https://github.com/godotengine/godot-proposals/issues/2610) proposal with some tests. For [Bevy](https://bevyengine.org/) I did [some benchmarks](https://github.com/bevyengine/bevy/issues/4586#issuecomment-1674097867). For other engines, we only have requests about PGO: [Fyrox](https://github.com/FyroxEngine/Fyrox/issues/498), [Cocos](https://github.com/cocos/cocos-engine/issues/16449), [Defold](https://github.com/defold/defold/issues/8193), [Dagor Engine](https://github.com/GaijinEntertainment/DagorEngine/issues/1), [Flax Engine](https://github.com/FlaxEngine/FlaxEngine/issues/1833), [Bitty](https://github.com/paladin-t/bitty/issues/38), [Hazel](https://github.com/TheCherno/Hazel/issues/645), [Open 3D Engine](https://github.com/o3de/o3de/discussions/16883). I am sure that I missed a lot of other engines - it will be nice to PGO results for them too.

Regarding using PGO with games based on the engines above - I don't have enough information. I asked the question multiple times on different forums (Unreal Engine forum [post](https://forums.unrealengine.com/t/profile-guided-optimization-pgo-results-with-unreal-engine/1253240), Unity forum [post](https://forum.unity.com/threads/add-lto-pgo-and-plo-recommendations-to-il2cpp-documentation.1525315/)) - no response. Only one person from a local Russian-speaking Telegram chat about UE said to me that they use PGO as a default optimization for their UE games. The PGO profiles are collected via crafted local test workloads (usually - the most difficult game scenes) with [Gauntlet](https://docs.unrealengine.com/4.27/en-US/TestingAndOptimization/Automation/Gauntlet/). The performance improvement is like 6-8% of CPU.

Another interesting area where PGO can help is driver optimization, especially video drivers. Modern video drivers - huge applications with a lot of branches inside. Even more - in drivers there are things like shader compilers. As we [see](https://github.com/gfx-rs/wgpu/discussions/4692), PGO helps in these cases in practice. However, more testing in more cases is needed. I hope one day shader compilers will be optimized too - I have too many lags due to the shader compilation process during the TOTK gameplay with Yuzu on my Steam Deck!

### Explore PGO for the embedded domain

PGO for embedded cases can be an interesting option since PGO can help with achieving better CPU efficiency (better battery life, lower requirements for CPU), and binary size optimization (because we can perform fewer optimizations for the "cold" parts of an application).

One of the interesting issues that arises quickly with the embedded domain is PGO profile collection. In all major compilers by default the PGO profile is saved somewhere in a filesystem. But with the embedded case you are often limited from the disk perspective (or even you don't have write permissions to a disk). What can we do here?

Usual option is to try mounting something like `tmpfs` and then save the PGO profile to this in-memory disk. If even this option is not available for you - it's still possible to dump the PGO profile directly to a memory, and then export the profile via any interface like TCP, HTTP, MQTT, etc. Profile dumping runtime intrinsics are usually undocumented so you need to check sources for your favorite compiler (and create an issue about improving documentation). These intrinsics are partially discussed in the article somewhere above.  

Right now I don't have enough data about PGO usage in the embedded domain. If you know something about the topic and want to share your thoughts - go ahead and reach out to me!

### Explore PGO for mobile domain

I didn't hear much about using PGO for mobile devices. Maybe it's because on mobile platforms nowadays we have not-so-PGO-friendly stack (mostly Android/Java and Apple/Swift) - both of these programming languages definetely are not huge PGO users. However, having better CPU performance is still important: better UI responsiveness, better battery life - nice things to have. Also, don't forget about underlying components on Android/iOS - they are written in native technologies and definitely can be optimized with PGO!

Android has two pages about using PGO with it: [one](https://source.android.com/docs/core/perf/pgo) for instrumentation PGO, [another one](https://source.android.com/docs/core/perf/autofdo) for sampling PGO. Instrumentation PGO is deprecated for usage with Android 14 and newer, so if you target the newest Android versions - probably there are no big reasons to invest your time into instrumentation PGO. Why did they deprecate it? Google invested and invests a lot of resources into polishing sampling PGO, they use mainly sampling PGO on their backends - they simply [don't want](https://issuetracker.google.com/issues/315464624#comment3) to maintain redundant (from their point of view) technology. Fair enough, I would say. Fun fact: even Google [had](https://issuetracker.google.com/issues/315464624) some issues regarding Frontend PGO vs IR PGO recommendations in their documentation - no one is perfect ;) What about iOS and other mobile operating systems - do they use PGO for building kernels and related things? I don't know - I wasn't able to find information about that in public sources.

However, even if the Android projects documents its PGO routines that doesn't mean that you can use it easily! At first, I tried to reproduce the [proposed](https://android.googlesource.com/platform/system/extras/+/refs/heads/main/simpleperf/doc/collect_etm_data_for_autofdo.md) PGO scenario step by step with my Pixel 8 with stock firmware (non-rooted for several reasons like banking software - it's my daily smartphone). Unfortunately, I quickly failed since `adb root` [doesn't work](https://stackoverflow.com/questions/25271878/android-adbd-cannot-run-as-root-in-production-builds) on production builds (with the `adbd cannot be runned in the production builds` error). Since I didn't want to root my device or use other dirtier tricks, I decided try to collect PGO profiles from an emulator with Android Studio installed on my Macbook. I achieved a quick fail again since according to the `simpleperf E event_type.cpp:508] Unknown event_type 'cs-etm', try simpleperf list to list all possible event type names` error the emulator doesn't emulate required functionality (at least with my naive setup) for [Simpleperf](https://developer.android.com/ndk/guides/simpleperf). Is it possible to collect PGO profiles with the emulator or not - I don't know. I guess that it's not possible for sampling PGO approach but possible for the instrumention - but instrumentation way is already deprecated :D Besides that, there are addional traps. There was a need to [patch](https://issuetracker.google.com/issues/337338041) AutoFDO tooling for the Android use case - already [fixed](https://issuetracker.google.com/issues/337338041#comment3). Also, some `profcollectd` [limitations](https://android.googlesource.com/platform/system/extras/+/main/profcollectd/) like Coresight ETM requirement for ARM (so I suppose other platforms like x86-64 are not supported at all with this tool).

Apple and Google with their centralized App Store/Play Store infrastructure also can bring additional value to PGO movement. One of the biggest issues with using PGO for applications on mobile platforms is the question "How to collect PGO profiles from clients?". Implementing PGO profile collection logic from clients' smartphones for each developer is too complicated way. It would be much better to integrate it as a developer platform. Ideally, this can look like just as an additional option in the application settings like "Enable PGO profiles collection". Users during the installation can opt-in/opt-out "Share with developers application execution profiles for improving performance" button, App Store/Play store then provides all required routines for delivering profiles from mobile phones to a developer build machine for further usage. It will allow developers in a much easier way collect actual PGO profiles from real users - a good thing to have for the whole ecosystem. Unfortunately, now there is no such functionality out of the box - if you want something similar, you need to implement it manually. I [found](https://issues.chromium.org/issues/326465539) some traces about usage [Finch](https://news.ycombinator.com/item?id=12954588) with PGO but without further details yet.

There are so many questions to research in this area and so much experience to get like "Is it possible to use Sampling PGO on mobile devices (with Linux perf on Linux-powered OS or others like iOS - doesn't matter) in the way like it's done on the desktop with the `perf` tool"? In theory - yes, why not? In practice - I didn't see such experiments before, we need to investigate it. If PGO usage will be expanded across the mobile domain, I suppose more questions will be raised.

### Explore PGO for HPC domain

High-Performance Computing (HPC) domain, according to the name, is very interested in better optimizations. Here even several percent optimization results are valuable since in many situations calculations can take several *weeks* on large computing clusters.

I don't have any results to share for HPC-related software since I don't have enough experience in this area. My friend that have a lot experience with HPC applied PGO to some internal developments and got very interesting results (unfortunately nothing to share publicly).

I think it will be interesting to perform multiple benchmarks on software like [HPC Foundation](https://hpsfoundation.github.io/). If you have something to share - please let me know!

### Explore PGO for machine learning domain

TODO: add my inference benchmarks here
TODO: add https://github.com/guillaume-be/rust-bert and https://github.com/guillaume-be/rust-tokenizers examples

Machine learning (for non-engineers - AI. However, I don't think that non-engineers will survive to this point of the article) is a hot topic nowadays. Performance here is especially important since running and especially training of modern neural networks is a **very** resource-consuming thing. However, from the PGO perspective here we have multiple challenges.

Most modern ML-related workloads nowadays are executed on GPU or more specialized hardware like TPU (Tensor Processing Units) or even more specialized hardware - major PGO implenentations right now don't support non-CPU targets. Most things around ML training is quite simple math logic like matrix multiplications that are already can be successfully optimized to the peak performance by compilers. Where compilers cannot achieve this goal - clever humans already did all the work for them and optimized all the code manually with CUDA, SIMD and similar technologies.

I have several ideas where the performance in the field can be improved: tokenizers (check [this](https://github.com/huggingface/tokenizers/issues/1426#issue-2069318532) benchmark), serving ML models (like Tensorflow Serving ([issue](https://github.com/tensorflow/serving/issues/2192)) and Triton ([issue](https://github.com/triton-inference-server/server/issues/6304))), ML compilers (like `tfcompile` ([benchmark](https://github.com/tensorflow/tensorflow/issues/60944#issuecomment-1637143591)) or Nvidia Fuser ([issue](https://github.com/NVIDIA/Fuser/issues/1185))). Maybe there are other places where PGO can bring measurable improvents for the ML domain like optimizing models inference on handheld devices (where battery life matters).

However, I want to see a bit different thing. PGO is just a specific implementation of a more generic idea - data-driven optimizations. I believe that this approach can and should be extended to other optimization fields like GPU/TPU optimizations. For now, I don't see many researches in this area but I hope this situation will change in the future. I am not a ML engineer and have very limited knowledge in this domain but I am sure that in ML there are many places where data-driven decisions can improve things too. And making such decisions can be at least semi-automated.

### Explore PGO for operating systems and drivers

Operating systems are **very** good candidates for PGO. Large piece of software, a lot of branches inside, many different workloads - sounds like a good PGO target, isn't it?

For Linux kernel we have a lot of available PGO materials: the ASOS idea discussed above, a [paper](https://web.eecs.umich.edu/~takh/papers/ugur-one-profile-fits-all-osr-2022.pdf), a [presentation](https://lpc.events/event/7/contributions/771/attachments/630/1193/Exploring_Profile_Guided_Optimization_of_the_Linux_Kernel.pdf) from Microsoft, Linux Plumbers 2020 [slides](https://lpc.events/event/7/contributions/798/attachments/661/1214/LTO_PGO_and_AutoFDO_-_Plumbers_2020_-_Tolvanen_Wendling_Desaulniers.pdf), more [slides](https://lpc.events/event/7/contributions/771/attachments/630/1193/Exploring_Profile_Guided_Optimization_of_the_Linux_Kernel.pdf) from Linux Plumbers, a benchmark [results](https://github.com/h0tc0d3/linux_pgo) and corresponding [article](https://habr.com/ru/companies/ruvds/articles/696236/) (in Russian), "Experiences in Profile-Guided Operating System Kernel Optimization" [paper](http://coolypf.com/apsys2014.pdf) with corresponding [article](http://coolypf.com/kpgo.htm) (10 years old - just a warning), Phoronix [post](https://www.phoronix.com/news/Clang-PGO-For-Linux-Next) about kernel PGO with Clang, Gentoo [wiki](https://wiki.gentoo.org/wiki/Kernel/Optimization#Performance). ChromeOS also uses PGO (via Sampling) for building its kernel ([click](https://issuetracker.google.com/issues/245629081), [clack](https://issuetracker.google.com/issues/327058411)). For Windows the only found by me material is already [linked](https://lpc.events/event/7/contributions/771/attachments/630/1193/Exploring_Profile_Guided_Optimization_of_the_Linux_Kernel.pdf) above presentation from Linux Plumbers (lol) - check the last slides of the presentation. It will be great to see PGO benchmarks for other operating systems: open-source like *BSD (like I [asked](https://www.reddit.com/r/freebsd/comments/150codp/profileguided_optimization_pgo_on_freebsd_kernel/) about PGO support for FreeBSD) and many others, and proprietary like macOS (yep, I know that its kernel is open-source). I already asked about testing PGO for different operating systems, and some maintainers already shown some interest about this topic. Unfortunately, without any further (at least publicly-visible) activity in this area (yet). Despite all the benchmarks above, some maintainers are still [skeptical](https://github.com/ValveSoftware/SteamOS/issues/1287#issuecomment-1840835012) about PGO usefulness for operating systems.

Drivers are another topic for investigation. I can expect that drivers performance (CPU part) can be improved as well. Drivers are not something so specific that cannot be optimized with PGO: usual code (mostly in "native" technologies like C, C++, Rust), many branches, executed mostly on CPU - sounds like a good PGO candidate. Right now I have nothing to show you about PGO in drivers. I [opened](https://gitlab.freedesktop.org/mesa/mesa/-/issues/10155) a request in Mesa - no results yet. Other good candidates to test - Linux in-tree drivers like filesystems and networking. Since they are in-tree and Linux kernel already has ready-to-use PGO switches, it should be easy enough to test drivers performance with PGO. Of course, drivers for other operating systems can be optimized with PGO as well.

Why optimizing OS and drivers with PGO could be a non-trivial task? Preoptimizing an OS (and drivers) with PGO can be non-trivial task since workloads for a generic-purpose OS can be too generic: browsing, gaming, some server-like workloads, etc. It's possible to gather different profiles and prepare some "optimized for a generic usage" OS build but you need to collect such profiles somehow, ideally from actual users - no ready-to-use ecosystem exists for that nowadays. Next difficult step - saving PGO profiles. OS is a bit specific kind of software that could require special modifications around PGO profile dumping logic. E.g. for optimizing the Linux kernel with PGO you [need](https://wiki.gentoo.org/wiki/Kernel/Optimization#GCC_PGO): rebuild the kernel with some specific options, and then PGO profiles will appear at specific paths at `debugfs`. Pretty simple to use but someone already implemented it in the kernel - you will need to do similar things with your OS too. Besides that, additional issues can [appear](https://github.com/asterinas/asterinas/issues/760#issuecomment-2065951706) like guest/host interactions. So we have a lot of things to improve the PGO UX for operating systems too.

### Expand PGO usage across the Go ecosystem

Nowadays we have **a lot** of software written in Go. Since we want to achieve better CPU performance across the industry, we need to optimize them too.

Right now Go implementation is too young and misses many PGO-related optimizations from more mature compilers (more details see above in the section about the PGO state in Go). I think we can wait a bit for the implementation and then start introducing PGO into more and more Go applications since I don't want to push projects to adapt not so finished yet technology (my personal opinion - you can have other point of view!). If you want to try achieving additional performance boost with PGO for your Go project and you are ready to use such a technology (and possibly fight a bit with a compiler) - go ahead, that's completely ok! I already know several positive PGO cases for Go applications from large companies like Datadog. For most of other project the proper time will come a bit later when the compiler will implement more PGO-related optimizations inside. [CNCF](https://www.cncf.io/) landscape, be careful - PGO is coming to you!

### Migrate projects from FE PGO to IR PGO

As we discussed at the beginning, IR PGO is the preferred way to do instrumentation-based PGO nowadays in the LLVM ecosystem. Unfortunately, for some reason, this information was not available in the documentation until recently. The only way to find it was to find the corresponding [issue](https://github.com/llvm/llvm-project/issues/45668) in the LLVM issue tracker or find a similar [discussion](https://discourse.llvm.org/t/profile-guided-optimization-pgo-related-questions-and-suggestions/75232/2?u=zamazan4ik) at LLVM forum. At the same time, official Clang documentation about PGO [suggested](https://clang.llvm.org/docs/UsersManual.html#profiling-with-instrumentation) using FE PGO (`-fprofile-instr-generate` compiler flag). People read the documentation ,and went with the first option in the documentation - with FE PGO. Oops! Fortunately, current Clang documentation is alrady fixed and suggests to use IR PGO by default.

There are already some projects that integrated FE PGO in their PGO pipelines. We need to help such projects migrate to the recommended and more perspective IR PGO. I created a tracking [issue](https://github.com/zamazan4ik/awesome-pgo/issues/3) for the activity. Probably the project list will be extended in the future.

### "Good first PGO project" issue list

I prepared the "[Are we PGO yet?](https://github.com/zamazan4ik/awesome-pgo/blob/main/are_we_pgo_yet.md)" list for several reasons. At least, I need a place to track where I already created a PGO request - human memory is not a very sustainable thing, you know. But this list also can be a good place where any potentially interested in PGO person can find some open-source project, and try to contribute PGO into it. I create PGO evaluations requests only for projects where I expect some performance gains from enabling PGO (based on my current expertise in this field). So if you want to play with PGO and just looking for a good playground - you can find something interesting in the list. It will be a win-win situation: you can polish your PGO skill on a project, and the project has a chance to get PGOed by the community. Of course, this list is not complete at all and I missed many projects that can benefit from PGO. However, it's better than nothing IMO.

### Perform more PGO benchmarks

Even if I already performed PGO benchmarks for many projects, I have a bunch of other projects in my TODO list! For some projects I even didn't create a "Evaluate PGO usage" request in the corresponding issue tracker. Maybe one day I will have enough motivation to continue work in this direction. The main reason why PGO benchmarks for some projects are postponed is amount of efforts to perform the benchmarks. If a project something more difficult than `cargo pgo bench` && `cargo pgo optimize bench` - the priority for this project is decreased in my list because I have a lot of other easier-to-test projects anyway!

But if someone wants to help, here I collected some potentially interesting from PGO perspective open-source projects:

* Erg: [GitHub repo](https://github.com/erg-lang/erg)
* Stract: [GitHub repo](https://github.com/StractOrg/stract)
* chipStar: [GitHub repo](https://github.com/CHIP-SPV/chipStar)
* starship: [GitHub repo](https://github.com/starship/starship)
* littlefs: [GitHub repo](https://github.com/littlefs-project/littlefs)
* Darktable. [These](https://github.com/darktable-org/darktable/blob/master/src/tests/benchmark/darktable-bench) can be used for PGO evaluation.
* revng: [GitHub repo](https://github.com/revng/revng-c)

Much more projects can be found in the "Are we PGO yet?" [list](https://github.com/zamazan4ik/awesome-pgo/blob/main/are_we_pgo_yet.md).

### Contribute to project-specific PGO documentation

It's always convenient to have project-specific information as close to the project as possible like in a project documentation. Especially if the project has some implementation nuances in its PGO implementation (like a [custom logic](https://github.com/yugabyte/yugabyte-db/commit/34cb791ed9d3d5f8ae9a9b9e9181a46485e1981d) in PGO dumping process). Many projects still miss PGO-related documentation like Rsyslog ([GitHub comment](https://github.com/rsyslog/rsyslog/issues/5048#issuecomment-1694533339)), YugabyteDB ([GitHub issue](https://github.com/yugabyte/yugabyte-db/issues/18497)) or Lace ([GitHub comment](https://github.com/promised-ai/lace/issues/172#issuecomment-1917406027)). Having such documentation can improve PGO adoption across projects' users.

### "Are we PGO yet" website

It can be helpful to have a resource where you can check the current PGO status for your favorite software: compiler, broswer, IDE, etc. Now you need to check build scripts, ask upstream developers/maintainers about the current state - too time-consuming. What if will be a resource where you can simply type let's say "MongoDB" and check current PGO status, PGO benchmarks in different scenarios, are MongoDB builds PGO-optimized and other information?

Now [awesome-pgo](https://github.com/zamazan4ik/awesome-pgo) is such a place. However, I suppose UX could be done much-much better because navigating over a dozen of links is [not easy](https://github.com/facebook/mariana-trench/issues/137#issuecomment-1658048628) for some people. We can try to imeplement a dedicated "Are we PGO yet?" website like it's done for many other domains - check [Areweyet website registry](https://wiki.mozilla.org/Areweyet) or [Arewemetayet](http://arewemetayet.com/). I didn't implement it yet because it requires some extra efforts from my side (and I have more valuable work to do, in my opinion). However, if anyone decides to help me here - I would appreciate it a lot!

### Write a PGO book

It would be nice to have something like "The PGO book" for people who want start using PGO. Read about different PGO approaches, pros and cons of each, check some PGO reference architectures, etc. This article is the closest to that (IMO) but currently the information is a bit unstructured. Having a good reference can lower the entry barrier into the PGO world, it consequently can increase PGO usage across applications - exactly what I want to achieve!

There are plenty of good examples to look at: [Rustbook](https://doc.rust-lang.org/book/), [The Rustonomicon](https://doc.rust-lang.org/nomicon/), [Rust Async book](https://rust-lang.github.io/async-book/), [The Rust Performance Book](https://nnethercote.github.io/perf-book/title-page.html), [Unofficial Bevy Cheat Book](https://bevy-cheatbook.github.io/). Yep, all these books are somehow connected to the Rust ecosystem since Rust popularized it a lot, and I am very glad to see such a change across the industry!

Maybe one day I will write something similar but not today - I am too tired. Maybe next year? By the way, I already got at least [one](https://github.com/lycheeverse/lychee/issues/1247#issuecomment-1882078012) request for such a book - it can bring me more motivation for doing it!

### Improve PGO state in various ecosystems

TODO: write here at least about Java AoT state (that blocks PGO efforts) and Go (we are waiting for the more mature PGO implementation in the main Go compiler), CNCF projects opportunity

* We have a lot of Java software nowadays. And before we Rewrite It In Rust (RIIR) we need to care about the Java performance too. [GraalVM](https://www.graalvm.org/) helps with AoT compiling Java applications to the native mode. Since we want to integrate PGO into Java too, at first we need to integrate GraalVM well into the ecosystem.

### Improve PGO tooling

From the tooling perspective, we have a lot of things to improve. And here I am not even talking about build systems/package managers integrations. There are many more ways to integrate PGO into the daily life.

I created an [issue](https://github.com/compiler-explorer/compiler-explorer/issues/5414) about integrating PGO into the [Godbolt](https://godbolt.org/) platform. Godbolt is a great place to play with different compilers, and compiler flags to measure their influence on the compiled programs. I think it would be great to have the possibility to show differences between PGOed and non-PGOed code on some toy examples directly on Godbolt. Right now it's not possible to do since PGO requires to store somehow the gathered profiles from the instrumentation phase but Godbolt is stateless (from this perspective). Hopefully, someone would be interested enough to implement it.

It seems to be an interesting idea to implement some kind of PGO profile integration into IDE. Let's say you open your favorite IDE/text editor, open some source code and right here you can see hot and cold paths in your code. This information is not just your mental prediction (like "exception path highly likely is a cold path") but based on the actual PGO profiles. How can you use this information? Well, e.g. for load balancing your efforts regarding optimizations across your program - take care first about the hot paths in your application. If there is a [request](https://youtrack.jetbrains.com/issue/IDEA-332604/Support-voice-messages-code-comments) voice message in code comments support (and someone even [implemented](https://github.com/polina4096/voices) it!) - why not have PGO support? :)

### Improve PGO/PLO profiles gathering ecosystem

The original idea is described [here](https://github.com/aaupov/school_topics/blob/main/perfd.md) (in Russian, so if you don't know this language - please use the translator or ask Amir about making it in English :), a few initial commits are available [here](https://github.com/linux-perfd/perfd).

The same thing is already implemented internally in Google. This system samples all applications across Google servers with Google Wide Profiler (GWP), transforms sample profiles into a compiler-compatible format, and then passes converted profiles to the build pipelines. More about GWP and its connection to PGO at Google can be found in these papers: [one](https://research.google/pubs/autofdo-automatic-feedback-directed-optimization-for-warehouse-scale-applications/), [two](https://research.google/pubs/google-wide-profiling-a-continuous-profiling-infrastructure-for-data-centers/). Google, do you want to open-source this system (even if this system has too many dependencies on other closed-sourced Google projects)? :) Other large companies also has similar system-wide profilers like Yandex with its [Yandex.Perforator](https://cppconf.ru/en/talks/45d2d5121e09431da5826778851b91ba/) or Ozon with its Vision ([article](https://habr.com/ru/companies/ozontech/articles/817309/), in Russian). Unfortunately, these projects cannot be used for continuous PGO yet. Maybe in the future Perforator will be extended with such functionality and open-sourced (as they already did for multiple awesome projects like [ClickHouse](https://clickhouse.com/), [YDB](https://ydb.tech/), [YTsaurus](https://ytsaurus.tech/)). If you work in a large company and you have a similar company-wide profiling platform - you can try to implement your custom continuous PGO optimization platform!

There is an idea to implement a similar approach based on an open-source project like Grafana Pyroscope ([discussion](https://github.com/grafana/pyroscope/discussions/2783)) or Elastic Universal Profiling ([issue](https://github.com/elastic/elasticsearch/issues/105802)). I am not alone with such an idea - other people outside Google already [tried](https://discuss.elastic.co/t/continuous-profile-guided-optimization-pgo-platform-based-on-elastic-universal-profiling/354605/2) to implement a similar approach with positive results. However, the current state of these activities is just a discussion, nothing to try yet.

Google Cloud has a [Cloud Profiler](https://cloud.google.com/profiler/docs/about-profiler) product that also can be used for Continuous PGO. I am guessing that Cloud Profiler internally somehow reuses Google Wide Profiler. This project allows continuosly measure performance of GCP-deployed applications (out-of-GCP scenarios are also supported) from multiple perspectives: CPU, RAM, thread and lock contentions, etc. I [asked](https://www.googlecloudcommunity.com/gc/Google-Cloud-s-operations-suite/Continuous-Profile-Guided-Optimization-PGO-platform-based-on/m-p/743125) and created the [issue](https://issuetracker.google.com/issues/337900460) about evolving Cloud Profiler into a Continuous PGO platform - let's wait for the answer. Some people also [wants](https://issuetracker.google.com/issues/296460994) such functionality. I didn't play with the product yet but from reading the documentation I found a strange limitation - at least from the documentation point of view measuring performance for applications written in C, C++, Rust, etc. is not supported yet. I [opened](https://www.googlecloudcommunity.com/gc/Google-Cloud-s-operations-suite/Support-for-more-programming-languages-in-Cloud-Profiler/m-p/743114) a discussion about that (and the corresponding [issue](https://issuetracker.google.com/issues/337905339)). What about other cloud providers like AWS and Azure? I didn't see similar products in their portfolio. If you know more about that - please let me know.

Another idea is gathering PGO profiles (instrumentation or sampling, whatever) via a crowd-sourcing-like mechanism. One of the biggest issues with PGO is collecting actual usage profiles. But what if we had a repository where each software engineer can get actual PGO profiles for its application and use it for PGO-optimized builds on their official site? It could be done e.g. via a specific OS build where all PGO-potent applications are built with instrumentation or continually collect sampling profiles via `perf` or something like that. There are **a lot** of concerns here like privacy issues (potential mapping "some_user_id -> application execution profiles"), security issues (organizing an attack on an application performance via messing publicly-gathered PGO profiles), implementation (who and how would implement all this stuff?!), and much more. Maybe community-driven projects like [Boinc](https://boinc.berkeley.edu/) could be a good example here - right now it's just an idea, no more. I am [not alone](https://www.phoronix.com/forums/forum/phoronix/latest-phoronix-articles/1422805-llvm-now-using-pgo-for-building-x86_64-windows-release-binaries-~22-faster-builds?p=1422977#post1422977) with such thoughts. Chromium already [has](https://chromium.googlesource.com/chromium/src.git/+/master/docs/pgo.md) something similar but access for such profiles is a bit [limited](https://issues.chromium.org/issues/325519354) and it can raise many interesting discussions about profiles transparency, reproducibility, access for OS distributions, etc.

The Rust ecosystem has wonderful [Crater](https://github.com/rust-lang/crater) project that allows perform large compilation experiments across many Rust packages - I have some thoughts about this tool can be (ab)used for PGO purposes: run PGO benchmarks at once across many crates, test different PGO/PLO approaches at scale, gather as much benchmark numbers as possible, test new PGO implementations on it, collect PGO profiles in a semi-centric way... Current Crater implementation of course is not ready for such purposes and need to be expanded with corresponding functionality but anyway - other ecosystems don't have even this thing! Unfortunately.

### Improve PGO/PLO visibility across the industry

PGO and similar optimizations have too poor visibility across the industry (IMHO). I think spreading the word about PGO, its benefits, traps, etc. would be a valuable investment to the whole industry. It's one of the reasons why I wrote this article.

Talk more about PGO at the conferences. I like conferences because of having a lot of tech-minded people around me - it's a lot of fun! I think it would be great to give multiple talks about PGO at different conferences. It could be language-specific conferences (like [Rust](https://foundation.rust-lang.org/events/) or [C++](https://isocpp.org/wiki/faq/conferences-worldwide) conferences) where we can discuss language-specific PGO aspects. It could be something more specific like "PGO from a maintainer perspective" on [PackagingCon](https://packaging-con.org/). Maybe even prepare some PGO workshops (like [performance workshops](https://conf.researchr.org/track/cgo-2023/cgo-2023-workshops-and-tutorials) on CGO) - why not?

This year I attended [FOSDEM 2024](https://fosdem.org/2024/) (it was my first FOSDEM btw) where I discussed PGO from different perspectives (open-source project developers, OS maintainers) and gathered a lot of valuable thoughts about this optimization. It was a win-win (IMHO): I started to better understand the main PGO pain points for different stakeholders, and I hope at least some people started to consider using PGO for their projects. Would be nice to have the same kind of conversation at many other conferences!

I am thinking about trying to collaborate with large firms like Google on a project about integrating PGO into different kinds of software as a part of [Google Summer Of Code](https://summerofcode.withgoogle.com/) (GSoC). Regarding GSoC, I have talked about at FOSDEM 2024 with GSoC representatives at the conference about expanding PGO usage across the industry. Unfortunately, GSoC is not a place where I can suggest integrating PGO into multiple projects - I need to propose this idea for each project instead, one by one, and only then it's possible to use GSoC resources for integrating PGO into each project. It's a pity but maybe there are other places? What about making collaboration between PGO and Hacktoberfest, huh? Does anyone know the way how can I push PGO in some more centralized way for the ecosystem? If someone has thoughts/ideas or even ready-to-spend (not waste!) budget - please let me know! Maybe some corporation with a thick wallet wants to contribute to this field...

### Expand PGO usage across programming languages

Nowadays we have many programming languages. If we want expand PGO usage, these programming languages (more precisely - compilers for these programming languages) should support PGO. In this case, all programs written in a programming language X can be optimized with PGO.

As we've seen above, current support for PGO across different language ecosystems isn't so great... I already created many issues regarding that across multiple compilers. Unfortunately, I didn't see much interest from compiler developers in implementing it. Maybe they just don't believe in the PGO efficiency (that's why I wrote the article!). Of course, ideally PGO should be integrated not only into the compiler: a build system, a package manager, etc. - but the compiler support is the most important part. If you evaluate some new programming language for your use cases and you care about performance - please ask about PGO support in the upstream. Demand creates supply.

## Some random thoughts why PGO usage is so low nowadays

TODO: discuss some proposals about JIT-ing everything
TODO: Some people actually read the article! https://github.com/asterinas/asterinas/issues/760#issuecomment-2060277998

Sometimes I hear "Just use JIT-compiler - it "automatically" does all that optimizations without problems like profile transfer, profile format conversion, etc.". Unfortunately, JIT way has its own limitations:

* Sometimes JIT usage is limited. As an example - Apple's [Hardened Runtime](https://developer.apple.com/documentation/security/hardened_runtime)
* JIT has problems with cold start time. When you run your application at first time, it needs to be compiled on-demand. It could be a problem for one-shot workloads like serverless stuff. Of course it can be mitigated via "warming up" procedures but it can be non-trivial to do. AoT doesn't have such problems. How is big the difference in practice? Check [this](https://maxday.github.io/lambda-perf/).
* JIT has runtime overhead on a target machine. The amount of overhead completely depends on your compiler (nice hint: if your JIT compiler is PGO-optimized, the overhead will be lower ;). However, if you want to apply as many as possible compiler optimizations (sometimes too time-consuming like [Souper](https://github.com/google/souper)) to optimize your code to the peak performance - overhead will be higher and higher. It's inevitable because all compiler optimizations, obviously, consume CPU resources. AoT compilation model solves this probelm with separating compilation and execution environments: users with their smartphones don't care how big and beefy your CI pipeline is but care a lot about battery life. And remember: JIT optimization will be executed on **every** target machine instead of being done once with AoT case (okay, N times if we do PGO build per workload but tbh it's quite rare case in practice).

That's why I still think that investments into PGO (and similar) optimization techniques for AoT is still a way to go.

From my experience, from talks with multiple developers (open-source and proprietary), from talks with package maintainers, I can highlight the following reasons why PGO adoption is low nowadays (IMHO):

TODO: describe in details each point

* Lack of knowledge about PGO existance.
* Lack of knowledge about PGO efficiency in practice.
* Weak tooling support.
* Additional maintenance cost.
* Lack of knowledge how to apply PGO in practice.
* Conservative point of view regarding new compiler optimizations aka fear of additional compiler bugs.
* Additional source of non-determinism.

Let's discuss each point in details!

### Lack of knowledge about PGO existance

TODO: add multiple proofs when developers didn't hear before about PGO from GitHub

### Lack of knowledge about PGO efficiency in practice

Even when engineers heard about PGO before, they don't believe or just [didn't know](https://github.com/candy-lang/candy/discussions/953#discussioncomment-8584992) well about PGO efficiency in practice. Why do we have such situation? I see multiple reasons. At first, PGO implementors (compiler engineers and their employers) don't talk enough about PGO in public-visible places like conferences, articles, etc. All what I see it's rare articles and some hidden papers. And even in these papers in many situations the results are demonstrated on things like [SPECint](https://en.wikipedia.org/wiki/SPECint) bench quite. I know that SPECint is not that bad, and it consists of many near real-world workloads. But hey - I don't have in my production environment SPEC benchmark! I have PostgreSQL, Vector and HAProxy (of course your environment can be different but you got the idea). For me as for an engineer would be much more helpful to see PGO results for my actual software - I don't have time to learn about SPEC because I don't care much about that thing, I am not a compiler engineer.

How to fight it? Exactly for this reason I collect as much as possible PGO benchmarks for various **real** software in different domains. I hope as more PGO benchmark examples will be demonstrated, the lower skepticism level will be across the industry about PGO. Will it work in practice? Who knows... Fortunately, I see a tendency from large companies like Datadog and Cloudflare (like [this](https://blog.cloudflare.com/reclaiming-cpu-for-free-with-pgo) one) to share their experience with PGO - their examples can be a good motivation for the industry giving PGO a try.

### Weak tooling support

TODO: https://github.com/grafana/loki/issues/11939#issuecomment-1966915084

### Additional maintenance cost

TODO: add examples:
  - https://github.com/facebook/hhvm/issues/9385#issuecomment-1943760010
  - https://github.com/ValveSoftware/SteamOS/issues/1287#issuecomment-1840835012
  - https://github.com/BurntSushi/ripgrep/issues/1225#issuecomment-1828965231

### Additional source of non-determinism

TODO: write about expanding ML-based stuff and how it introduces more non-determinism. Add reference to a complicated ML-landscape
TODO: mention reproducibility issue with non-deterministic stuff. Add a reference to Solarwinds and its reworked CI pipeline

### Conservatism

Developers have a lot of things to do. Implement features, fix bugs, fight with dependencies/OS updates, configure CI/CD, tooling traps and many-many other trickier things (like [-O2 vs -O3 performance differences](https://news.ycombinator.com/item?id=28895896)) - they are busy with all of that stuff. We like when things "just work" without a need to spend/waste additional amount of time to figure out "what is going on in this case again holy crap".

And here appears PGO/PLO. Not so widely-used technology, with high-amount of non-determinism, weak tooling support, maintenance cost - too high risks for many projects (especially if we are talking about mature projects like PostgreSQL). And in exchange for all these risks you get a *chance* to improve performance for your application. Not so many applications care so much about its performance that taking these risks seem a good choice, tbh. We are tend to reduce risks as much as we can because we don't want to resolve all potential issues from enabling such fancy optimizations - we are humans, and we are lazy by design.

What can we do here? Reduce risks. How? I have several ideas:

TODO: more ideas?

* Explain developers how PGO works in practice. Talk about unknown PGO things a lot and make them visible because unknown things are highly unpredictable.
* Improve PGO tooling since it helps with reducing PGO maintenance cost for projects.

## Call for action (instead of conclusion)

TODO: finish the chapter

* Piece of advice for the developers:
  - If you are a language/compiler designer - please consider integrating PGO into your language/compiler
  - If you are a project developer - please consider providing better PGO integration into your project if you care about the performance
  - If you are a maintainer - please consider enabling PGO in your packages
  - If you are an OS package committee member (like Fedora FESCO) - please consider general PGO movement across the whole package policy
  - If you have experience with PGO in production - please share your numbers/pains/experience with us! Examples from huge companies are interesting to read and they are really motivating. E.g. Datadog developer [promised](https://twitter.com/felixge/status/1754772490889396415) to share their experience with PGO - I didn't forget ;)

I want to finish with the call to the community.

If you are a language designer - please at least consider implementing AoT compilation model for your language. Of course, this model is not suitable for all cases but would be nice to have the possibility to compile (and preoptimize with PGO) a program before the launch. Cold start time matters nowadays, you know.

If you are a language/compiler designer - please consider integrating PGO into your language/compiler

I hope you enjoyed the article! I tried to share all my experiences with PGO. And if at least one person after reading all these materials will try to integrate PGO into their applications - I would be happy!

What about my short-term plans? I spent almost 1.5 year on this work on a **daily** basis, almost all my free time was contributed to this work. So yeah, I need some rest to restore my burnt down soul from PGO. Maybe go to a forest for a week or so, and listen to birds with a cup of warm shroom tea (youknowwhatimean)...

## My contacts

If you want to discuss with me more things about PGO, think about contributing to PGO or know anything PGO-related, you can find me via:

* Email: zamazan4ik (at) tut (dot) by (primary email), zamazan4ik (at) gmail (dot) com (secondary email)
* Telegram: zamazan4ik
* Discord: zamazan4ik
* Twitter/X: [zamazan4ik](https://twitter.com/zamazan4ik)

Also, you can create an issue or open a discussion at https://github.com/zamazan4ik/awesome-pgo, if you want to contribute PGO results or discuss something publicly. Just be careful about possible NDA violations and similar stuff.

## License

This work is licensed under a Creative Commons Attribution 4.0 International License. However, images (if any) are kindly taken from the Internet without checking the license question properly because I don't care much ;)

TODO: article meme ideas

* meme about Nier androids and "Are they PGO optimized?"
* meme about DMC Vergil and "Show me your PGO motivation"
* meme about Rick and Morty "Go here and there  - 10 minutes" about PGO journey
* think about Rika Makiba meme and PGO? What would be a better idea here?
* make a meme about CompilerGym where GCC, Clang and other are making exercises in a gym (gachi would be even more fun!)

