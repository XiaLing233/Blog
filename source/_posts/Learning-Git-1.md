---
title: Git学习-1
date: 2024-02-14
update: 2024-02-14
"categories": Git
---
## 序
Git是一个使用普遍的版本控制工具，这是我[根据Git在线文档](https://www.git-scm.com/docs)的学习笔记。有关 Github 的内容，请期待后续文章。

## 下载
直接点击[官网](https://git-scm.com)的Download按钮即可

## 准备工作
设置用户名与邮箱
```
$ git config --global user.name "Your Name Comes Here"
$ git config --global user.email you@yourdomain.example.com
```
## 导入项目
1. 如果有tar包，可以使用以下命令：

```
$ tar xzf project.tar.gz
$ cd project
$ git init  # 初始化一个git目录
```

2. 记录最初的版本

使用 ```$ git add .``` 可以记录当前目录下的所有内容，这些文件会被储存到一个临时的 staging area，Git 的术语叫 "index"。

3. 最终提交

用```$ git commit``` 语句，可以永久地将 index 中的内容存储到 repository 中。

## 修改项目
修改项目也是同理，总共分三步：
1. 用文本编辑器，或者 IDE 修改文件
2. 用 ```$ git add file1 file2 file3``` 命令将修改的文件添加到 index 中
3. 用 ```$ git commit ``` 命令提交修改，并配上修改的信息

### 注意
* 在 **步骤2.** 后， **步骤3.** 前，可以使用 ```$ git diff --cached``` 查看 index 中要被上传的文件信息；

![--cached 方法](https://s2.loli.net/2024/02/14/G2JoPF3wBYCpySr.png)
* 当然也可以使用 ```$ git status``` 查看更简略的信息；

![status方法](https://s2.loli.net/2024/02/14/ZmgPbtkyuO1G2a8.png)
* 可以使用更简略的 ```$ git commit -a``` 命令一次性地完成 stage 所有被修改的文件（不包含新文件），并 commit，一站式完成；
* 关于 commit 的信息，有两种方式。一种是短的，大约在50个字符以内，用一行完成描述；另一种是较长的，具体格式为，第一行为大标题，第二行为空行，第三行开始时较长的详细描述。

## 查询项目历史
方法1，用 ```$ git log``` 查询，会按由近及远的顺序依次显示修改的简略信息；

![方法1](https://s2.loli.net/2024/02/14/t9gypf3KN4n1BYP.png)

方法2，用 ```git log -p``` 可以查询到具体的修改内容，如对文件内容的修改，增加、删除了哪些行等；

![方法2](https://s2.loli.net/2024/02/14/KTw2zkgO6sHbjp5.png)

方法3，用```git log --stat --summary```，可以看到每次修改的大致内容，如每个文件增减了多少行等等。

![方法3](https://s2.loli.net/2024/02/14/5NtOdaP3FhVMuAj.png)

### 注意
浏览时，用上下方向键可以调整阅读的区域，要退出，依次输入 **wq** 即可（虽然不太确定，不过这似乎是 vim 的语法。在这里，冒号已经输入好了，所以我们只需要输入 wq 两个字符）。

## 管理分支(branch)
### 创建并切换分支
可以通过 ```$ git branch 分支名``` 新建一个分支。

用 ```$ git branch``` 可以浏览现存的所有分支。

用 ```$ git switch 分支名``` 切换分支。（注意 **不是** ```$ git branch switch 分支名```）

#### 注意
在创建分支之前，务必确保曾经 commit 过一次。查询的方法，可以输入 ```$ git branch``` 查看有没有分支，如果列表是空的，则需要 commit。否则会报错：fatal: not a valid object name: 'master'。[参考链接](https://blog.csdn.net/hengyunabc/article/details/6058145)

### 修改分支
修改分支和修改 master 是一样的流程，参见上文 **修改项目** 。

### 融合分支
融合分支，需要在 **master branch** 使用 ```$ git merge 分支名``` 命令，如果两个分支没有冲突，则融合结束。

如果二者有冲突，则需要通过 ```$ git diff``` 命令查看冲突，并直接修改文件夹中的文件。此时要注意，不需要输入其他指令，切换分支，或者退出融合过程，直接修改文件夹中冲突的部分。见下图：

![cmd](https://s2.loli.net/2024/02/14/rVlx5n3ZAe9azHh.png)

![txt](https://s2.loli.net/2024/02/14/yz8gfepGXhq3Wa2.png)  
修改的时候，不只要修改冲突的内容，也要把类似 >>>>>>HEAD 的内容也删除，否则这些内容会保留下来。

可以使用 ```$ gitk``` 查看图形化的 merge 流程，更加直观。

之后就可以用 ```git branch -d 分支名``` 删除实验性的分支了。这种修改，可以保证被删除的分支里的内容完全被融合到 master 中。如果分支中有部分内容未被融合，会提示无法删除。若不顾融合强制删除，则将小写d改为大写，```git branch -D 分支名```。

## 协作(clone, pull, fetch, merge)
乙克隆了甲的项目，如果在本地，用 ```$ git clone 地址 仓库名``` ，内容与原仓库一模一样。  
乙修改了文件，并且 commit 了修改。甲可以通过 ```$ git pull 地址 master``` fetch 并 merge 乙的修改，甲可能要处理一些冲突。

如果甲想先看看乙做了哪些修改，可以用 ```$ git fetch 地址 master``` 获取到更改，并用 ```$ git log -p HEAD..FETCH_HEAD``` 查看乙的修改。```HEAD..FETCH_HEAD``` 意味展示所有 ```FETCH_HEAD``` 中的内容但是排除 ```HEAD``` 中的内容。  
可视化，则可以用 ```$ gitk HEAD..FETCH_HEAD```。  
如果想看看他们两个人共同修改过的内容，则将两个点改成三个点：```$ gitk HEAD...FETCH_HEAD```，这样展示了排除相同文件后，甲乙所有的文件。  


甲还可以用 ```$ git remote add 乙 地址``` 来速写，这样就可以用 ```$ git fetch 乙 ``` 来获取仓库了，而不需要输入一大串地址。相比之下，乙不需要进行这样的操作，因为甲的地址已经在克隆时自动记录了。  
对比内容的命令则变为 ``` $ git log -p master..bob/master```，其他形式的可以类推。  

对于远程克隆，则将地址改为 ssh 协议或者其他协议的地址名即可，只是多了登录的操作。

## 探索历史
上文已经探讨过 ```$ git log``` 的用法，这里更进一步地讨论。

在 ```$ git log``` 中，展示的第一行内容是某个  的名字。如
```
$ git log
commit c82a22c39cbc32576f64f5c6b3f24b99ea8149c7
Author: Junio C Hamano <junkio@cox.net>
Date:   Tue May 16 17:18:22 2006 -0700

    merge-base: Clarify the comments on post processing.
```
中， ```c82a22c39cbc32576f64f5c6b3f24b99ea8149c7``` 就是这次 commit 的名字。

要查询这个 commit，只需要用 ``` $ git show c82a22c39c``` 即可。用更长的名字也无妨，但这个长度已经足够分别出不同的 commit。除此之外，还有一些查询方法，如查询某个分支的修改，查询某个分支的 "parent" commit 等等。这里就不一一列出，可以参见[教程](https://git-scm.com/docs/gittutorial)。

还可以通过 ```$ git tag 标签名 commit名``` 来给 commit 加标签，这个我认为比较有用。

除此以外的指令，大多比较细枝末节，不再一一列出。

## 跋
以上就是本文的全部内容。这些都是 Git 最基础的本地操作，还没有涉及到 Github 在线存储的环节。关于后者，请参见后续文章。