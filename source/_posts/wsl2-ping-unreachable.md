---
title: wsl2中docker环境突然无法ping通宿主机的问题处理
date: 2024-06-06 15:22:13
tags: 
    - wls2
    - docker
    - windows
    - bridge
    - 子网冲突
---

> 【省流提示】：以下内容作用不大，仅属于作者个人的踩坑记录，请忽略。


## 前言

windows下的docker用得好好的，但突然有一天docker容器无法访问宿主机中的代理，进入容器发现无法ping通宿主机。（我只是个小呆瓜，我也很懵啊）
![image](https://github.com/Samge0/samge-blog/assets/17336101/293ab6c5-d899-4581-97ee-e321e9572395)



## 处理过程>>>
### 1、检查网络配置
查看wsl2网络配置跟宿主机网络配置是否一致

分别在wsl2虚拟机中执行`ifconfig`跟在windows中执行`ipconfig /all`，发现wsl2中的ip为`172.24.101.x`，windows中的WSL桥适配器的ip是`172.24.96.x`。
![image](https://github.com/Samge0/samge-blog/assets/17336101/4d27e4e2-b438-4bba-85e7-94adc2d890db)

### 2、修改配置
修改wsl2中的ip（例如这里修改为`172.24.96.100`）
- 【可选】在wsl2终端中直接修改，测试是否可行：
    ```shell
    sudo ifconfig eth0 172.24.96.100 netmask 255.255.240.0
    sudo route add default gw 172.24.96.1 eth0
    ```
- 【推荐】修改wsl2虚拟机中的配置文件`/etc/netplan/01-netcfg.yaml`
    ```text
    cat <<EOF | tee /etc/netplan/01-netcfg.yaml
    network:
    version: 2
    ethernets:
        eth0:
        dhcp4: no
        addresses: [172.24.96.100/20]
        gateway4: 172.24.96.1
        nameservers:
            addresses: [8.8.8.8]
    EOF
    ```
    然后执行 `sudo netplan apply` 使配置生效

### 3、验证结果
修改wsl2的ip配置后，可以正常ping通宿主机了。
![image](https://github.com/Samge0/samge-blog/assets/17336101/c318db43-5f42-4430-a1a9-9f0632540f6c)


### 4、被啪啪打脸
以为上面可以ping通就可以了吗？

NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO!NO^10086

测试了其他docker容器，发现还是无法ping通，所以上面解决方案是错误的（留下没有技术的泪水(̂ ˃̥̥̥ ˑ̫ ˂̥̥̥ )̂……洪湖水啊……）。

![image](https://github.com/Samge0/samge-blog/assets/17336101/a797c956-8cdb-4001-96f1-7361a497f025)


### 5、继续排查继续尝试解决
心想这问题就是这两天发生的，`嫌疑犯`必定在其中。我似乎回忆起最近用vscode的`Dev Containers`插件来运行过开发环境，经排查，确实又找到了一些线索。

以下是其中一个`.devcontainer/docker-compose.yml`的配置。
```yaml
version: '3.8'
services:
  storydiffusion:
    build: 
      context: ..
      dockerfile: .devcontainer/Dockerfile-dev
      args:
        PROXY: http://192.168.50.48:7890
    volumes:
      - ..:/app    
      - F:/.cache:/root/.cache    
    command: sleep infinity
    restart: unless-stopped
    environment:
      NVIDIA_VISIBLE_DEVICES: 0
      XXXX_DEMO: XXXX_DEMO
    deploy:
      resources:
        reservations:
          devices:
          - driver: nvidia
            count: 1
            capabilities: [gpu]
    shm_size: 22g
    ports:
      - "0.0.0.0:7890:7860"
```
可以看到`PROXY`配置了端口是`7890`的代理，而下面的`ports`本来应该将容器的`7860`映射到宿主机的`7860`，但是实际上映射到了`0.0.0.0:7890`。

我怀疑过是否跟端口冲突有关，但是正常情况下，启动docker容器时应该会报端口占用才对，这里没有提示，为什么呢？可能是当时代理软件没有打开，而`7890`端口刚好未被占用吧。

于是乎，开发环境的docker容器就这么启动了，且中奖式地创建了一条`192.168.50.0/20`的`bridge`网络。而我宿主机的地址是`192.168.50.48`，跟这个docker的`bridge`网络有冲突。

我的网络知识实在有限（留下没有技术的泪水……(-̩̩̩-̩̩̩-̩̩̩-̩̩̩-̩̩̩___-̩̩̩-̩̩̩-̩̩̩-̩̩̩-̩̩̩)），实在不明白这一切为什么就这么巧合地发生了。

![image](https://github.com/Samge0/samge-blog/assets/17336101/a797c956-8cdb-4001-96f1-7361a497f025)

我只知道，当我将那条`192.168.50.0/20`的`bridge`网络删除掉 + 修正之前错误的`7890`端口映射后，一切都恢复了正常。

我感到欢喜又感到悲伤。欢喜是因为这个问题终于解决了，可以继续`c`docker了。悲伤的是，我并不知道到底是什么原因，导致docker创建了`192.168.50.0/20`的`bridge`网络。 

或许 Docker 在创建默认 bridge 网络时检测到冲突（例如，如果 `172.17.0.0/16` 已经被占用），它可能会选择其他可用的子网。而 `192.168.50.0/20` 碰巧是下一个可用的子网，那么它就选择了这个子网……
![image](https://github.com/Samge0/samge-blog/assets/17336101/b886082e-1e2d-409b-83be-6633d3a0f4e9)

我能想到的是，这一切可能都源于中奖式的巧合（再再次留下没有技术的泪水……）(༎ຶ ෴ ༎ຶ) ( ´༎ຶㅂ༎ຶ`) (༎ຶ⌑༎ຶ) ฅ(⌯͒•̩̩̩́ ˑ̫ •̩̩̩̀⌯͒)ฅ

![image](https://github.com/Samge0/samge-blog/assets/17336101/a797c956-8cdb-4001-96f1-7361a497f025)


### 6、小结

其实上面的`.devcontainer/docker-compose.yml`配置，没有明确配置docker的网络，启动时会创建一个新的docker桥接网络。这里建议还是主动指定容器的网络，尽可能的避免奇怪（这里的奇怪仅对于我这样的小白而言）的问题发生。

没指定网络的情况：
![image](https://github.com/Samge0/samge-blog/assets/17336101/b886082e-1e2d-409b-83be-6633d3a0f4e9)

指定网络的情况：
![image](https://github.com/Samge0/samge-blog/assets/17336101/1f55ed21-5801-4aa7-9a5f-46a9288d68f8)

如何配置指定的网络？只需要在`.devcontainer/docker-compose.yml`中增加`network_mode: bridge`即可，默认会自动配置到名称为`bridge`的网络下。

如果需要配置其他网络，则需要手动创建，并指定网络名称，例如这里创建一个名为`vscode_devcontainer`的网络：。

- 创建网络
  ```shell
  docker network create vscode_devcontainer
  ```

- 配置`.devcontainer/docker-compose.yml`
  ```yaml
  version: '3.8'
  services:
    storydiffusion:
      build: 
        context: ..
        dockerfile: .devcontainer/Dockerfile-dev
        args:
          PROXY: http://192.168.50.48:7890
      volumes:
        - ..:/app    
        - F:/.cache:/root/.cache    
      command: sleep infinity
      restart: unless-stopped
      environment:
        NVIDIA_VISIBLE_DEVICES: 0
        XXXX_DEMO: XXXX_DEMO
      deploy:
        resources:
          reservations:
            devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
      network_mode: bridge
      networks:
        - vscode_devcontainer
      shm_size: 22g
      ports:
        - "0.0.0.0:7860:7860"
  ```

- 修改`.devcontainer/devcontainer.json`
  ```json
  {
      "name": "storydiffusion",
      "dockerComposeFile": "docker-compose.yml",
      "service": "storydiffusion",
      "workspaceFolder": "/app",
      "extensions": [
          "ms-python.python",
          "ms-python.debugpy",
          "humao.rest-client",
          "samge.vscode-samge-translate"
      ],
      "network": "vscode_devcontainer"
  }
  ```

## 其他说明>>>
### 1、参考文章
- [WSL2 网络异常排查 [ping 不通、网络地址异常、缺少默认路由、被宿主机防火墙拦截]](https://www.jianshu.com/p/ba2cf239ebe0)
