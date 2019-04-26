---
layout: post
title:  GNU Build System
date:   2019-02-01 11:02:00 +0800
tags: [GNU, Autotools]
---

![GNU logo](https://upload.wikimedia.org/wikipedia/en/thumb/2/22/Heckert_GNU_white.svg/440px-Heckert_GNU_white.svg.png)

[WIKI: GNU Build System](https://en.wikipedia.org/wiki/GNU_Build_System)

> The GNU Build System, also known as the `Autotools`, is a suite of programming tools designed to assist in making source code packages portable to many Unix-like systems.

It can be difficult to make a software program portable: the C compiler differs from system to system; certain library functions are missing on some systems; header files may have different names. One way to handle this is to write conditional code, with code blocks selected by means of preprocessor directives (#ifdef); but because of the wide variety of build environments this approach quickly becomes unmanageable. **Autotools is designed to address this problem more manageably**.

Autotools is part of the GNU toolchain and is widely used in many free software and open source packages. The GNU Build System makes it possible to build many programs using **a two-step process**: 

> **configure** followed by **make**

Autotools consists of the GNU utility programs `Autoconf`, `Automake`, and `Libtool`. Other related tools frequently used alongside it include GNU's `make` program, GNU `gettext`, `pkg-config`, and the `GNU Compiler Collection`, also called `GCC`.

![Flow diagram of autoconf and automake](https://upload.wikimedia.org/wikipedia/commons/thumb/8/84/Autoconf-automake-process.svg/800px-Autoconf-automake-process.svg.png)

主要工具：

- autoscan
- autoheader
- aclocal
- autoconf
- automake
- make
- gettext
- pkg-config
- gcc

### autoscan

The autoscan program can help you create and/or maintain a `configure.ac` file for a software package. autoscan examines source files in the directory tree rooted at a directory given as a command line argument, or the current directory if none is given. It searches the source files for common portability problems and creates a file `configure.scan` which is a preliminary configure.ac for that package, and checks a possibly existing configure.ac for completeness.

When using autoscan to create a configure.ac, you should **manually** examine configure.scan before renaming it to configure.ac; it probably needs some adjustments. Occasionally, autoscan outputs a macro in the wrong order relative to another macro, so that autoconf produces a warning; you need to move such macros manually. Also, if you want the package to use a configuration header file, you must add a call to AC_CONFIG_HEADERS (see Configuration Headers). You might also have to change or add some #if directives to your program in order to make it work with Autoconf .

When using autoscan to maintain a configure.ac, simply consider adding its suggestions. The file `autoscan.log` contains detailed information on why a macro is requested.

### autoheader

The autoheader program can create a template file of C **#define** statements for configure to use. It searches for the first invocation of AC_CONFIG_HEADERS in configure sources to determine the name of the template. (If the first call of AC_CONFIG_HEADERS specifies more than one input file name, autoheader uses the first one.)

It is recommended that only one input file is used. If you want to append a boilerplate code, it is preferable to use `AH_BOTTOM([#include <conf_post.h>])`. File conf_post.h is not processed during the configuration then, which make things clearer. Analogically, AH_TOP can be used to prepend a boilerplate code.

You might wonder why autoheader is needed: after all, why would configure need to “patch” a `config.h.in` to produce a `config.h` instead of just creating config.h from scratch? Well, when everything rocks, the answer is just that we are wasting our time maintaining autoheader: generating config.h directly is all that is needed. When things go wrong, however, you'll be thankful for the existence of autoheader.

### aclocal

### autoconf

### automake

### make

### gettext

### pkg-config

### gcc
