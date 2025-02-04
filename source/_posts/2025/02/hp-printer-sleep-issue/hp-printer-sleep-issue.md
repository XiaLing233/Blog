---
title: 【已解决】HP 打印机无法休眠
date: 2025-02-04
updated: 2025-02-04
permalink: 2025/02/hp-printer-sleep-issue/
categories: 个人工作
---

## 问题现象描述

HP Color LaserJet Pro MFP M281fdw 打印机无法休眠。在 60s 无操作后触摸屏自动关闭，然后等待几秒钟后屏幕再次亮起。如此反复。同时伴随着细微的咔咔声，很烦人。

## 解决方案

发现打印机的 USB 口和电脑的连接在了一起，同时笔记本电脑处在关闭的状态。这一连接是没有必要的，因为打印机已经连接无线网了。因此，把 USB 线拔掉就好了。问题解决。

## 参考

[https://h30434.www3.hp.com/t5/LaserJet-Printing/Problem-with-sleep-mode-on-my-printer-MFP-M278-M281/td-p/7534286](https://h30434.www3.hp.com/t5/LaserJet-Printing/Problem-with-sleep-mode-on-my-printer-MFP-M278-M281/td-p/7534286)

## 走的弯路

检索到了[同型号的打印机一直开机关机的问题](https://h30434.www3.hp.com/t5/Printing-Errors-or-Lights-Stuck-Print-Jobs/Color-LaserJet-Pro-MFP-M281fdw-turning-on-and-off-and-on-and/td-p/7823441)，通过文中的方法进行 `Permanent Storage Init`，但发现解决不了问题。这也是我两三年前尝试过解决的方案。当时认为这一问题无解，所以止步于此。

之后今天早上又被这一声音整的烦躁，才再次检索。发现是 USB 线的问题。あああー，困扰我许久的问题终于解决了。看来学了计算机两年，解决问题的能力 upupup 呢。

对问题的准确描述很重要。表面上看，是触摸屏反复亮灭，事实上，更准确的描述应该是 Printer won't stay in sleep mode。

Keep learning.
