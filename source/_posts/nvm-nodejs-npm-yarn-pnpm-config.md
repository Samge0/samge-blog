---
title: 前端的nvm、nodejs、npm、yarn、pnpm配置备忘
date: 2024-09-26 17:38:19
tags:
    - nvm
    - nodejs
    - npm
    - yarn
    - pnpm
---

### 前言
`npm、yarn、pnpm`默认的`cache目录在`系统盘，现希望迁移其缓存`cache`及`global`目录到`F:\.cache\nodejs`下。

> 这里默认用windows安装为例

推荐路线：`安装nvm` -》 `用nvm安装nodejs` -》 `配置npm` -》【可选】`安装yarn/pnpm`（这两个也是跟npm类似的包管理工具，可选）

### 参考文章
- [关于这几个工具的介绍&对比文章](https://blog.csdn.net/zm06201118/article/details/125048608)
- [关于npm、yarn、pnpm的缓存路径与清理的介绍](https://www.cnblogs.com/breezefaith/p/17976013)

### 安装nvm
- [windows](https://github.com/coreybutler/nvm-windows/releases)

    windows的从release中下载安装文件进行安装

- [linux/mac](https://github.com/nvm-sh/nvm)

    linux/mac的执行仓库提供的sh脚本进行安装，安装完毕后在终端可以看到nvm的安装目录，默认在`~/.nvm`

- 安装完毕后修改默认配置：

    修改nvm安装目录下的settings.txt文件，可指定nvm的路径跟nodejs的路径（nodejs的路径实际默认会安装在nvm的安装目录下，指定的nodejs路径其实是一个软链接路径）：
    ```text
    root: F:/APP/nvm
    path: F:/APP/nvm/nodejs
    arch: 64
    originalpath: 
    originalversion: 
    node_mirror: https://registry.npmmirror.com/binary.html?path=node/
    npm_mirror: https://registry.npmmirror.com/binary.html?path=npm/
    ```

- 配置环境变量（方面直接用`nvm命令`、`nodejs全局命令`）：
    ```text
    setx NVM_HOME "F:\APP\nvm" /M
    setx NVM_SYMLINK "F:\APP\nvm\nodejs" /M
    ```
    在Path中追加配置：`%NVM_HOME%` 跟 `%NVM_SYMLINK%`

### 用nvm安装nodejs：
- 安装并使用nodejs
    ```shell
    nvm install 20
    nvm use 20
    ```

- 查看nvm版本
    ```shell
    nvm v
    ```

- 查看node版本：
    ```shell
    node --version
    ```

### 配置npm：
配置文件路径为：`C:/Users/userName/.npmrc`，自定义缓存目录跟仓库源（这里用了`nvm`就不需要配置`prefix`目录了，默认-g的全局依赖包会安装到nvm下的nodejs目录）

- 指定cache目录+仓库源：
    ```shell
    npm config set cache "F:/.cache/nodejs/npm/cache" && npm config set registry "http://registry.npmmirror.com"
    ```

- 查看当前配置：
    ```shell
    npm config ls -l
    ```

- 安装全局依赖：
    ```shell
    npm install -g <package_name>
    ```

- 移除全局依赖：
    ```shell
    npm uninstall -g <package_name>
    ```

- 查看全局依赖：
    ```shell
    npm list -g
    ```

- 安装局部依赖（默认安装在当前项目根目录的`node_modules`下）:
    ```shell
    npm install <package_name>
    ```

- 移除局部依赖：
    ```shell
    npm uninstall <package_name>
    ```

- 【仅记录，请忽略】原先默认的路径配置：
    ```shell
    cache = "C:/Users/userName/AppData/Local/npm-cache"
    prefix = "C:/Program Files/nodejs"
    ```

- 获取缓存路径：
    ```shell
    npm config get cache
    ```

- 清理缓存：
    ```shell
    npm cache clean -f
    ```


### 【可选】安装yarn
配置文件路径为：`C:/Users/userName/.yarnrc`

- 安装：
    ```shell
    npm install -g yarn
    ```

- 配置cache跟glabal目录+仓库源（执行完毕后需要将global目录配置到环境变量中）：
    ```shell
    yarn config set prefix "F:\.cache\nodejs\Yarn\Data" && `
    yarn config set global-folder "F:\.cache\nodejs\Yarn\Data\global" && `
    yarn config set cache-folder "F:\.cache\nodejs\Yarn\Cache" && `
    yarn config set link-folder "F:\.cache\nodejs\Yarn\Data\link" && `
    yarn config set registry "http://registry.npmmirror.com"
    ```

- 查看当前配置：
    ```shell
    yarn global dir && yarn cache dir
    ```

- 安装全局依赖：
    ```shell
    yarn global add <package_name>
    ```

- 移除全局依赖：
    ```shell
    yarn global remove <package_name>
    ```

- 查看全局依赖：
    ```shell
    yarn global list <package_name>
    ```

- 安装局部依赖（默认安装在当前项目根目录的`node_modules`下）:
    ```shell
    yarn add <package_name>
    ```

- 移除局部依赖：
    ```shell
    yarn remove <package_name>
    ```

- 【仅记录，请忽略】原先默认的路径配置：
    ```shell
    cache = "C:/Users/userName/AppData/Local/Yarn/Cache"
    prefix = "C:/Users/userName/AppData/Local/Yarn/Data"
    ```

- 获取缓存路径：
    ```shell
    yarn cache dir
    ```

- 清理缓存：
    ```shell
    yarn cache clean -f
    ```


### 【可选】安装pnpm
默认配置文件路径为：`C:\Users\userName\AppData\Local\pnpm\config\rc`

- 安装：
    ```shell
    npm install -g pnpm
    ```

- 配置环境变量（全局包会存放于该目录）
    ```text
    setx PNPM_HOME "F:\.cache\nodejs\pnpm" /M
    ```
    在Path中追加配置：`%PNPM_HOME%`

- 配置cache跟glabal目录+仓库源：
    ```shell
    pnpm config set cache-dir "F:/.cache/nodejs/pnpm/cache" && `
    pnpm config set store-dir "F:/.cache/nodejs/pnpm/store" && `
    pnpm config set registry "http://registry.npmmirror.com"
    ```

- 查看当前配置：
    ```shell
    pnpm config list
    ```

- 安装全局依赖：
    ```shell
    pnpm add -g <package_name>
    ```

- 移除全局依赖：
    ```shell
    pnpm remove -g <package_name>
    ```

- 查看全局依赖：
    ```shell
    pnpm list -g <package_name>
    ```

- 安装局部依赖（默认安装在当前项目根目录的node_modules下）:
    ```shell
    pnpm add <package_name>
    ```

- 移除局部依赖：
    ```shell
    pnpm remove <package_name>
    ```

- 【仅记录，请忽略】原先默认的路径配置：
    ```shell
    cache = "C:/Users/userName/AppData/Local/pnpm"
    prefix = "C:/Users/userName/AppData/Local/pnpm"
    ```

- 获取缓存路径：
    ```shell
    pnpm store path
    ```

- 清理缓存：
    ```shell
    pnpm store prune
    ```


### 汇总需要配置到环境变量的目录：
```text
%NVM_HOME%
%NVM_SYMLINK%
%PNPM_HOME%
F:\.cache\nodejs\Yarn\Data\bin
```
