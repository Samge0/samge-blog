---
title: uv虽好，不可贪杯
date: 2025-05-14 00:42:02
tags:
    - uv
    - conda
    - python
---

最近在使用mcp服务时接触了[uv](https://github.com/astral-sh/uv)，感觉其的依赖构建速度确实迅速，第一感觉挺不错的，于是尝试在一些新项目中逐步使用uv替换conda。

但是用一段时间后，发现uv还是不够稳定，有时候存在一些莫名奇妙的问题，命名依赖都安装了，但运行却还是存在缺失依赖问题。然后改为conda创建环境并安装依赖，运行就是正常的。

暂且没有深入研究，但uv确实存在一些问题，使用过程中需要多久留意。

比如最近使用pyinstall+customtkinter打包ubuntu环境下的可执行程序，在物理机中打包跟执行是正常的，然后切换到docker环境下的[gezp/ubuntu-desktop:24.04](https://hub.docker.com/r/gezp/ubuntu-desktop/tags)容器桌面环境测试，使用uv创建的环境打包后，运行时提示缺失依赖。
我怀疑过docker容器的问题、兼容问题，就是没怀疑uv的问题。后面改为conda创建环境，打包后运行就正常了，有种淡淡的忧伤……-_-