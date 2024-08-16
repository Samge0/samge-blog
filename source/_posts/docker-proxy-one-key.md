---
title: 一键部署 Docker 代理：DockerProxyOneKey 开源项目介绍
date: 2024-08-17 06:04:46
tags:
    - 编程
    - 脚本
    - docker
    - 代理
    - DockerProxyOneKey
    - 开源
    - Cloudflare
    - Caddy
---

<p align="center" style="background-color: #33f0f0f0;">
    <img src="https://github.com/user-attachments/assets/d32957f6-94fd-4ae2-83f3-12622b7d9429" alt="DockerProxyOneKey" width="200"/>
</p>

## 引言
在当今的云计算和容器化技术环境中，简化和加速部署过程成为了开发者们的共同目标。DockerProxyOneKey 项目就是为此目的设计的，它提供了一个便捷的解决方案，通过一键部署来轻松设置 Docker 代理，节省时间与精力。

## 什么是 DockerProxyOneKey？
DockerProxyOneKey 是一个开源项目，旨在通过一键脚本帮助用户快速部署和配置 Docker 代理。该项目通过整合多种工具和服务，提供了一个高效且灵活的解决方案，适合那些希望简化部署过程的开发者。

## 核心功能
DockerProxyOneKey 提供了一系列核心功能，使其成为一个强大的工具：
- **一键安装与部署**：通过简单的脚本，用户可以轻松完成 Docker 代理的安装与配置，避免了繁琐的手动操作。
- **灵活的配置**：支持多种配置选项，用户可以根据自己的需求进行定制，包括代理的端口、证书等。
- **支持多平台**：该工具不仅可以在 Linux 环境下使用，还支持多种服务器架构，包括 x86 和 ARM。

## 项目安装与使用
DockerProxyOneKey 的安装过程非常简单，只需几个步骤即可完成：
- 1. **克隆项目**：首先，用户需要克隆项目的 GitHub 仓库。
```shell
git clone https://github.com/Samge0/DockerProxyOneKey.git

cd DockerProxyOneKey
```

- 2. **执行一键脚本**：进入项目目录后，运行以下命令以启动一键部署：
```shell
chmod +x run.sh

./run.sh \
--domain your_domain.com \
--reverse_proxy_server http://127.0.0.1 \
--cf_token your_cloudflare_token
```

## 典型应用场景
DockerProxyOneKey 适用于以下几种典型场景：
- **多容器部署环境**：对于需要在多容器环境中进行代理配置的用户，该工具能够大大简化操作流程。
- **云服务器管理**：在管理多个云服务器时，通过 DockerProxyOneKey，您可以轻松在不同的服务器上统一配置 Docker 代理。
- **跨平台支持**：如果您的开发环境涉及多个平台，如 x86 和 ARM，这款工具的跨平台支持将为您带来便利。

## 开源与社区支持
DockerProxyOneKey 是一个开源项目，用户可以通过 GitHub 仓库获取最新版本。

- GitHub 仓库地址：[https://github.com/Samge0/DockerProxyOneKey](https://github.com/Samge0/DockerProxyOneKey)


## 可选操作

[可选] 指定镜像仓库
```shell
./run.sh \
--services hub,ui,caddy \
--domain your_domain.com \
--reverse_proxy_server http://127.0.0.1 \
--cf_token your_cloudflare_token
```

[可选] 自定义configs配置目录
```shell
./run.sh \
--services hub,ui,caddy \
--domain your_domain.com \
--reverse_proxy_server http://127.0.0.1 \
--cf_token your_cloudflare_token \
--custom_cofigs_dir /path/to/your/configs
```

[可选] 自定义yml配置文件中的参数
```shell
./run.sh \
--services hub,ui,caddy \
--domain your_domain.com \
--reverse_proxy_server http://127.0.0.1 \
--cf_token your_cloudflare_token \
--update_yml true \
--proxy_ttl 168h \
--health_enabled true \
--health_interval 10s \
--health_threshold 3 \
--http_max_age 1728000 \
--storage_upload_enabled true \
--storage_upload_age 168h \
--storage_upload_interval 24h \
--storage_readonly false
```

[可选] 参考脚本参数说明
```shell
./run.sh -h
```


## Cloudflare说明
- 购买一个便宜的域名
- 在[Cloudflare](https://dash.cloudflare.com)中添加域名，配置DNS记录，添加A记录+子域名泛解析，指向`IPv4`地址，开启CDN`Proxy`（小黄云）
- 在Cloudflare中[创建API Token](https://dash.cloudflare.com/profile/api-tokens)，并选择`Zone: Zone: DNS: Edit DNS`权限
- 在目标域名的`SSL/TLS`选项卡中开启HTTPS的`完全（严格）/ Full（Strict）`模式
![image](https://github.com/user-attachments/assets/c2ac3b10-8f00-487b-ac65-34e4b15b40ba)
![image](https://github.com/user-attachments/assets/cb27e29d-9945-4829-bc8d-7d285fc98938)


## 相关截图
![image](https://github.com/user-attachments/assets/a5520800-1f94-4dd1-acd2-6e4d15708e89)
![image](https://github.com/user-attachments/assets/679cd11e-8331-4b5f-ba4d-e24cda4d684f)
![image](https://github.com/user-attachments/assets/face75c5-74e1-4aa1-b040-d40cf21f8fae)


## 总结
对于需要在复杂的多容器环境中部署代理的开发者而言，DockerProxyOneKey 提供了一个简化流程、提高效率的解决方案。

通过一键部署和灵活配置，它能够帮助您快速上手，节省宝贵的开发时间。希望这篇文章能够帮助您更好地了解并使用 DockerProxyOneKey 项目。
