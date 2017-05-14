# linux-build-environment
I've been looking for a generic Linux build environment that I can use to build binaries that I can deploy to most distros out there. So far, I've only found build environments for specific projects.
There's Open Build Service by openSUSE but I want something simpler and more lightweight... the bare minimum.
This is my attempt to build such an environment.

Note that a development environment is different from a build environment. I use both. My development environment is based on Ubuntu 16.04 because I can easily find the tools that I need built for that system and if not, I can try to build it myself.

A build environment is something else. It's the environment that you use to build the release version of the software that you want to produce.

There's also a third environment: the environment where your software product is going to be used on or deployed to, e.g. RHEL 7 or CentOS 7.

The focus of this article is the second item, the build environment.

## Compatibility
The main thing to understand about Linux binaries on userland is that everything is linked with glibc, the GNU C library. Other dependencies can be found here: http://refspecs.linuxfoundation.org/LSB_4.1.0/LSB-Core-generic/LSB-Core-generic/baselib.html.

For C++, note here the important factor: https://gcc.gnu.org/onlinedocs/libstdc++/manual/using_dual_abi.html.

Based on the information here: http://www.linux-m68k.org/faq/glibcinfo.html, since most Linux distros are now using glibc based on libc.so.6, and due to this breaking bug: https://lists.gnu.org/archive/html/info-gnu/2007-08/msg00001.html, I looked for a stable distro with long-term support that has at least glibc 2.6.1 and found that CentOS 6.x is the closest thing that fit the criteria.

## Building the environment

