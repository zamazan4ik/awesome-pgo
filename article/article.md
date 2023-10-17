# Profile-Guided Optimization (PGO): a long hard road out of hell

In this article, I want to discuss one compiler optimization technique - Profile-Guided Optimization (PGO). We will talk about PGO pros and cons, obvouis and tricky traps with PGO, take a look on the current PGO ecosystem from different perspectives (programming languages, build systems, operating systems, etc.). I spent many days with applying PGO to different applications... Tweaking build scripts, many measurements, sometimes tricky debug sessions, several compiler bugs and much more - it was (and still is) a great journey! I think my experience would be helpful for other people so you will be able to avoid my mistakes from the beginning and much nicer optimization experience.

The article will be huge enough, so do not forget to grab some tea, beer or something like that before the begginning. Enjoy!

## Attention anchor

Many of the technology-related people likes numbers, especially when we are talking about performance. Since I want to get your attention to the topic of how PGO helps in real-life, I will show you at the beginning some performance numbers with links to the actual benchmarks.

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

## Intro

I like performant applications. 

## Target audience

I think the article would be interesting for anyone, who cares about software performance. Especially it should be interesting for people who wants to extract more performance from applications without tweaking the application itself.

## PGO and other similar names

PGO has multiple other names, that means **completely** the same thing:
* Profile-Guided Optimization (PGO)
* Feedback-Directed Optimization (FDO)
* Profile-Driven Optimization (PDO)
* Profile-Directed Feedback (PDF)

At most places, you will see PGO (like in Clang (TODO ref), GCC (TODO ref), Rustc (TODO ref)). Sometimes, FDO is used (like in Bazel (TODO ref)). Other names almost never used. In this article, I use only PGO abbreviation.

## Why am I writing this?
* I like performant applications
There are multiple reasons for that:
* For me it would be easier to work in the industry, where we have "PGO by default" mindset. Because with faster software it's easier to achieve required NFR (Non-Functional Requirements) before the horizontal scaling questions.
* Because I can
* Just for lulz

## FAQ

### Do I need to use LTO with PGO?

No! Link-Time Optimization (LTO) and PGO are completely independent and can be used without each other.

However, usually LTO is enabled before PGO. Why it happens? Because both LTO and PGO are optimization technique with an aim to optimize your program. And usually enabling LTO is much easier than PGO because enable one compiler/linker switch with LTO is an easier task than introduce 2-stages build pipelines with PGO. So please - before trying to add PGO to your program try to use LTO at first.

There are some situations, when you may want to avoid using LTO with PGO:

* Weak build machines. LTO (even in ThinLTO mode) consumes a large amount of RAM on your build machines. That means if your build environment is highly memory-constrained - you may want to use PGO without LTO since PGO usually has lighter RAM requirements for your CI.
* Compiler bugs. Sometimes PGO does not work for some reason with LTO (TODO: add Rust compiler bug issue here)


TODO: note about MSVC compiler and combined LTO/PGO mode as the only option to enable PGO.

TODO: Possible plot for the talk:

1. Why did I write this post? Intro why I like fast software, etc.
2. Show, why PGO is a worth thing to discuss with performance numbers for the most famous projects like PostgreSQL, Clang, MongoDB, 
2. PGO current ecosystem state from multiple perspectives: languages, compilers, problems, etc.
3. What it can be in the future?