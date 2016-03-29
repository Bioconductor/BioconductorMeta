# Building Bioconductor packages on gcc-4.9.3-based toolchain

## Overview

If all goes well, R-3.3.0 will be released using a [new toolchain](https://github.com/rwinlib/r-base) based on gcc/g++ 4.9.3.

As of [the last time this document was updated](https://gist.github.com/dtenenba/c15ed227d6b6df46949c/revisions),
CRAN is not building Windows binaries packages using the new toolchain. (`bin/contrib/windows/3.4` is a symlink
to `bin/contrib/wndows/3.3` which contains packages built by the old toolchain).

Bioconductor is going to attempt to build its devel packages against this new toolchain.
(Note that we will **not** use R-devel for this, we will use R-3.3.0-alpha)

There is a **binary incompatibility** between packages built with the old toolchain and packages
built with the new one. More precisely, packages are only incompatible if they have C++ code
(or bindings) in them. If a package has a system dependency containing C++ code, that dependency
must also be rebuilt, and many of them have--see below for pointers to them.
