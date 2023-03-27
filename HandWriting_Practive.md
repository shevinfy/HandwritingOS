# 实践：

参考文章：

vmware虚拟机安装ubuntu：[ubuntu20.04安装教程,ubuntu详细安装教程20.04 - ubuntu安装配置教程 - 博客园 (cnblogs.com)](https://www.cnblogs.com/ubuntuanzhuang/p/ubuntu2004.html)

[从零实现操作系统-手把手教你搭建环境 - 腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1872346)

## 构建纯粹的GCC交叉编译套件（i686-elf）：

### GCC一条龙构建脚本：

从osdev官网教程直接拿下来的。

osdev官网（强力推荐）：[Expanded Main Page - OSDev Wiki](https://wiki.osdev.org/Main_Page)

```shell
#! /usr/bin/bash
sudo apt update &&\
     apt install -y \
	build-essential \
	bison\
	flex\
	libgmp3-dev\
	libmpc-dev\
	libmpfr-dev\
	texinfo

BINUTIL_VERSION=2.37
BINUTIL_URL=https://ftp.gnu.org/gnu/binutils/binutils-2.37.tar.xz

GCC_VERSION=11.2.0
GCC_URL=https://ftp.gnu.org/gnu/gcc/gcc-11.2.0/gcc-11.2.0.tar.xz

GCC_SRC="gcc-${GCC_VERSION}"
BINUTIL_SRC="binutils-${BINUTIL_VERSION}"

# download gcc & binutil src code

export PREFIX="$HOME/cross-compiler"
export TARGET=i686-elf
export PATH="$PREFIX/bin:$PATH"

mkdir -p "${PREFIX}"
mkdir -p "${HOME}/toolchain/binutils-build"
mkdir -p "${HOME}/toolchain/gcc-build"

cd "${HOME}/toolchain"

if [ ! -d "${HOME}/toolchain/${GCC_SRC}" ]
then
	(wget -O "${GCC_SRC}.tar" ${GCC_URL} \
		&& tar -xf "${GCC_SRC}.tar") || exit
	rm -f "${GCC_SRC}.tar"
else
	echo "skip downloading gcc"
fi

if [ ! -d "${HOME}/toolchain/${BINUTIL_SRC}" ]
then
	(wget -O "${BINUTIL_SRC}.tar" ${BINUTIL_URL} \
		&& tar -xf "${BINUTIL_SRC}.tar") || exit
	rm -f "${BINUTIL_SRC}.tar"
else
	echo "skip downloading binutils"
fi

echo "Building binutils"

cd "${HOME}/toolchain/binutils-build"

("${HOME}/toolchain/${BINUTIL_SRC}/configure" --target=$TARGET --prefix="$PREFIX" \
	--with-sysroot --disable-nls --disable-werror) || exit

(make && make install) || exit

echo "Binutils build successfully!"

echo "Building GCC"

cd "${HOME}/toolchain/gcc-build"

which -- "$TARGET-as" || echo "$TARGET-as is not in the PATH"

("${HOME}/toolchain/${GCC_SRC}/configure" --target=$TARGET --prefix="$PREFIX" \
	--disable-nls --enable-languages=c,c++ --without-headers) || exit

(make all-gcc &&\
 make all-target-libgcc &&\
 make install-gcc &&\
 make install-target-libgcc) || exit

echo "done"
```

--disable-nls：所有错误英文报告。（编译的时候有错误可以直接去google去查）

编译大概半小时。

## 构建/安装Bochs（推荐源码安装）：

bochs官网：[bochs: The Open Source IA-32 Emulation Project (Home Page) (sourceforge.io)](https://bochs.sourceforge.io/)

```shell
export CC=gcc
export CXX="g++"
export CFLAGS="-Wall -O2 -fomit-frame-pointer -pipe"
export CXXFLAGS="$CFLAGS"

(./configure --enable-smp \
              --enable-cpu-level=6 \
              --enable-all-optimizations \
              --enable-x86-64 \
              --enable-pci \
              --enable-vmx \
              --enable-debugger \
              --enable-disasm \
              --enable-debugger-gui \
              --enable-logging \
              --enable-fpu \
              --enable-3dnow \
              --enable-sb16=dummy \
              --enable-cdrom \
              --enable-x86-debugger \
              --enable-iodebug \
              --disable-plugins \
              --disable-docbook \
              --with-x --with-x11 --with-term --with-sdl2) || exit

make && make install
```

./configure里的内容让我们的dubug体验更加好点

sdl2需要手动安装

## 文件目录：

```shell
mkdir arch kernel includes hal libs
# arch 平台/架构相关代码
	# 在arch中：x86的平台相关
# kernel 所有内核相关代码
# includes 内核头文件
# hal 硬件抽象层
# libs 一些常用库实现
```

