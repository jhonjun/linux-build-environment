# linux-build-environment
I've been looking for a generic Linux build environment that I can use to build binaries that can be deployed to most distros out there. So far, I've only found build environments for specific projects.
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
Note: You may need sudo or switch to root in some of the commands and replace some of the absolute paths to your paths.

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

mkdir /usr/local/root/gcc
chown -R builder /usr/local
su - builder

cd /usr/local/src
svn co svn://gcc.gnu.org/svn/gcc/tags/gcc_4_8_5_release gcc
cd gcc
mkdir build ; cd build

../configure --prefix=/usr/local/root/gcc --enable-bootstrap --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-linker-hash-style=gnu --enable-languages=c,c++ --enable-plugin --enable-initfini-array --disable-libgcj --enable-gnu-indirect-function --with-tune=generic --disable-multilib

make
make install

export PATH=/usr/local/root/gcc/bin:$PATH
export LIBRARY_PATH=/usr/local/root/gcc/lib64:/usr/lib64:/lib64:/usr/local/lib64
export LD_LIBRARY_PATH=$LIBRARY_PATH

```

## Maintaining Binary Compatibility in the Future when Using C++
At some point in time, depending on your distro and your priorities, your C++ binary will break compatibility. Details can be found here: https://developers.redhat.com/blog/2015/02/05/gcc5-and-the-c11-abi/.

If your target environment was built with the new ABI (and GCC 5 and above), you will need to declare -D_GLIBCXX_USE_CXX11_ABI in your build. Otherwise, if your priority is supporting *older* targets, build with -D_GLIBCXX_USE_CXX11_ABI=0 flag.

## Using Clang in lieu of GCC
To maintain 100% binary compatibility, use GNU's C++ library (libstdc++.so.6) that comes with GCC. You can force clang++ to do this by passing the -stdlib=libstdc++ flag.

## Bonus: Building Clang

```
git clone https://github.com/llvm-mirror/llvm.git
git clone https://github.com/llvm-mirror/clang.git
git clone https://github.com/llvm-mirror/clang-tools-extra.git

cd llvm
git checkout release_39
cd ../clang
git checkout release_39
cd ../clang-tools-extra
git checkout release_39

cd llvm/tools
ln -s ../../clang .
cd ../..
cd clang/tools
ln -s ../../clang-tools-extra extra
cd ../..

mkdir clang-build
cd clang-build

yum install python34
# update the python symbolic link to point to python3

CC=`which gcc` CXX=`which g++` cmake -DCMAKE_INSTALL_PREFIX=/usr/local/root/clang -DLLVM_ENABLE_PROJECTS=clang -DLLVM_TARGETS_TO_BUILD=X86 -DCMAKE_BUILD_TYPE=Release ../llvm

make
make install

# Add /usr/local/root/clang/bin to your PATH
```
