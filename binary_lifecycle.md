# PGO: where to apply?

Here I want to discuss software delivary chains and places, where we can apply PGO. I think this information could help with communicating with the right persons to apply PGO for each project.

At first, we can divide software into the two categories: open-source and proprietary.

## Open-source

People usually consumes open-source projects in the following ways:

1. By using binaries from their favourite repositories (usually in Linux-based OS, more rare in macOS and Windows)
2. By using binaries provided by developers of the project (like prebuilt from a site)
3. Build binaries in their own infra (like company-wide internal repos)

The most efficient from the human resources point of view will be PGO integration into the source repository. Usually it's as a build option or extra scripts. This way allows using the same scripts (or with a few changes) for all use-cases: building PGO'ed binaries locally, by project developers itself or using by maintainers for building package scripts. That's why I recommend interact with upstream projects and ask them to implement PGO scripts into the sources.

If you are building all binaries on your own - you are almost done here. Use upstream PGO scripts, build locally and enjoy PGO.

In case of externally-built binaries, things become more complicated. Even if developers have implemented PGO into their project (like there are PGO build mode in their build scripts) that does not mean a binary from their site is PGO-optimized. The only way to check this - ask developers. And if it isn't - ask them to add support (or at least mention somewhere in the documentation about it). 

With binaries maintained by someone the situation is similar. The only way is to check build scripts of your package (if you able to do it). Usually, will be easier to ask a maintainer of a package about it (directly, on a forum, somewhere else). And if the project already supports PGO build mode - ask them about enabling it. Once again - it would be much easier to do if the upstream already supports building in PGO mode.

There are more caveats about enabling PGO:
* Multiple builds, so increased load on the build pipelines
* Reproducibility questions. Some distros prefer reproducibility over performance (not to blame at all - just a fact)

How to overcome these issues - hugely depends on the situation: distro/maintainer/anything else.

## Proprietary

Well, here is the only way - ask your vendor explicitly about PGO'ing their products, if you care about performance. They are also humans (I hope) and could not know about this fancy optimization. And maybe one day they will provide you a new PGO-optimized binary.

If we are talking about backend services or something like that... Well, maybe one day PGO will as usual as using -O3 :)
