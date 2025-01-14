---
title: subconverter 后台服务运行
date: 2025-01-14
updated: 2025-01-14
permalink: 2025/01/subconverter-background-run/
categories: Network
---

## 需求

希望使用自定义规则。subconverter 的 release 版本的请求头有一些字段，导致其被一些机场网络服务商屏蔽（[参见之前的文章](../../../2024/10/clash-sub-link/)），自己编译的版本每次需要手动打开运行，点击 Clash 进行节点更新，太麻烦。因此使用 Windows 服务的形式来让 exe 后台运行。

## 步骤

### 下载

nssm 是一个开源工具，用于将任何 exe 安装为 Windows 服务。下载地址[https://nssm.cc/download](https://nssm.cc/download)，国内访问有点慢。

### 服务注册

把 exe 注册为服务：

```bash
nssm install YourServiceName "C:\path\to\your_program.exe"
```

> **注意**：如果文件名包含空格，需要用双引号括起来。否则执行 `net start` 会报错：发生特定服务错误：3。请键入 NET HELPMSG 3547。如果已经这样做了，可以通过 `nssm edit <service_name>` 来修改路径。

### 启动

```bash
net start <service_name>
```

## 结束

这样应该就可以让节点的定时更新起作用了。当然，还有可能出现无法请求到自定义规则集的情况，到时候再看看吧。
