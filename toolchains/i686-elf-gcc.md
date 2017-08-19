# i686-elf-gcc Toolchain

## But, Why?
I use this toolchain for some of my hobby kernel projects. It's fairly easy to set up, and writing for it is fairly easy as well.

## Dependancy Tree
```
GCC
├ Binutils
│ ├ ISL
│ │ └ GMP
│ └ GLooG
│   └ GMP
├ MPC
│ └ MPFR
│   └ GMP
└ ISL
  └ GMP
```

The dependancy tree is pretty simple for this one.

## Compiling all this junk

This will compile everything into a `tools` folder in your home folder.

```Bash
export GMP_VERSION=6.1.2
export MPFR_VERSION=3.1.5
export MPC_VERSION=1.0.3
export ISL_VERSION=0.18
export CLOOG_VERSION=0.18.4
export BINUTILS_VERSION=2.28.1
export GCC_VERSION=7.1.0

export GNU_MIRROR_BASE=https://ftp.gnu.org/gnu
export ISL_MIRROR_BASE=http://isl.gforge.inria.fr
export CLOOG_MIRROR_BASE=http://www.bastoul.net/cloog/pages/download

export PATH=$HOME/tools:$PATH

function download_compile {
	curl $1 > $2.archive
	tar -xf $2.archive
	cd $2
	./configure --prefix=$HOME/tools $3
	make -j8
	make install
	cd ..
	rm -rf $2.archive
	rm -rf $2
}

set -e
download_compile $GNU_MIRROR_BASE/gmp/gmp-$GMP_VERSION.tar.bz2 gmp-$GMP_VERSION
download_compile $GNU_MIRROR_BASE/mpfr/mpfr-$MPFR_VERSION.tar.bz2 mpfr-$MPFR_VERSION "--with-gmp=$HOME/tools"
download_compile $GNU_MIRROR_BASE/mpc/mpc-$MPC_VERSION.tar.gz mpc-$MPC_VERSION "--with-gmp=$HOME/tools --with-mpfr=$HOME/tools"
download_compile $ISL_MIRROR_BASE/isl-$ISL_VERSION.tar.gz isl-$ISL_VERSION "--with-gmp-prefix=$HOME/tools"
download_compile $CLOOG_MIRROR_BASE/cloog-$CLOOG_VERSION.tar.gz cloog-$CLOOG_VERSION "--with-gmp-prefix=$HOME/tools --with-isl-prefix=$HOME/tools"
download_compile $GNU_MIRROR_BASE/binutils/binutils-$BINUTILS_VERSION.tar.gz binutils-$BINUTILS_VERSION "--with-isl=$HOME/tools --with-cloog=$HOME/tools --with-sysroot --disable-nls --disable-werror --target=i686-elf"
download_compile $GNU_MIRROR_BASE/gcc/gcc-$GCC_VERSION/gcc-$GCC_VERSION.tar.gz gcc-$GCC_VERSION "--with-isl=$HOME/tools --with-cloog=$HOME/tools --with-gmp=$HOME/tools --with-mpfr=$HOME/tools --with-mpc=$HOME/tools --disable-nls --enable-languages=c,c++ --without-headers --target=i686-elf"
```
