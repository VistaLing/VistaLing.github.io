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

# 二、[Intel® oneAPI HPC Toolkit](https://www.intel.com/content/www/us/en/developer/tools/oneapi/hpc-toolkit.html)下载与安装

高性能计算 （HPC） 是人工智能、机器学习和深度学习应用程序的核心。英特尔® oneAPI HPC 工具包为开发人员提供构建、分析、优化和扩展 HPC 应用程序所需的一切，采用矢量化、多线程、多节点并行化和内存优化方面的最新技术。

```bash
sudo sh ./intel-oneapi-hpc-toolkit-2025.0.1.47_offline.sh -a --silent --cli --eula accept
```

# 三、Intel Fortran编译器配置

每次使用前，需要先执行，临时环境有效。

```bash
source /opt/intel/oneapi/setvars.sh
```

将上述命令添加到 ~/.bashrc 文件中，使其永久生效：

```bash
vim ~/.bashrc
```

通过vim打开当前用户下所对应的.bashrc文件，在其中加入（建议添加在最下面）

```bash
source /opt/intel/oneapi/setvars.sh
```

保存退出之后 ，使用source这一命令执行一下就把新加的命令执行了。

```bash
source ~/.bashrc 
reboot
```

用which来检验下配置的环境变量是否正确。如果显示了其路径，则说明安装顺利完成了。

```bash
which ifx
which ifort
```

其中：

**ifx**：支持 Fortran 90 到 Fortran 2023 的标准，适合快速测试和调试。如果你只是想快速测试代码，ifx 是一个方便的选择。

**ifort**：支持 Fortran 90 到 Fortran 2023 的标准，功能更全面，适合专业开发和高性能计算。如果你需要使用最新的 Fortran 标准或高级功能（如并行编程），建议使用 ifort。

另外，2024年10月之后英特尔编译器不再包含ifort编译器，只有ifx去兼容。

# 四、默认Fortran编译器配置

如果系统之前有安装gfortran编译器，编译时候会调用编译器错误。

**错误原因**：gfortran 编译器不识别 -module 选项。

-module 是某些 Fortran 编译器（如 Intel Fortran 编译器）的选项，用于指定模块文件的输出目录。

gfortran 使用的是 -fmodules 选项来指定模块文件的输出目录。

**解决方法**：强制使用intel的icx和ifort（新版为ifx）编译器，需要在./configure之前，临时环境有效。

```
export CC=icx
export CXX=icpx
export F77=ifx
export F90=ifx
export FC=ifx
```

如编译提示编译器冲突，可以进行卸载，如：

```bash
unset F90=ifx
```
