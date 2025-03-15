---
title: CFD常用库配置：FFTW & Hypre
subtitle: 
layout: post
author: "Ling"
header-style: text
tags:
  - CFD
  - Hypre
  - Linux
  - FFTW
  - Fortran
---

# 一、[FFTW Download](https://fftw.org/download.html)下载与安装

FFTW is a C subroutine library for computing the discrete Fourier transform (DFT) in one or more dimensions, of arbitrary input size, and of both real and complex data (as well as of even/odd data, i.e. the discrete cosine/sine transforms or DCT/DST). We believe that FFTW, which is [free software](https://fftw.org/faq/section1.html#isfftwfree), should become the [FFT](http://en.wikipedia.org/wiki/Fast_Fourier_transform) library of choice for most applications.
FFTW 是 C 用于计算离散傅里叶变换 （DFT） 的子例程库 在一个或多个维度中，具有任意输入大小，并且都是实数 和复杂数据（以及偶数/奇数数据，即离散数据 余弦/正弦变换或 DCT/DST）。我们认为 FFTW、 这是[自由软件](https://fftw.org/faq/section1.html#isfftwfree)，应该成为大多数人的首选 [FFT](http://en.wikipedia.org/wiki/Fast_Fourier_transform) 库 应用。

The latest official release of FFTW is version **3.3.10**, available from [our download page](https://fftw.org/download.html). Version 3.3 introduced support for the AVX x86 extensions, a distributed-memory implementation on top of MPI, and a Fortran 2003 API. Version 3.3.1 introduced support for the ARM Neon extensions. See the [release notes](https://fftw.org/release-notes.html) for more information.
FFTW 的最新正式版本是 **3.3.10** 版，可从[我们的下载页面](https://fftw.org/download.html)获取。版本 3.3 引入了对 AVX x86 扩展的支持、基于 MPI 的分布式内存实现以及 Fortran 2003 API。版本 3.3.1 引入了对 ARM Neon 扩展的支持。有关详细信息，请参阅[发行说明](https://fftw.org/release-notes.html)。

The FFTW package was developed at [MIT](http://web.mit.edu/) by [Matteo Frigo](http://www.fftw.org/~athena/) and [Steven G. Johnson](http://math.mit.edu/~stevenj/).
FFTW 包由 [Matteo Frigo](http://www.fftw.org/~athena/) 和 [Steven G. Johnson](http://math.mit.edu/~stevenj/) 在 [MIT](http://web.mit.edu/) 开发。

Our [benchmarks](http://www.fftw.org/benchfft/), performed on on a variety of platforms, show that FFTW's performance is typically superior to that of other publicly available FFT software, and is even competitive with vendor-tuned codes. In contrast to vendor-tuned codes, however, FFTW's performance is *portable*: the same program will perform well on most architectures without modification. Hence the name, "FFTW," which stands for the somewhat whimsical title of "**Fastest Fourier Transform in the West**."
我们的[基准测试](http://www.fftw.org/benchfft/)， 在各种平台上演出，表明 FFTW 的表现 通常优于其他公开可用的 FFT 软件，甚至与供应商调整的代码竞争。 在 然而，与供应商调整的代码相比，FFTW 的性能是 *可移植*：相同的程序无需修改即可在大多数架构上表现良好。因此得名“FFTW”，它代表着“**西方最快的傅里叶变换**”这个有点异想天开的标题。

下载之后进行解压安装：

```bash
tar -zxvf fftw-3.3.10
cd fftw-3.3.10
mkdir build
cd build
./configure --prefix=/opt/fftw
make -j
sudo make install
```

需要特别注意的是，默认一般使用gcc和gfortran编译器。

如果需要使用intel编译器icc和ifx（ifort），最好在编译前强制指定：

```bash
export CC=icc
export CXX=icpc
export F77=ifort
export F90=ifort
export FC=ifort
```

# 二、[hypre --- Releases](https://github.com/hypre-space/hypre/releases)下载与安装

[HYPRE](http://www.llnl.gov/casc/hypre/) is a library of high performance preconditioners and solvers featuring multigrid methods for the solution of large, sparse linear systems of equations on massively parallel computers.
[HYPRE](http://www.llnl.gov/casc/hypre/) 是一个高性能预条件器和求解器库，具有多网格方法，用于在大规模并行计算机上求解大型稀疏线性方程组。

For documentation, see our [readthedocs page](https://hypre.readthedocs.io/en/latest/).
有关文档，请参阅我们的 [readthedocs 页面](https://hypre.readthedocs.io/en/latest/)。

For information on code development, build requirements, publications, and more, see our [Wiki page](https://github.com/hypre-space/hypre/wiki).
有关代码开发、构建要求、出版物等的信息，请参阅我们的 [Wiki 页面](https://github.com/hypre-space/hypre/wiki)。

To install HYPRE, please see either the documentation or the file [INSTALL.md](https://github.com/hypre-space/hypre/blob/master/INSTALL.md).
要安装 HYPRE，请参阅文档或文件 [INSTALL.md](https://github.com/hypre-space/hypre/blob/master/INSTALL.md)。

An overview of the HYPRE release history can be found in the file [CHANGELOG](https://github.com/hypre-space/hypre/blob/master/CHANGELOG).
HYPRE 发行历史的概述可以在文件 [CHANGELOG](https://github.com/hypre-space/hypre/blob/master/CHANGELOG) 中找到。

Support information can be found in the file [SUPPORT.md](https://github.com/hypre-space/hypre/blob/master/SUPPORT.md).
支持信息可以在 file [SUPPORT.md](https://github.com/hypre-space/hypre/blob/master/SUPPORT.md) 中找到。

下载之后进行解压安装，在src文件夹中：

```bash
tar -zxvf hypre-2.32.0
cd hypre-2.32.0
cd src
mkdir build
cd build
../configure --prefix=/opt/hypre  #路径有时候设置不生效
cmake ../
```

hypre安装需要cmake ../，且安装路径有时候设置不生效，需要打开**CMakeCache.txt**文件，搜索**PREFIX**。一共有3处，修改为要安装的路径（/opt/hypre）。

继续完成编译和安装。

```bash
make -j
sudo make install
```

需要特别注意的是，hypre默认使用并行编译器：

```bash
export CC=mpicc
export CXX=mpicxx
export F77=mpif90
export F90=mpif90
export FC=mpif90
```
