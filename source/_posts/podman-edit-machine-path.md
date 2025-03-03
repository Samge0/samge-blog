---
title: Windows下Podman Desktop容器目录迁移至其他硬盘的详细步骤
date: 2025-03-03 17:03:05
tags:
    - Podman Desktop
    - Windows Podman迁移
    - WSL2 Podman容器迁移
    - Podman配置文件修改
    - Podman机器目录迁移
    - Podman容器硬盘空间优化
    - Podman机器目录迁移教程
---

windows系统下，[podman-desktop](https://github.com/podman-desktop/podman-desktop)使用`wsl2`时会创建1个名为`podman-machine-default`容器，容器默认存放于C盘的`C:\Users\<用户名>\.local\share\containers\podman\machine\wsl\wsldist\podman-machine-default\ext4.vhdx`中。随着时间的推移，这个`ext4.vhdx`文件会越来越大，可能导致C盘空间不足，下面记录`podman-desktop`修改容器目录到其他的硬盘的操作记录。

> 本文默认将容器导出到 F:\.cache 目录，然后重新导入到 F:\podman\machine\wsl\wsldist\podman-machine-default 目录

- 导出镜像
    ```shell
    mkdir -p F:\.cache && mkdir -p F:\podman\machine\wsl\wsldist\podman-machine-default

    wsl --export podman-machine-default F:\.cache\podman-machine-default.tar
    ```

- 销毁原docker镜像：
    ```shell
    wsl --unregister podman-machine-default
    ```

- 重新导入镜像到F盘下

    使用刚备份的tar重新导入docker镜像并指定自定义的映射目录，并指定wsl版本为2
    ```shell
    wsl --import podman-machine-default F:\podman\machine\wsl\wsldist\podman-machine-default\ F:\.cache\podman-machine-default.tar --version 2
    ```

- 修改podman配置:
    - 迁移podman配置文件到F盘下

        手动将用户目录下的`C:\Users\<用户名>\.local\share\containers\podman\machine`目录下的的`wsl目录、machine文件、.pub文件`等复制到`F:\podman\machine`目录下

    - 修改`podman-machine-default.json`配置下的`machine`目录

        修改`C:\Users\<用户名>\.config\containers\podman\machine\wsl\podman-machine-default.json`配置文件下的`machine目录`为`F:\\podman\\machine`

    - 修改`podman-connections.json`配置下的`machine`目录

        修改`C:\Users\<用户名>\AppData\Roaming\containers\podman-connections.json`配置文件下的`machine目录`为`F:\\podman\\machine`

- 重启

    重启`podman-desktop`应用，对应的wsl容器会自动关联启动，此时容器目录已经迁移到F盘下了，使用podman拉取镜像，可观察到`F:\podman\machine\wsl\wsldist\podman-machine-default\ext4.vhdx`这个文件体积变化。
