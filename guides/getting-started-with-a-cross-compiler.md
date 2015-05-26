---
title: Getting Started With A Cross Compiler
author: wxwsk8er
categories: [c, libraries, kernel]

template: page.toffee
---

# Cross-Compiler Step-by-Step Setup

This article is all about the GCC cross compiler and how to get you started with it on 
[Koding](https://koding.com).

## Cross Complier: What's it all about.

A cross compiler is used when developing an operating system, or cross-archicture software in architecture-specific languages like C.
For instance, suppose I was using [Koding's](https://koding.com) VM as my source hoster, but then I used another home based PC for testing,
how would I compile the code on the x86_64 architecture, then run it on i686 architecture? Well a cross compiler does just that. It uses
binaries (programs) on the host system to build code into a non-native architecture. Cool right?

## Getting Started

First we need to free up some space. The cross compiler has a huge source code base and I recommend that you have at least 4GB free. The final compiled
binaries take a little over 110MB so allocating as much memory to one VM as possible is suggested (You can adjust allocated disk space from
VM->Disk Usage->(bottom)Resize your VM).

A usefull way to free up memory is to use the following commands:

    sudo rm -r /tmp/*
    sudo apt-get clean
    sudo rm -r ~/.bash_history

Now our cross compiler is going to be located at `/opt/cross` so we need to relay that, and the target achitecture we are building this compiler for to the
shell:

    sudo mkdir -p /opt/cross
    sudo chown [YOUR USERNAME] /opt/cross
    export PATH=/opt/cross/bin:$PATH
    export TARGET=i686-elf
    export PREFIX=/opt/cross

### Downloading sources

Now we need to actually download the source code for each package. We will do this using the `wget` command

    mkdir ~/build-cross
    cd ~/build-cross
    wget http://ftpmirror.gnu.org/binutils/binutils-2.24.tar.gz
    tar xf binutils-2.24.tar.gz
    rm -r binutils-2.24.tar.gz

That will make us a fancy directory, download the source code for binutils, then de-compress it, then delete the compressed
form to give us more space.

Now we actually need to build it:

    mkdir build-binutils
    cd build-binutils
    ../binutils-2.24/configure --prefix=$PREFIX --target=$TARGET --disable-multilib --with-sysroot -disable-werror
    make -j4
    make install
    cd ...
    sudo rm -r build-binutils
    sudo rm -r binutils-2.24

Let me explain. This makes a directory called `build-binutils`, then moves in there, then configures binutils for our target arch, then builds the
sources, then installs them to `/opt/cross` then switches back to our parent directory (`~/build-cross`).

Now we need to download all of GCC's components, get ready for some space hogging:

    wget http://ftpmirror.gnu.org/gcc/gcc-4.9.2/gcc-4.9.2.tar.gz
    wget http://ftpmirror.gnu.org/glibc/glibc-2.20.tar.xz
    wget http://ftpmirror.gnu.org/mpfr/mpfr-3.1.2.tar.xz
    wget http://ftpmirror.gnu.org/gmp/gmp-6.0.0a.tar.xz
    wget http://ftpmirror.gnu.org/mpc/mpc-1.0.2.tar.gz
    wget ftp://gcc.gnu.org/pub/gcc/infrastructure/isl-0.12.2.tar.bz2
    wget ftp://gcc.gnu.org/pub/gcc/infrastructure/cloog-0.18.1.tar.gz

Now we need to decompress all that (wil take a while):

    for f in *.tar*; do tar xf $f; done

Now we need to create some symlinks for GCC so we can compile all the sources:

    cd gcc-4.9.2
    ln -s ../mpfr-3.1.2 mpfr
    ln -s ../gmp-6.0.0 gmp
    ln -s ../mpc-1.0.2 mpc
    ln -s ../isl-0.12.2 isl
    ln -s ../cloog-0.18.1 cloog
    cd ..

Now we need to actually build GCC:

    mkdir build-gcc
    cd build-gcc
    ../gcc-4.9.2/configure --target=$TARGET --prefix=/opt/cross --disable-nls --enable-languages=c,c++ --without-headers --disable-multilib
    make all-gcc
    make all-target-libgcc
    make install-gcc
    make install-target-libgcc

So now if you followed directions, you should be able to invoke `i686-elf-gcc` or `/opt/cross/bin/i686-elf-gcc` correctly.
Enjoy your cross compiler, and for more OS-development related tutorials and fun check out http://wiki.osdev.org
  
Have a good day :)
