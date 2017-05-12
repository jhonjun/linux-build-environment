# linux-build-environment
I've been looking for a generic Linux build environment that I can use to build binaries that I can deploy to most distros out there. So far, I've only found build environments for specific projects.
There's Open Build Service by openSUSE but I want something simpler and more lightweight... the bare minimum.
This is my attempt to build such an environment.

Note that a development environment is different from a build environment. I use both. My development environment is based on Ubuntu 16.04 because I can easily find the tools that I need built for that system and if not, I can try to build it myself.

A build environment is something else. It's the environment that you use to build the release version of the software that you want to produce.

There's also a third environment: the environment where your software product is going to be used on or deployed to. e.g. RHEL 7 or CentOS 7.

The focus of this article is the second item, the build environment.

## Compatibility
The main thing to understand about Linux binaries on userland is that everything is linked with glibc, the GNU C library.
