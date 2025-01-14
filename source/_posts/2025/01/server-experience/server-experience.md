---
title: 服务器常见问题
date: 2025-01-01
updated: 2025-01-14
permalink: 2025/01/server-experience/
categories: 服务器
---

## 概述

本文总结了在服务器使用上常见的问题。

## `MySQL` 占用内存过高

参见 `https://stackoverflow.com/questions/10676753/reducing-memory-consumption-of-mysql-on-ubuntuaws-micro-instance`。

具体做法

```ini
[mysqld]
performance_schema = 0
```

可以 cut off 近 20% 的内存占用，我的服务器一共约 900MB 的内存，也就是释放了约 200MB 的空间出来。

之所以发现这个问题，是因为 2025-01-01 早上起来发现服务器登录不上去，当天凌晨的 CPU 占用率 100%。用 `top` 命令发现内存只剩下 5% 左右，很容易溢出。

还有一个方案是

```ini
key_buffer              = 8M 
max_connections         = 30 # Limit connections
query_cache_size        = 8M # try 4m if not enough 
query_cache_limit       = 512K
thread_stack            = 128K
```

然而 `query_cache_size` 和 `query_cache_limit` 已经 `deprecated`，而且 `key_buffer` 要替换为 `key_buffer_size`，我看了看，效果不是很明显。

## 停止再启动服务器后，连接不上了

这里之所以采用停止再启动的说法，是因为重启不会改变机器的 `IP`，而停止后会的。所以说，要么申请一个弹性 `IP`，就始终是你了。要么就每次停止再启动后，手动修改 `DNS` 解析。还是前者方便一些吧。

注意，停止再启动后，服务器的 `IP` 会变，所以可能引发的现象是：原来的域名提示连接超时；通过 `XSHELL` 等第三方 `ssh` 连接不上服务器，`Connection Failed...`。本质就是服务器的 `IP` 变了。

## 新建用户

用 Github 白嫖的 DigitalOcean 服务器默认是 root 用户，太可怕了。因此新建一个用户，并进行 ssh 的配置。且看：

```bash
sudo adduser xialing # 新建一个叫 xialing 的用户
sudo usermod -aG sudo xialing # 把用户加到 sudo 组
```

这样的话，每次使用 sudo 还需要输入密码，不好，因为已经可以通过 ssh 登录了（见下），所以不想输入密码。

使用 `sudo visudo` 修改 `visudo`，其实用 `vim` 修改 `/etc/visudo` 也可以。在文件的 **最后一行** 添加 `xialing ALL=(ALL) NOPASSWD: ALL`。

通过 ssh 登录，使用 `XShell` 工具选项卡的密钥对生成工具，把公钥复制到 `~` 下的 `.ssh` 下的 `authorized_keys` 文件中，如果没有的话，新建这个文件夹。

完毕。

## 未完待续...
