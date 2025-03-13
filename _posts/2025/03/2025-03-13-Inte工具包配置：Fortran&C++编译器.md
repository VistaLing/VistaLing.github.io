---
title: Inte工具包配置：Fortran&C++编译器
subtitle: 
layout: post
author: "Ling"
header-style: text
tags:
  - Intel
  - 环境配置
  - Linux
  - C++
  - Fortran
---

Intel Developer Toolkits  开发人员工具包：使用一流的编译器、性能库、框架、分析器和调试工具，在 CPU、GPU 和 FPGA 上构建、分析和优化高性能、跨架构的应用程序。Windows下安装配置较为简单，主要记录为Linux环境。

# 一、[Intel® oneAPI Base Toolkit](https://www.intel.com/content/www/us/en/developer/tools/oneapi/base-toolkit.html)下载与安装

英特尔® oneAPI Base Toolkit （Base Kit） 是一组核心工具和库，用于跨不同架构开发高性能、以数据为中心的应用程序。它采用业界领先的 C++ 编译器，可实施 SYCL*，SYCL* 是 C++ 的演变，适用于异构计算。

```bash
sudo sh ./intel-oneapi-base-toolkit-2025.0.1.46_offline.sh -a --silent --cli --eula accept
```

如显示没有找到，则需要分别进行安装。 OpenMPI的按照方法也同理。

# 二、[Intel® oneAPI HPC Toolkit](https://www.intel.com/content/www/us/en/developer/tools/oneapi/hpc-toolkit.html)下载与安装

高性能计算 （HPC） 是人工智能、机器学习和深度学习应用程序的核心。英特尔® oneAPI HPC 工具包为开发人员提供构建、分析、优化和扩展 HPC 应用程序所需的一切，采用矢量化、多线程、多节点并行化和内存优化方面的最新技术。

```bash
sudo sh ./intel-oneapi-hpc-toolkit-2025.0.1.47_offline.sh -a --silent --cli --eula accept
```

如显示没有找到，则需要分别进行安装。

红帽系(Red Hat、CentOS、Fedora、openSUSE等)：

```bash
yum install glibc-headers
yum install gcc-c++
yum install gcc-gfortran
```

Debian系（Debian、Ubuntu、Mint等）:

```bash
apt-get install build-essential 
apt-get install g++
apt install gfortran
```

当检查完编译器之后，去[https://www.mpich.org/downloads/](https://link.zhihu.com/?target=https%3A//www.mpich.org/downloads/) 选择合适的版本下载，对于没有图形界面的服务器，也可使用wget命令下载。 tar命令是解压文件的命令。MPI库通常采用的是源码安装，因此，需要使用cd命令进入到解压后的文件夹中，使用./configure进行安装前的设置与检查，由于我们只需要更改一下安装的路径，因此在--prefix这一参数中，设置你想要安装的路径即可。执行这一行之后，就会开始检查编译环境是否满足，此时报错多半是因为编译器安装的问题或者编译器版本不匹配的问题，一般通过安装最新的编译器能解决。这一步成功完成之后，即可使用make命令去执行编译。该路径下makefile文件已经写好了这些源代码的编译规则，因此输入make即可开始按照makefile的规则对源码进行编译。如果你做的工作和底层语言如Fortran、C或C++之类的，还是有必要学习一下makefile的写法，有助于之后的多文件编译工作。顺便一提，使用Linux学习底层语言是更有好处的，因为Windows初学编程语言通常是使用集成开发环境，在对代码进行编译时通常是一键操作，导致中间的几个过程会被下意识的忽略掉，这将导致一开始适应不了Linux环境下编译文件的命令。当make完成之后，就可以使用make install命令进行安装了。

```bash
wget http://www.mpich.org/static/downloads/3.3.2/mpich-3.3.2.tar.gz 
tar -zxvf mpich-3.3.2.tar.gz #解压下载的压缩包 
cd mpich-3.3.2 #进入解压后的文件夹内 
./configure  --prefix=/usr/local/mpich-3.3.2 
# --prefix这一参数是设置安装的路径，根据需要设置合适的路径即可，但需要记住安装的位置 
make -j
sudo make install
```

# 二、环境变量的配置

安装完成后，可以去之前--prefix设置的路径去看一下安装结果。安装好了之后，还需要告诉系统mpi库的路径地址，这样当你调用mpi的命令时，系统才知道你在干什么。在用户的根目录下，有一个.bashrc的文本文件。这个文件可以理解为，每次打开终端时都会加载的启动项。~即为家目录，也就是用户的根目录。

```bash
vim ~/.bashrc
```

通过vim打开当前用户下所对应的.bashrc文件，在其中加入（建议添加在最下面）

```bash
export PATH="/usr/local/mpich-3.3.2/bin:$PATH"
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/mpich-3.3.2/lib
```

保存退出之后 ，使用source这一命令执行一下就把新加的命令执行了。

```bash
source ~/.bashrc 
```

用which来检验下配置的环境变量是否正确。如果显示了其路径，则说明安装顺利完成了。

```bash
which mpicc
which mpiexec
which mpif90 
```

提示：mpicc、mpif90是编译命令；mpiexec是执行命令。

进入到最开始解压的文件夹中，到解压的文件夹内的examples文件夹中，测试一下hello是否能顺利运行。首先编译然后运行：

```bash
mpicc -o hellow hellow.c
mpirun -np 4 ./hellow
```

若可运行说明顺利完成安装。

```
Hello world from process 1 of 4
Hello world from process 3 of 4
Hello world from process 0 of 4
Hello world from process 2 of 4
```

##### 参考资料

1.都志辉. 高性能计算并行编程技术:MPI并行程序设计[M]. 清华大学出版社, 2001.

2.[两小时入门MPI与并行计算系列 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/355652501)