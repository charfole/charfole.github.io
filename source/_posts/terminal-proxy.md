---
title: 各类终端走代理的设置方法
date: 2021-02-27 20:27:19
tags:
- terminal
- proxy
- shell
- WSL
categories: 技术
---

最近在 GitHub 上游玩，发现各种终端中 git 操作都变得奇慢无比，于是今天一并记录一下各类终端走代理的方法。

假设代理的端口号为1080，以下的一切命令均通过 `curl www.google.com` 命令验证成功。

## Powershell

```powershell
$env:http_proxy="http://127.0.0.1:1080"
$env:https_proxy="http://127.0.0.1:1080"
```
<!--more-->


## CMD

```powershell
set http_proxy=http://127.0.0.1:1080
set https_proxy=http://127.0.0.1:1080
```



## Git Bash

```shell
git config --global https.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080
```



## WSL2

WSL2 的情况稍微复杂一点，因为 WSL2 不像 WSL1 一样和 Windows 共享网络端口，而是为 Linux 子系统分配了一张新的虚拟网卡，让Linux 虚拟机和 Windows 组成了一个局域网，因此想要通过运行在 Windows 的代理来上网，必须获取 Windows 的主机地址。

```shell
# 需预先获取主机地址保存到变量中
host_ip=$(cat /etc/resolv.conf |grep "nameserver" |cut -f 2 -d " ")
export ALL_PROXY="http://$host_ip:1080"
```
建议将上述命令写到终端的配置文件（默认为.bashrc）中，这样每次启动 WSL 时就不需要手动设置一遍了。


## 以后再折腾

听说 proxychains 可以让每个应用单独走代理，而不需要用全局的方法，但目前暂时还没有这种需求，以后遇到了折腾。

