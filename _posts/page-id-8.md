---
title: "定制你的编译过程：像专业人士一样从源码构建和安装LLVM&Clang"
date: 2023-06-21 11:02:46
img: https://wangxs020202.gitee.io/pbad/background/page_8.png
top: false
summary: 本文将介绍如何从源码构建和安装LLVM&Clang，以及如何配置开发环境。
categories: 笔记
tags:
  - LLVM
  - Clang
  - 编译
  - 安装
  - 教程
  - 笔记
---

### 序幕

LLVM是一个功能强大的编译器基础设施，而Clang则是其最著名的前端。本文将介绍如何从源码构建和安装LLVM&Clang，以及如何配置开发环境。

### LLVM

1. 简介

   LLVM是一个开源的编译器基础设施，由C++编写，提供了一系列工具和库，可以用于构建编译器、优化器、链接器等。Clang是LLVM的一个前端，支持C、C++、Objective-C和Objective-C++等语言。通过使用LLVM&Clang，你可以轻松构建自己的编译器，或者为现有的编译器添加新的功能。
2. 下载源码

   你可以从[LLVM官网](https://llvm.org/)下载最新的源码包，也可以从[GitHub](https://github.com/llvm/llvm-project/)下载最新的稳定版本。
   本文以LLVM 11.0.0为例，下载地址为：[LLVM 11.0.0](https://github.com/llvm/llvm-project/releases/tag/llvmorg-11.0.0)
3. 安装依赖

   LLVM&Clang的构建过程需要一些依赖，这些依赖可以通过包管理器安装，也可以从源码构建。本文以Ubuntu 20.04为例，介绍如何安装依赖。

    ```bash
    sudo apt-get install build-essential cmake
    ```
4. 解压源码

    ```bash
    sudo apt-get install unzip
    unzip you/zip/path -o you/project/unzip/path
    ```
5. 创建构建目录

    ```bash
    mkdir you/build/path
    ```
6. 配置构建

    ```bash
    cd you/build/path
    # 在 shell 中执行此命令，将path/to/llvm/source/root替换为 LLVM 源树根目录的路径
    cmake path/to/llvm/source/root
    ```
7. 开始构建

    ```bash
    # CMake 运行完成后，从构建目录开始构建
    cmake --build .
    ```
8. 安装

    ```bash
   # LLVM 完成构建后，从构建目录安装它
    cmake --build . --target install
    ```
9. 配置开发环境

    ```bash
    # 在 shell 中执行此命令，将path/to/llvm/install/root替换为 LLVM 安装根目录的路径
    export PATH=path/to/llvm/install/root/bin:$PATH
   ```
10. 测试

    ```bash
    # 测试 LLVM 是否安装成功
    llvm-as --version
    ```

### Clang

1. 简介

   Clang是LLVM的一个前端，支持C、C++、Objective-C和Objective-C++等语言。通过使用LLVM&Clang，你可以轻松构建自己的编译器，或者为现有的编译器添加新的功能。

2. 构建安装
   Clang的构建安装过程与LLVM类似。
3. 测试

   ```bash
   # 测试 Clang 是否安装成功
   clang --version
   ```

### 总结

本文介绍了如何从源码构建和安装LLVM&Clang，以及如何配置开发环境。如果你想了解更多关于LLVM&Clang的内容，可以参考[LLVM官网](https://llvm.org/)和[Clang官网](https://clang.llvm.org/)。
