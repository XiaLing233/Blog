---
title: Latex教程学习笔记
date: 2024-07-09
updated: 2024-07-09
"categories": Latex
permalink: 2024/07/learn-latex-1/
---

# 学习链接

* [Part 1](https://www.overleaf.com/latex/learn/free-online-introduction-to-latex-part-1)
* [Part 2](https://www.overleaf.com/latex/learn/free-online-introduction-to-latex-part-2)
* [Part 3](https://www.overleaf.com/latex/learn/free-online-introduction-to-latex-part-3)
* **本文最好配合链接中的ppt食用**

# 第一部分：基础

## 态度的转变

* 用指令描述“这段文字是什么”，而不是“它应该长什么样”
  * 这个观点很发人深省
* 注重内容
* 让Latex进行排版工作

## 文章的起点

在写每一篇文章之前，都需要指明这篇文章的类型，用`\documentclass{}`

## 特殊字符

* 要显示单引号，形如：``text'`；
* 要显示双引号，形如：```text''`
* 要打印字面`%`, `#`, `&`, `$`，需要在字符前加上\来进行转义，或者理解为指令

### `$`符号——表示数学公式

$符号在Latex中用来表示数学公式，使用时要配对，会自动忽略公式中的空格

* `^`表示上标
  * 如`2^3`
* `_`表示下标
  * 如`F_n`
* `{}`把公式组成一组
  * 如`F_{n-1}`
* 用`\`开头还可以指明希腊字母
  * 如`\mu`
* `$$`包裹的是行内公式，如果要打印比较大的行间公式，使用`\begin{equation}`和`\end{equation}`

## 环境

刚刚提到的`\begin{equation}`和`\end{equation}`之间的部分就是一个环境。

在环境当中，一些成分的显示可能会有不同，如`\sum`指令显示的`大\sigma`会更大，上下标的位置也会有不同。

有一些有用的环境，如`itemize`和`enumerate`。前者创建一个无标号的列表，后者创建一个有标号的列表。

## 包

上面提到的命令和环境都是`Latex`自带的，除此之外，还可以使用其他的宏包。

## 再看数学公式

这次，使用`amsmath`这一宏包下的指令写一些数学公式。

* 使用略有不同的`\begin{equation*}`，可以让行间公式不参与编号。

* 使用`\min`，`\max`等指令可以让公式更好看。可以用`\operatorname`指定自己的变量名，达到和`min`一样的效果。
* 要对齐等式，使用`align*`环境，&符分隔了=两侧，指明了对齐的参考系。

# 第二部分：有结构的文档与更多

## 标题和摘要

* 在导言区创建标题、作者、日期等内容，注意，这里只是声明了相关内容，并没有实际创建（回忆C语言中关于声明和定义的区别）。如果要创建标题，需要在`document`环境中使用`\maketitle`命令。
* 在`document`环境中，用`\begin`与`\end`创建`abstract`环境。以后为了行文方便，简写为创建某环境。

## 分节

* 使用`\section`和`\subsection`即可
* 命令的结尾带上`*`，意思是在目录中不显示该节，和数学公式带`*`的含义相同

## 标签与引用

* 用`\label`指明一个标签，用`\ref`在行文中引用标签，这部分内容会被替换为标签对应的小节号，或者用`amsmath`的命令`\eqref`来引用公式。
  * 使用的时候，先指明`\section`或`equation`环境，再在下一行划定`\label`

## 图片

* 需要使用`graphicx`宏包
  * 使用宏包带的`\includegraphicx`指令，这里，`[]`与`{}`不同，方括号表示可选参数，可写可不写，有点像C++中带默认参数的函数
  * `\documentclass`命令也允许可选参数，如文字大小，纸张大小等
  * 图片要在`figure`环境中插入
  * 可以在`figure`环境中指定`\caption`，有说明文字的图片也可以加上标签来引用(`\ref`)

## 表格

* 需要使用`tabular`宏包
  * 使用`tabular`环境，参数指定了每栏的对齐方式：左**l**对齐或右**r**对齐
  * 在参数部分可以用`|`指定垂直线，在每栏的末尾用`\hline`可以画水平线
  * 用`&`分隔每栏，用`\\`另起一行，和数学公式的对齐是不是很类似？

## 参考文献

1. 把`bibtex`格式的引用放到`.bib`文件中
2. 每个引文的_key_用来在正文中指向这篇引用
3. 可以使用`natbib`包来引用文献，使用`\cite`也行
4. 在文末用`\bibliography`命令生成引用，还可以指定参考文献方式`\bibliographystyle`

## 更多

* `\tableofcontents`生成目录
* 可以更改文章的类型为
  * `\documentclass{scratcl}`
  * `\documentclass[12pt]{IEEEtran}`
* 用`\newcommand`生成自己的命令
* 包
  * `beamer`生成ppt
  * `todonotes`待办事项
  * `tikz`更好的图片

## 安装Latex

* 在Windows或Linux上的环境：TEXLive

# 第三部分：不只是论文

## 幻灯片

* 用`beamer`包
* 文档类型为`beamer`

### 标题页

* 设定的信息有所不同
  * `\title`
  * `\author`
  * `\institute`
  * `\date`
* `\titlepage`来生成标题页

### 一页Frame

* 一页幻灯片就是一个`frame`环境
  * 用`\frametitle`给`frame`一个标题
  * 添加内容和文档中的方式一样

### 小节

* 就像Powerpoint一样，也可以用`\section`来新增节

### 分栏

* 用`columns`环境包含许多个`column`环境来分栏，`column`环境的参数决定了栏的宽度
* `multicol`包可以自动分栏

### 强调

* 用`\emph`或者`\alert`来强调，前者是斜体，后者是红色
* 加粗或者斜体，使用`\textbf`或者`\textit`
* 指定颜色，使用`\textcolor{COLOR}{CONTENTS}`(美式拼写)

### 图表

图表的方式，和文档的一致

### 块

* 创建`block`环境可以让幻灯片的一页上出现有标题的组织块

### 主题

* 使用`\usetheme`来自定义主题
* 在[这个链接上](https://deic.uab.es/~iblanes/beamer_gallery/index_by_theme.html)展示了许多可供选择的主题

### 动画

* 一个`Frame`可以产生许多幻灯片
* 用`\pause`命令来只显示一页的一部分
* `\only`, `\alt`, `\uncover`也是很有意思的命令

## 用`tikz`作图

需要在`tikzpicture`环境中

### 坐标

* 默认的坐标是厘米
* 标定起止坐标，可以画出方格辅助线

### 线

* 用`->`等类似的参数，可以画出箭头，并指定虚实

### 路径

* 用多个点连接起来可以画出一条路径，命令具体的形式是`--`
* `cycle`用来封闭一个回路

### 颜色

* 颜色也是`\draw`命令的可选参数

### 图形

* `tikz`有内置的简单图形，如circle或者rectangle

### 结点

* 用`\node`在一点上放置简单文本
* 并可以用结点名来代替坐标

### 图像

略

