---
title: PGI Fortran + Intel MKL 安装使用
date: 2019-03-19
mathjax: true
tags:
  - PGI Fortran
  - MKL
categories:
  - 笔记
---


## 安装 PGI Community Edition 

前往官网的下载页面，下载[免费社区版](https://www.pgroup.com/products/community.htm)，选择 Windows 系统即可。首页底部的`Documentation`->`Installation Guide`里面有对 Linux 、macOS 和 Windows 操作系统下安装过程详细的说明。其中 Windows 部分在[第 6 节](https://www.pgroup.com/resources/docs/19.4/x86/pgi-install-guide/index.htm#install-win-pgi)，这里给出简单的总结：

<!-- more -->

### 安装步骤

正式安装之前，应确保 PGI 软件已下载，而且其他必需的非 PGI 软件已经安装完毕，具体而言就是如下 3 个：

1. [Microsoft Update for Universal C Runtime](https://support.microsoft.com/en-us/help/2999226/update-for-universal-c-runtime-in-windows)，通常而言，Windows的自动更新已经确保它已经安装上。

1. Microsoft Windows SDK

1. Microsoft Visual Studio 2017

最后两项安装 Visual Studio 2017 社区版就行，`工作负载`选择`使用 c++ 的桌面开发`，可以看到右侧安装详细信息里默认有 `Windows 10 SDK` 选项，其他选择默认即可。

![title](/images/vs.png)

最后以管理员权限运行 PGI 安装包即可，最后把 `bin` 文件夹需要添加至环境变量。

### 测试一下

```Fortran
PROGRAM main
  IMPLICIT NONE
  INTEGER i,j
  INTEGER, PARAMETER :: n = 3
  REAL(8) :: A(n,n), x(n,1), y(n,1)
  
  DO j = 1,n
    DO i = 1,n
      A(i,j) = (j-1)*n + i
    END DO
    x(j,1) = j
  END DO
  
  y = MATMUL(A,x)

  WRITE(*,*) y

END PROGRAM main
```

顺利的话，可以看到编译成功，并输出正确结果。

```
pgfortran main.f90
main.exe
```
