---
title: Build heka on Windows
date: 2015-03-17
comments: true
---

Since there is currently no public release of [heka](https://github.com/mozilla-services/heka) 0.9 available for windows, the only way is to build it.


First of all, you need to install the compiler chain:

* [TDM-GCC MinGW Compiler](http://sourceforge.net/projects/tdm-gcc/?source=typ_redirect)
* [CMake](http://www.cmake.org/download/)
* [Go](https://golang.org/dl/)

And some other stuff like utilities:

* [Git](http://git-scm.com/download/win) Make sure to select `Use on command line`
* [Mercurial](http://mercurial.selenic.com/wiki/Download#Windows)

Pay attention to installer options, path must be set for all these binaries.

# Getting heka

    git clone https://github.com/mozilla-services/heka.git

Note: You can not use a source release from github releases, a .git directory must be included!

# Building

Get inside the `heka` directory, open a Command prompt.

    build.bat

If you've done anything right, there's a `[hekaroot]/build/heka/bin/hekad.exe` now.

# Running

To run `hekad`, it needs to find `liblua.dll` which is located at `[hekaroot]/build/heka/lib`. You can simply copy everything from `lib/` to your `bin/` directory to solve this problem.