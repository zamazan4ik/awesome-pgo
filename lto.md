# LTO state

LTO is not widely used in the distributions. However, more and more OS enables LTo by default for their packages. Is it enabled by default on your distribution - ask your maintainers.

Is LTO enabled in other binaries - ask their authors. Do not forget that enabling LTO could uncover some bugs: [GentooLTO](https://github.com/InBetweenNames/gentooLTO/issues).

State across distrubutions:

* Debian: [link](https://wiki.debian.org/ToolChain/LTO)
* Ubuntu: [link](https://wiki.ubuntu.com/ToolChain/LTO)
* Fedora: [link](https://fedoraproject.org/wiki/LTOByDefault)
* OpenSUSE: [link](https://en.opensuse.org/openSUSE:LTO)
* NixOS: [Reddit](https://www.reddit.com/r/NixOS/comments/146wdfk/lto_by_default/)
* Gentoo: TODO

Performance improvments from enabling LTO:
* Vector: https://github.com/vectordotdev/vector/issues/15631#issuecomment-1694554798
