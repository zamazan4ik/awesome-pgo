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

Links:

* https://convolv.es/guides/lto/
