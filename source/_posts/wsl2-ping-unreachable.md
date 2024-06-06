---
title: wsl2突然无法ping通宿主机的问题处理
date: 2024-06-06 15:22:13
tags: 
    - wls2
    - docker
    - windows
---

## 前言

windows下的docker用得好好的，但突然有一天docker容器无法访问宿主机中的代理，进入容器发现无法ping通宿主机。（我只是个小呆瓜，我也很懵啊）
![image](https://github.com/Samge0/samge-blog/assets/17336101/293ab6c5-d899-4581-97ee-e321e9572395)

经过重重排查，发现是wsl2中的网络配置问题，这里简单记录一下。


## 处理过程>>>
### 1、检查网络配置
查看wsl2网络配置跟宿主机网络配置是否一致

分别在wsl2虚拟机中执行`ifconfig`跟在windows中执行`ipconfig /all`，发现wsl2中的ip为`172.24.101.x`，windows中的WSL桥适配器的ip是`172.24.96.x`，需要将wsl2中的ip改成windows中WSL桥适配器同一网段的ip。
![image](https://github.com/Samge0/samge-blog/assets/17336101/4d27e4e2-b438-4bba-85e7-94adc2d890db)

### 2、修改配置
修改wsl2中的ip（例如这里修改为`172.24.96.100`）
- 【可选】在wsl2终端中临时修改，用于测试是否可行：
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


## 其他说明>>>
### 1、参考文章
- [WSL2 网络异常排查 [ping 不通、网络地址异常、缺少默认路由、被宿主机防火墙拦截]](https://www.jianshu.com/p/ba2cf239ebe0)