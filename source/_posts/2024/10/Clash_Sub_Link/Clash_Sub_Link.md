---
title: Clash 订阅转换无法获取结点
date: 2024-10-08
permalink: 2024/10/Clash_Sub_Link/
---

## 问题

市面上有许多常用的订阅转换程序，可以让对于机场自带的分流规则不满意的用户实现自己的规则。这一结果的实现有赖于 `subconverter` 这一开源程序。然而，有些机场禁用了 `subconverter`，使得其无法获取到结点。那么该如何解决呢？

## 现象

输入订阅链接以及自定义的规则后，在 `Clash` 中，提示下载失败；在浏览器中可以查询到更详细的信息。

![订阅转换样例](https://static.xialing.icu/img/202410080956325.webp)

在浏览器中输入获得的整合链接，得到提示：

>The following link(**以下简称：整合链接**) doesn't contain any valid node info: `https://api.xxx.top/sub?target=clash&new_name=true&emoji=true&clash.doh=true&filename=XXX&udp=true&url=https%3A%2F%2Fapi.xxx.xyz%2Fosubscribe.php%3Fsid%3Dxxxxx%26token%3Dxxxxxxxxxx`

翻译过来，就是对应的链接没有有效结点。

## 猜测 1：真的没有有效结点吗?

赋值该链接到浏览器中，发现下载到了有效的结点信息。

## 猜测 2：某些订阅转换的前端被列入黑名单

市面上有不少订阅转换的前端页面，都是套皮。例如下列的这几个

* [sub.dler.io](https://sub.dler.io/)
* [sub-web.wcc.best](https://sub-web.wcc.best/)
* [sublink.dev](https://sublink.dev)

等等等等...无法一一列举。

有这样多的域名，有些机场主为了防止攻击，将某些 `API` 列入白名单：

> 由于API时常被攻击，目前需要直连导入|更新订阅或使用我们的节点进行。由于被攻击，目前也无法使用使用第三方API，教程 FAQ 内的 API 为白名单。

是不是因为我没有用白名单中的订阅链接呢？否！官方推荐的转换服务仍然复现出错误现象。

## 猜测 3：链接编码不对?

注意到**整合链接**的 `url=` 部分采用了 `url` 编码，例如 `:` 是 `%3A` 等等。如果我把编码复原成字符，是不是就正确了呢？

答案是，报错略有不同，但仍然无法获取到有效结点。

## 猜测 4：白名单不够白

真是奇怪了，为什么在浏览器能获取到结点，但是用**整合链接**就访问不到？我试试自建的服务吧！

在 [subconverter](https://github.com/tindy2013/subconverter/releases) 的 `Github` 仓库下载了 `.exe` 文件，双击运行。把前端页面（也就是**现象**部分的那张截图）的后端地址改成 `http://127.0.0.1:25500/sub?`，获取连接。您猜怎么着——仍然无法获得结点，错误提示不变。

## 正题：坎坷的搜集资料与解决

这下奇了，为啥子呢？

以 `The following link doesn't contain any valid node info` 为关键词在 `Google` 检索到：

[Github 上 subconverter 的 issue](https://github.com/tindy2013/subconverter/issues/209)。开发者回应是白名单的问题，这个问题我已经排除。

不看浏览器了，看看 `subconverter` 后端的提示信息吧。发现提示了 `No nodes were found!`。有点不同，但其实本质问题是一样的。不过还是以此为关键词吧，检索到：

[仍然是 subconverter 中的 issue](https://github.com/tindy2013/subconverter/issues/579)，不过日期更近了。有大佬回复多半是[请求头的问题](https://github.com/tindy2013/subconverter/issues/573#issuecomment-1436012060)，因此考察请求头。

> 某些时候，有些订阅提供者会根据请求头屏蔽部分请求，`subconvertor` 带有 `SubConverter-Request` 和 `SubConverter-Version` 的请求头，而且目前似乎没有提供更改请求头的方法，目前好像无解。
>
>或者你可以自己修改源代码然后编译。

根据对照试验的思路：浏览器可以请求，`subconverter` 无法请求，对比 `Request Header`，发现 `subconverter` 的请求多出来了上述两个请求头，评论区里也有大佬给出了注释掉这两条语句后的可执行文件。我也想自己动手来着，但是确实在 `Windows` 或者 `Linux` 下配置 `CMAKE` 环境有点困难，我就坐享其成了。

最后当然成功了。在 `Clash` 中可以获取到以自定义规则组织的结点信息。

## 其他

在这过程中，有大佬建议修改 `.toml` 文件中的代理条目，具体[参见](https://github.com/tindy2013/subconverter/issues/579#issuecomment-1587796982)。我一开始修改了，仍然不成功。下载别人编译好的文件，发现这个条目并没有被修改。因此要是哪天发现转换不成功了，可能这里需要再修改下吧。目前还能跑，就不动它了。