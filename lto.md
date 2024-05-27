# LTO state

LTO is not widely used in the distributions. However, more and more OS enable LTo by default for their packages. Is it enabled by default on your distribution - ask your maintainers.

Is LTO enabled in other binaries - ask their authors. Do not forget that enabling LTO could uncover some bugs: [GentooLTO](https://github.com/InBetweenNames/gentooLTO/issues) or [this](https://github.com/yugabyte/llvm-project/commit/64d871949eb23145af7b97cb13feaeeeee7ab39a).

State across distributions:

* Debian: [link](https://wiki.debian.org/ToolChain/LTO)
* Ubuntu: [link](https://wiki.ubuntu.com/ToolChain/LTO)
* Fedora: [link](https://fedoraproject.org/wiki/LTOByDefault)
* OpenSUSE: [link](https://en.opensuse.org/openSUSE:LTO)
* NixOS: [Reddit](https://www.reddit.com/r/NixOS/comments/146wdfk/lto_by_default/)
* Gentoo: TODO

Performance improvements from enabling LTO:

* Vector: https://github.com/vectordotdev/vector/issues/15631#issuecomment-1694554798
* Ladybird: https://twitter.com/awesomekling/status/1702247511698587678
* Symbolicator: https://github.com/getsentry/symbolicator/issues/1334
* pylyzer: https://github.com/mtshiba/pylyzer/discussions/80#discussion-6500001
* Rustc: https://issues.fuchsia.dev/issues/42066512
* prettyplease: [GitHub comment](https://github.com/dtolnay/prettyplease/issues/74#issue-2292685589)
* resvg: https://github.com/RazrFalcon/resvg/issues/765#issue-2314946307

Links:

* https://convolv.es/guides/lto/
* Multiple great articles about LTO (mostly in GCC but sometimes LLVM is covered too) with A LOT of benchmarks from Honza Hubička: [LTO history](https://hubicka.blogspot.com/2014/04/linktime-optimization-in-gcc-1-brief.html), [GCC 4.8](https://hubicka.blogspot.com/2014/04/linktime-optimization-in-gcc-2-firefox.html), [GCC 4.9.1 and LLVM 3.5](https://hubicka.blogspot.com/2014/09/linktime-optimization-in-gcc-part-3.html), [GCC 5](https://hubicka.blogspot.com/2015/04/GCC5-IPA-LTO-news.html), [GCC 6](https://hubicka.blogspot.com/2016/03/building-libreoffice-with-gcc-6-and-lto.html), [GCC 8](https://hubicka.blogspot.com/2018/12/even-more-fun-with-building-and.html), [more about GCC 8](https://hubicka.blogspot.com/2018/06/gcc-8-link-time-and-interprocedural.html), [GCC 9](https://hubicka.blogspot.com/2019/05/gcc-9-link-time-and-inter-procedural.html)
