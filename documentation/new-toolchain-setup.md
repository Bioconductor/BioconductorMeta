# Building Bioconductor packages on gcc-4.9.3-based toolchain

## Overview

If all goes well, R-3.3.0 will be released using a [new toolchain](https://github.com/rwinlib/r-base) based on gcc/g++ 4.9.3.

As of [the last time this document was updated](https://github.com/Bioconductor/Bioconductor/commits/master/documentation/new-toolchain-setup.md),
CRAN is not building Windows binaries packages using the new toolchain. (`bin/contrib/windows/3.4` is a symlink
to `bin/contrib/wndows/3.3` which contains packages built by the old toolchain).

Bioconductor is going to attempt to build its devel packages against this new toolchain.
(Note that we will **not** use R-devel for this, we will use R-3.3.0-alpha)

There is a **binary incompatibility** between packages built with the old toolchain and packages
built with the new one. More precisely, packages are only incompatible if they have C++ code
(or bindings) in them. If a package has a system dependency containing C++ code, that dependency
must also be rebuilt, and many of them have--see below for pointers to them.

## Resources

* [New Toolchain](https://github.com/rwinlib/r-base)
* [Libraries needed to build CRAN packages](http://www.stats.ox.ac.uk/pub/Rtools/libs.html)
* [New Builds of System Dependencies](http://www.stat.ucla.edu/~jeroen/mingw-w64/libraries/)
* [Results of unofficial CRAN builds on new toolchain](https://docs.google.com/spreadsheets/d/1XNcw0NexzVkwc4uiZoqJKgxPoO4ZGlXtw6jHr1FAaf4/edit?usp=sharing) (from Jim Hester)
* [Third-Party Software for CRAN on Windows](https://cran.r-project.org/bin/windows/contrib/ThirdPartySoftware.html) (variables that need to be set in $R_ARCH\\Makeconf)

## Setting up R to use the new Toolchain

* Install [Rtools33](https://cran.r-project.org/bin/windows/Rtools/Rtools33.exe). Be sure
  and install it in c:\Rtools.
  Make sure your `PATH` **starts** with:
  `c:/Rtools/bin;C:/Rtools/mingw_32/bin;C:/Rtools/mingw_64/bin;`
* Install [R-3.3.0](https://cran.r-project.org/bin/windows/base/rtest.html)
  Add the following lines to `$_HOME\etc\Rprofile.site`:

```
Sys.setenv(BINPREF = "C:/Rtools/mingw_$(WIN)/bin/")
options(pkgType="source")
```

*  This tells R to use the newer compilers, and disables binary
  package installs (since the binaries currently available were built
  with the old toolchain).

* In **both** `Makeconf` files (`etc/i386/Makeconf` and `etc/x64/Makeconf`) change
  this line:

```
BINPREF ?= c:/Rtools/mingw_32/bin/
```

to

```
BINPREF = c:/Rtools/mingw_32/bin/
```

In other words, just remove the `?`.

## Setting up system Dependencies

You will need to set up a number of system dependencies, since you have to build
many common CRAN packages (such as `XML` and `RCurl`) from source, as CRAN
is not yet building these using the new toolchain. You'll also need to install
from source the system dependencies of Bioconductor packages.

For each dependency you install, you will need to add or modify an entry
to `$R_HOME\etc\i386\Makeconf` and `$R_HOME\etc\x64\Makeconf`. *You must
modify both files*. The instructions below will discuss how to install each system
dependency, as well as the entries you need to make or change in the `Makeconf`
files.

### `NM_FILTER`

Make sure the `NM_FILTER` line in `etc/$R_HOME/Makeconf` looks like this:

```
NM_FILTER = | sed -e '/\.refptr\./d' -e '/\.weak\./d'
```

Otherwise packages that use igraph may fail. See [here](https://github.com/RcppCore/Rcpp/issues/442#issuecomment-190848342)
for background.


### Local Tree

The headers and libraries for a number of system dependencies
(XML, gsl, curl, etc.) are
made available at the [External Software](http://www.stats.ox.ac.uk/pub/Rtools/libs.html)
page maintained by Brian Ripley.

* Download [local323.zip](http://www.stats.ox.ac.uk/pub/Rtools/goodies/multilib/local323.zip)
* Unzip it into `c:\local323`.
* Download [spatial324.zip](http://www.stats.ox.ac.uk/pub/Rtools/goodies/multilib/spatial324.zip)
* Also unzip it into `c:\local323`.
* Add the following lines to **both** Makeconf files (or **modify** the these settings if they already exist):

```
LOCAL_SOFT = c:/local323
LIB_XML = c:/local323
NETCDF = c:/local323
LIB_GSL = c:/local323
```

### HDF5

* Clone the [H5 git repository](https://github.com/mpastell/h5-libwin) to
  `c:\h5-libwin`
* Add the following to **both** Makeconf files:

```
LIB_HDF5 = c:/h5-libwin
```

### Boost

* Download [boost-1.57.zip](http://www.stat.ucla.edu/~jeroen/mingw-w64/libraries/boost-1.57.zip)
* Unzip it in `c:\boost_1-57`
* Add the following to **both** Makeconf files:

```
BOOSTLIB = c:/boost_1-57
```

### JAGS

* Install [JAGS-4.2.0.exe](
https://sourceforge.net/projects/mcmc-jags/files/JAGS/4.x/Windows/JAGS-4.2.0-Rtools33.exe/download)
  (this is a special version built for the new toolchain)
* Add the following to **both** Makeconf files:

```
JAGS_ROOT = c:/progra~1/JAGS/JAGS-4.2.0
```



### Protocol Buffers

* Download [protobuf-2.6.1](http://www.stat.ucla.edu/~jeroen/mingw-w64/libraries/protobuf-2.6.1.zip)
* Unzip it in `c:\protobuf-2.6.1-newtoolchain`
* Some packages expect this library to be laid out differently, so do:

```
c:
cd \protobuf-2.6.1-newtoolchain
mkdir -p i386/lib x64/lib i386/include x64/include
cp -r include/* i386/include/
cp -r include/* x64/include/
cp -r lib/i386/* i386/lib/
cp -r lib/x64/* x64/lib/

```

* Add the following to **both** Makeconf files:

```
LIB_PROTOBUF = C:/protobuf-2.6.1-newtoolchain
```


### NLopt

* Download [nlopt-2.4.2.zip](http://www.stat.ucla.edu/~jeroen/mingw-w64/libraries/nlopt-2.4.2.zip)
* Unzip it in `c:\nlopt\2.4.2`
* Some packages expect this library to be laid out differently, so do:

```
c:
cd \nlopt\2.4.2
mkdir -p i386/lib x64/lib i386/include x64/include
cp -r include/* i386/include/
cp -r include/* x64/include/
cp -r lib/i386/* i386/lib/
cp -r lib/x64/* x64/lib/
```

* Add the following to **both** Makeconf files:

```
NLOPT_HOME = c:/nlopt/2.4.2
```

### System Dependencies not yet updated

We hope to have these resolved the week of April 4.

* Gtk2
* ggobi
* MPI

## Changes made to packages to faciliate installation

* [flowWorkspace](https://github.com/Bioconductor-mirror/flowWorkspace/commit/70d2e35da63323dc352f519732255aee24f04b8c)

## Packages that still do not install

### GeneNetworkBuilder (and others)

R crashes (not visible in build report)

### mzR

Updated the embedded libnetcdf.a files. Also need to update libpwiz.a.
Seems like running `make -f Makefile.libpwiz` in `mzR\src` does this,
but I am not sure if the resulting .a file is for i386 or x64, or
how to build the other sub-architecture. Need to ask maintainer.
See [this question](https://github.com/sneumann/mzR/issues/21#issuecomment-203683171).

### plrs

CRAN package Rcsdp cannot be installed:

```
C:/Rtools/mingw_64/bin/gcc -shared -s -static-libgcc -o Rcsdp.dll tmp.def Rcsdp.o convert.
o testing.o -LCsdp/lib -lsdp -Le:/biocbld/BBS-3~1.3-B/R/bin/x64 -lRlapack -Le:/biocbld/BBS
-3~1.3-B/R/bin/x64 -lRblas -lgfortran -lm -lquadmath -lm -Lc:/local323/lib/x64 -Lc:/local3
23/lib -Le:/biocbld/BBS-3~1.3-B/R/bin/x64 -lR
Csdp/lib/libsdp.a: error adding symbols: Archive has no index; run ranlib to add one
collect2.exe: error: ld returned 1 exit status
no DLL was created
```

### Rdisop

Solved by installing RcppClassic **without** the `--merge-multiarch` flag.

### Rhtslib (affects bamsignals, csaw and deepSNV)

Rhtslib actually installs OK but (we think) the static htslib libraries
inside it need to be rebuilt after commenting out
[this line](https://github.com/Bioconductor-mirror/Rhtslib/blame/master/src/htslib/sam.c#L1078).

### Rbowtie

Seems to be using a non-standard Makefile. It needs to   be updated to use the `BINPREF` make variable to find the appropriate compiler.
Tried  commenting out the CC and CXX > variables but that didn't work.


### xps

Not sure, maybe need to rebuild ROOT? 
(Package has been marked as unsupported on Windows at the request of the maintainer; building ROOT from source
using the toolchain is a massive undertaking.)


