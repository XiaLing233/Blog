---
title: Git学习-2
date: 2024-02-14
"categories": Git
---
# 序
书接上回，上次我们说了 Git 在本地的一些命令，本次我们看看 Git 和 Github 是如何联结在一起的。[学习链接](https://docs.github.com/en/get-started/getting-started-with-git)

# 准备工作
## 安装 Git
请参见[上一篇文章](https://xialing233.github.io/2024/02/14/Learning-Git-1/)，这里从略。

## 设置 Git 的用户名
这里在之前文章的基础上有了拓展，既可以设置全局用户名，也可以为每个仓库设置单独的用户名。命令分别如下：  
```$ git config --global user.name "Mona Lisa"```  
```git config  user.name "Mona Lisa"```

## 缓存 Github 凭证
下载Github CLI 即可，跟随文档无脑走。

```cmd
fatal: unable to access `<link>`, Failure when receiving data from the peer.
```
这种报错和登录凭证关系不大，是代理设置的问题，参见:[链接](https://blog.csdn.net/qq_17229141/article/details/134484804)。

## 远程仓库
Git 用仓库名关联一个仓库的链接。默认的远程仓库名称通常为 ```origin```。远程仓库仅仅是一个名字，不是本质，还不能起到远程仓库的效果，改成"张三"也成。如果要使用 ```pull``` 或者 ```push``` 等命令，则需要进行设置，如在 push 命令中设置：```git push --set-upstream origin main```，意思是，将要 track 的分支设置为 origin 这个地址中的 main 分支。**upstream** 上游，就是这个意思。

* 创建远程仓库  
创建一个远程仓库，在 **项目的目录** 用 ```$ git remote add origin <REMOTE_URL>```，可以通过 ```$ git remote -v``` 查看 remote 的信息。

* 更改远程仓库的URL  
具体的语法为，```$ git remote set-url <name> <link>```

* 重命名仓库  
```$ git remote rename <origin> <destination>```

* 删除仓库  
```$ git remote rm <repo name>```  
只是删除了 repository 的引用。

## 设置文本编辑器
Git 默认的文本编辑器是 vim，一开始可能不太好上手，这里可以通过命令设置为 VS Code：  
```git config --global core.editor "code --wait"```

# Git 大纲
## 概念理解
* repository - 是存放代码的地方。是一个 Git 项目；
* commit - 是文件按时间线的快照；
* branch - commits 可以被分为许多开发的分支，叫 branch；
* Github - 是 Git repositories 的托管平台，并提供方便的研发工具；
* pull request - 简称 PR，一个合并分支的请求，通常发生在将研发完成的新功能分支合并到主分支的过程中。

## 主要命令
* ```git init``` 初始化一个 git repository；
* ```git clone``` 从URL或者本地克隆一个建成的 git repository；
* ```git add``` 将修改或新建的文件添加到 staging area，标志着项目进行到了一个阶段；
* ```git commit``` 保存快照，完成版本记录的进程；
* ```git status``` 显示文件的状态，如 untracked, modified, staged；
* ```git branch```  和分支相关的命令，参见[上一篇文章](https://xialing233.github.io/2024/02/14/Learning-Git-1/) **管理分支** ；
* ```git merge``` 将不同开发路线融合在一起；
* ```git pull``` 从 remote 更新内容；
* ```git push``` 向 remote 中更新内容。

有几个例子，我照搬过来。注意上传到 Github 要用到什么命令。  
## Example 1: Contribute to an existing repository
```cmd
# download a repository on GitHub to our machine
# Replace `owner/repo` with the owner and name of the repository to clone
git clone https://github.com/owner/repo.git

# change into the `repo` directory
cd repo

# create a new branch to store any new changes
git branch my-branch

# switch to that branch (line of development)
git checkout my-branch

# make changes, for example, edit `file1.md` and `file2.md` using the text editor

# stage the changed files
git add file1.md file2.md

# take a snapshot of the staging area (anything that's been added), -m means using "my snapshot" as commit message
git commit -m "my snapshot"

# push changes to github
git push --set-upstream origin my-branch
```

## Example 2: Start a new repository and publish it to Github
```cmd
# create a new directory, and initialize it with git-specific functions
git init my-repo

# change into the `my-repo` directory
cd my-repo

# create the first file in the project
touch README.md

# git isn't aware of the file, stage it
git add README.md

# take a snapshot of the staging area
git commit -m "add README to initial commit"

# provide the path for the repository you created on github
git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPOSITORY-NAME.git

# push changes to github
git push --set-upstream origin main
```

## git push
* 基本格式  
```$ git push REMOTE-NAME BRANCH-NAME```

* 重命名分支  
```$ git push REMOTE-NAME LOCAL-BRANCH-NAME:REMOTE-BRANCH-NAME```  
这样本地分支 ```LOCAL-BRANCH-NAME``` 被 push 到了 ```REMOTE-NAME``` 但是名字变成了 ```REMOTE-BRANCH-NAME```

如果本地副本的版本落后于 repository，则会在 push 时收到错误：```non-fast-forward updates were rejected```，此时需要从 repository 中 pull 更新的部分。

* Push tags(从略，照搬)  
可以使用 ```git push REMOTE-NAME TAG-NAME```

* 删除分支  
```git push REMOTE-NAME :BRANCH-NAME``` 告诉 Git，向 ```BRANCH-NAME` 中发送 _nothing_ 。

## 获得修改
这里再次辨析 ```pull``` ```fetch``` ```clone``` 与 ```merge```。什么都没有的时候，用 ```clone```，直接获得一个拷贝；已经复制完了，用 ```fetch``` 可以先行查看修改，参见上一篇文章；之后，可以通过 ```merge``` 将修改进行合并。而 ```pull``` 可以同时完成 ```fetch``` 和 ```merge``` 两个步骤。

# 跋
作为小白，到此应该可以比较熟练地运用 Git 与 Github 了。在阅读本文的同时，应该[尝试学习官方文档](https://docs.github.com/en/get-started/getting-started-with-git)，并亲自上手创建几个测试项目。祝大家学习顺利。