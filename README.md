# linux-build-environment
I've been looking for a generic Linux build environment that I can use to build binaries that I can deploy to most distros out there. So far, I've only found build environments for specific projects.
There's Open Build Service by openSUSE but I want something simpler and more lightweight... the bare minimum.
This is my attempt to build such an environment.
## Compatibility
The main thing to understand about Linux binaries on userland is that everything is linked with glibc, the GNU C library.
