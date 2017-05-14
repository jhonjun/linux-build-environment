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

I have also chosen GCC 4.8.x as my base C++ compiler because it has the most complete C++11 support (https://gcc.gnu.org/gcc-4.8/cxx0x_status.html) before the new ABI was introduced in GCC 5.1.

## Building the environment
Note: You may need sudo or switch to root in some of the commands.

```
lxc-create -t download -n centos6
mount --bind /dev /home/jjd/.local/share/lxc/centos6/rootfs/dev
mount --bind /dev/pts /home/jjd/.local/share/lxc/centos6/rootfs/dev/pts
mount --bind /proc /home/jjd/.local/share/lxc/centos6/rootfs/proc

chroot /home/jjd/.local/share/lxc/centos6/rootfs
passwd

yum install epel-release
yum install which tar perl

yum install gcc-c++ zlib-devel
yum install gmp-devel mpfr-devel libmpc-devel cloog-ppl-devel
yum install autoconf automake gettext gperf dejagnu guile flex texinfo texinfo-tex subversion patch

adduser builder
passwd builder
su - builder

cd /usr/local/src
svn co svn://gcc.gnu.org/svn/gcc/tags/gcc_4_8_5_release gcc
cd gcc
mkdir build ; cd build

../configure --prefix=/usr/local/root/gcc --enable-bootstrap --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-linker-hash-style=gnu --enable-languages=c,c++ --enable-plugin --enable-initfini-array --disable-libgcj --enable-gnu-indirect-function --with-tune=generic --disable-multilib

make
make install
```
