---
title: Cuda环境配置01：WSL2+Linux
subtitle: 
layout: post
author: "Ling"
header-style: text
tags:
  - GPU
  - Cuda
  - WSL
  - Linux
  - 环境配置
---

# 一、Windows下的Cuda环境配置

Nvidia官方页面通过选择，有不同系统、不同架构、不同版本的CUDA Toolkit可供离线和在线安装，所以版本内容的存档在[CUDA Toolkit Archive | NVIDIA Developer](https://developer.nvidia.com/cuda-toolkit-archive)均可以找到，请注意一一对应。

Windows平台的安装比较简单，下载需要版本后，双击即可自动完成安装和系统环境配置。

通过在cmd控制台输入指令，可以校验cuda driver_api配置的正确性：

```
nvidia-smi
```

显示类似如下：

```
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 536.99                 Driver Version: 536.99       CUDA Version: 11.8     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                     TCC/WDDM  | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  NVIDIA GeForce RTX 4070      WDDM  | 00000000:01:00.0  On |                  N/A |
|  0%   36C    P8               6W / 215W |    562MiB / 12282MiB |      0%      Default |
|                                         |                      |                  N/A |
+-----------------------------------------+----------------------+----------------------+
```

通过在cmd控制台输入指令，可以校验cuda runtime_api配置的正确性：

```
nvcc -V 或 nvcc --version
```

显示类似如下：

```
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2022 NVIDIA Corporation
Built on Wed_Sep_21_10:41:10_Pacific_Daylight_Time_2022
Cuda compilation tools, release 11.8, V11.8.89
Build cuda_11.8.r11.8/compiler.31833905_0
```

# 二、WSL2下的Linux系统部署

微软[安装 WSL | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/install)在Windows 10 21H2以及更高版本中提供了WSL2可以直接驱动Linux系统，同时英伟达官方提供该虚拟系统的Cuda的直驱版本，非常方便可以使Windows和Linux无缝衔接，双系统调试和编译。

在管理员模式下打开 PowerShell 或 Windows 命令提示符，输入命令：

```
wsl --install
```

此命令将启用运行 WSL 并安装 Linux 的 Ubuntu 发行版所需的功能，默认为Ubuntu系统。输入下面命令可以查询在线可安装的Linux系统版本：

```
wsl --list --online 或 wsl -l -o
```

目前有如下一些系统：

```
NAME                                   FRIENDLY NAME
Ubuntu                                 Ubuntu
Debian                                 Debian GNU/Linux
kali-linux                             Kali Linux Rolling
Ubuntu-18.04                           Ubuntu 18.04 LTS
Ubuntu-20.04                           Ubuntu 20.04 LTS
Ubuntu-22.04                           Ubuntu 22.04 LTS
OracleLinux_7_9                        Oracle Linux 7.9
OracleLinux_8_7                        Oracle Linux 8.7
OracleLinux_9_1                        Oracle Linux 9.1
openSUSE-Leap-15.5                     openSUSE Leap 15.5
SUSE-Linux-Enterprise-Server-15-SP4    SUSE Linux Enterprise Server 15 SP4
SUSE-Linux-Enterprise-15-SP5           SUSE Linux Enterprise 15 SP5
openSUSE-Tumbleweed                    openSUSE Tumbleweed
```

可使用下面命令指定安装：

```
wsl --install -d <NAME>
```

后续就是等待和根据提示输入账户名和密码就好。

# 三、Linux下的Cuda环境配置

在[CUDA Toolkit Archive | NVIDIA Developer](https://developer.nvidia.com/cuda-toolkit-archive)页面中选择对应windows主机的cuda版本，选择Linux下WSL_Ubuntu系统，个人喜好使用第三种方式离线安装：

```
wget https://developer.download.nvidia.com/compute/cuda/11.8.0/local_installers/cuda_11.8.0_520.61.05_linux.run
sudo sh cuda_11.8.0_520.61.05_linux.run
```

**切记选择WSL_Ubuntu**，原因是WSL的硬件直驱需要使用Windows主机的显卡驱动，而不使用Ubuntu的显卡驱动。其它版本的Cuda工具虽然可以安装，但是会自带有Linux显卡驱动，造成错误；WSL版本文件中已经剔除掉显卡驱动。

```
#进入配置文件
vim ~/.bashrc
#启用编辑
i

#添加以下两行
#在/.bashrc中配置LD_LIBRARY_PATH路径、配置PATH路径，完整配置如下：
export LD_LIBRARY_PATH=/usr/local/cuda/lib
export PATH=$PATH:/usr/local/cuda/bin
#保存退出
ESC
:wq

#刷新生效
source ~/.bashrc
```

最后，和Windows一样，验证nvidia--smi和nvcc-V配置的正确性。

