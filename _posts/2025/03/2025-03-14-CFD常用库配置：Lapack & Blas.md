---
title: CFD常用库配置：Lapack & Blas
subtitle: 
layout: post
author: "Ling"
header-style: text
tags:
  - CFD
  - Lapack
  - Linux
  - Blas
  - Fortran
---

# 一、[LAPACK — Linear Algebra PACKage](https://www.netlib.org/lapack/)下载与安装

LAPACK is written in Fortran 90 and provides routines for solving systems of simultaneous linear equations, least-squares solutions of linear systems of equations, eigenvalue problems, and singular value problems. The associated matrix factorizations (LU, Cholesky, QR, SVD, Schur, generalized Schur) are also provided, as are related computations such as reordering of the Schur factorizations and estimating condition numbers. Dense and banded matrices are handled, but not general sparse matrices. In all areas, similar functionality is provided for real and complex matrices, in both single and double precision.
LAPACK 是用 Fortran 90 编写的，提供了用于求解联立线性方程组、线性方程组最小二乘解、特征值问题和奇异值问题的例程。此外，还提供了相关的矩阵分解（LU、Cholesky、QR、SVD、Schur、广义 Schur）以及相关计算，例如 Schur 分解的重新排序和估计条件数。处理密集矩阵和带状矩阵，但不处理一般稀疏矩阵。在所有领域中，都为实数矩阵和复数矩阵提供了类似的功能，包括单精度和双精度。

The original goal of the LAPACK project was to make the widely used EISPACK and LINPACK libraries run efficiently on shared-memory vector and parallel processors. On these machines, LINPACK and EISPACK are inefficient because their memory access patterns disregard the multi-layered memory hierarchies of the machines, thereby spending too much time moving data instead of doing useful floating-point operations. LAPACK addresses this problem by reorganizing the algorithms to use block matrix operations, such as matrix multiplication, in the innermost loops. These block operations can be optimized for each architecture to account for the memory hierarchy, and so provide a transportable way to achieve high efficiency on diverse modern machines. We use the term "transportable" instead of "portable" because, for fastest possible performance, LAPACK requires that highly optimized block matrix operations be already implemented on each machine.
LAPACK 项目的最初目标是使广泛使用的 EISPACK 和 LINPACK 库在共享内存向量和并行处理器上高效运行。在这些机器上，LINPACK 和 EISPACK 效率低下，因为它们的内存访问模式忽略了机器的多层内存层次结构，因此花费了太多时间移动数据，而不是执行有用的浮点运算。LAPACK 通过重新组织算法以在最内层循环中使用块矩阵运算（例如矩阵乘法）来解决此问题。这些块作可以针对每个架构进行优化，以考虑内存层次结构，从而提供一种可传输的方式，以便在各种现代机器上实现高效率。我们使用术语“可传输”而不是“可移植”，因为为了获得尽可能快的性能，LAPACK 要求在每台机器上都已经实现了高度优化的块矩阵作。

LAPACK routines are written so that as much as possible of the computation is performed by calls to the Basic Linear Algebra Subprograms (BLAS). LAPACK is designed at the outset to exploit the Level 3 BLAS — a set of specifications for Fortran subprograms that do various types of matrix multiplication and the solution of triangular systems with multiple right-hand sides. Because of the coarse granularity of the Level 3 BLAS operations, their use promotes high efficiency on many high-performance computers, particularly if specially coded implementations are provided by the manufacturer.
编写 LAPACK 例程时，可以通过调用基本线性代数子程序 （BLAS） 来执行尽可能多的计算。LAPACK 从一开始就设计为利用 3 级 BLAS，这是一组用于 Fortran 子程序的规范，用于执行各种类型的矩阵乘法和具有多个右侧的三角系统的解。由于 3 级 BLAS 运算的粒度很粗糙，因此使用它们可以提高许多高性能计算机的高效率，尤其是在制造商提供特殊编码实现的情况下。

Highly efficient machine-specific implementations of the BLAS are available for many modern high-performance computers. For details of known vendor- or ISV-provided BLAS, consult the BLAS FAQ. Alternatively, the user can download ATLAS to automatically generate an optimized BLAS library for the architecture. A Fortran 77 reference implementation of the BLAS is available from netlib; however, its use is discouraged as it will not perform as well as a specifically tuned implementation.
许多现代高性能计算机都提供了 BLAS 的高效机器特定实现。有关已知供应商或 ISV 提供的 BLAS 的详细信息，请参阅 BLAS 常见问题解答。或者，用户可以下载 ATLAS 以自动生成为架构优化的 BLAS 库。BLAS 的 Fortran 77 参考实现可从 netlib 获得;但是，不鼓励使用它，因为它的性能不如专门调整的实现。



下载之后进行解压安装，在install文件夹中，有多种已经配置好的make.inc，拷贝到上一级目录，并删除后缀（如.gfortran）即可。

同时，打开makefile文件找到以下内容：

```
#lib: lapacklib tmglib
lib: blaslib variants lapacklib tmglib
```

 注释掉第1行，保留第2行，这样子可以保证每次再编译lapack时也将blas同时编译。

```bash
tar -zxvf lapack-3.12.1 #解压下载的压缩包 
cd lapack-3.12.1
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX=/opt/lapack ..
sudo cmake --build . -j --target install
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
