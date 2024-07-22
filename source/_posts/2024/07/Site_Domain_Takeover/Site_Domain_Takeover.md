---
title: 建站相关|Github Pages被其他用户使用
date: 2024-07-22
"categories": 建站
permalink: 2024/07/Site_Domain_Takeover/
---

## 问题

我的`Github Pages`只绑定了`blog.xialing.icu`这一域名，其他的`@.xialing.icu`以及`www.xialing.icu`没有投入使用，
此前，访问后两个网站，显示无法连接，符合预期。今天访问的时候，突然指向了另一个页面，如下图：
![域名被其他人使用](https://static.xialing.icu/img/202407221744765.webp)
**我没有配置过这个页面，为什么会出现这种问题？**

## 原因推测

1. 是不是域名被劫持了？关掉魔法，发现不管是在美国还是在国内，都会指向该页面，排除运营商干扰；
2. 是不是我在腾讯云DNS解析的配置有误？发现解析指向正确，并没有指向别的网址（这种做法能不能实现博客的访问暂时不谈，至少不会指向别的页面）
![正确的腾讯云配置](https://static.xialing.icu/img/202407221748647.webp)
3. 难道是`Github`的问题？尝试能不能把自定义域名改成`xialing.icu`，发现提示域名已被占用！
![Github的错误提示](https://static.xialing.icu/img/202407221751433.webp)
大概定位到问题所在了，应该是别人把我拥有的域名用于他的`Github Pages`。这还了得！哪能白占我便宜！

## 解决问题

跟随[链接](https://docs.github.com/pages/configuring-a-custom-domain-for-your-github-pages-site/verifying-your-custom-domain-for-github-pages)，找到**Verify**域名的方法，在腾讯云的DNS解析中添加一次TXT记录即可。如下图操作：
![左侧是腾讯云的配置页面，右图为Github帮助文档的示例](https://static.xialing.icu/img/202407221806495.webp)
之后点击`Verify`即可。如此操作后，还要大概等待一周左右，才能把被占用的页面释放。

## 反思

为什么我的页面会被其他人占用？网页上的原文为：
>Domain takeovers can happen when you delete your repository, when your billing plan is downgraded, or after any other change which unlinks the custom domain or disables GitHub Pages while the domain remains configured for GitHub Pages and is not verified.

也就是说，域名被其他人占用，需要具备：

1. 域名未验证(`verified`)；
2. 域名已配置为`Github Pages`；
3. 域名和先前的`Github Pages`解除了关系。

我怀疑，造成我的域名被其他人占用的原因，一个是域名没有验证；再一个是，我曾经使用其他的域名配置过`Github Pages`，但后来又把这个域名解除掉了。这样就满足了以上三点。

对方怎么操作的呢？只要把自己的个人`Github`页面的自定义域名写成我的即可。因为我的域名已经配置好了`Github Pages`。但更细节的原因，由于我欠缺相关知识，不知道如何解释。感兴趣的读者欢迎在知乎私信我。

目前还不能准确排查出来问题的原因是什么，但是如果验证了自己的域名，这种问题一定不会发生。不幸的是，个人建站的许多教程中并没有提到验证域名这一关键步骤，导致有些人乘虚而入。七天之后，我再检验一下，此方法是否有效。