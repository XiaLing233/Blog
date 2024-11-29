---
title: Missing Semester 学习笔记(1)
date: 2024-02-28
"categories": Missing Semester
permalink: 2024/02/missing-1/
---

# 序

[Missing Semester](https://missing.csail.mit.edu/)的第一课，大致介绍了 Shell 的用法。注意，课堂使用的是 Bash Shell，全称为 "Bourne Again SHell"。所以 Windows 自带的 PowerShell 或者命令行窗口不能100%完成命令。

# 笔记

## 使用 Shell

Notes 讲解了几个 Shell 的基本用法：

* `date`  命令可以输出当前日期；
* `echo` 重复我们说过的话，默认输出到屏幕上；
  * 如果参数有空格，则需要用引号（单引号或双引号）括起来；
  * 或者可以使用转义字符 `\`
* 程序通过环境变量知晓 `date` 与 `echo` 程序的位置，Shell 就是一个编程环境，所以叫环境变量。
  * 通过 `echo $PATH` 可以输出环境变量的内容。`$` 形象地表示 PATH 的值；
  * 可以通过 `which` 命令获得某一程序的位置，如 `which echo`；

## 在 Shell 中航行

* `pwd`，打印当前工作的文件夹(present working directory)；

* `ls` ，列出当前文件夹内所有的内容，默认是当前文件夹；

  * 用 `-l` flag 可以打印出更详细的信息，

    ```bash
    drwxr-xr-x 1 missing  users  4096 Jun 15  2019 missing
    ```
  * 第一个字母 `d` 表示当前环境是个文件夹；其后三个由三个字母组成的组合 `rwx, r-x, r-x` 分别表示所有者，所有组，以及所有用户的权限；
  * 对文件而言，显然 `r` 表示读取，`w` 表示写入，`x` 表示执行；
  * 对文件夹而言，`r` 表示获取文件夹所含内容，`w` 表示修改文件夹，`x` 表示进入文件夹。 

* `cd` ，改变当前打开的文件夹，changing directory；

  * `.` 表示当前文件夹，可以通过相对路径打开文件夹；
  * `..` 表示父文件夹。

* `mv` 表示重命名或移动一个文件，`cp` 是复制，`mkdir` 来新建一个文件夹；

* 用 ` man` 更详细地了解一个指令的参数等信息。**man** 表示 **manual** 。

## 连接程序

程序默认的输入输出环境都是终端，可以通过一些符号达到修改流向的目的。

### > 与 <

* `<` 表示修改输入；
* `>`表示修改输出。

#### 一个例子：

```bash
echo hello > hello.txt
cat hello.txt
hello
cat < hello.txt > hello2.txt
cat hello2.txt
hello
```

* 这里，第一行通过 `>` 将 hello 写入到 `hello.txt` 中;
* `cat` 可以`cat`enate 文件，输出文件的内容。这里，用 `>` 把输出路径改成了 `hello2.txt` ，输入路径用 `<` 改为 `hello.txt`。

* 还可以用 `>>` 在末尾附加内容。

### 管道

用 `|` 运算符可以将多个程序连接到一起。这样，一个程序的输入就成了另一个程序的输出。

## 超级用户

`sudo` 表示 **Super User**，拥有一切权限，可以修改一些系统设置。

# 练习

以下练习是在 Ubuntu 虚拟机下完成的。

1. For this course, you need to be using a Unix shell like Bash or ZSH. If you are on Linux or macOS, you don’t have to do anything special. If you are on Windows, you need to make sure you are not running cmd.exe or PowerShell; you can use Windows Subsystem for Linux or a Linux virtual machine to use Unix-style command-line tools. To make sure you’re running an appropriate shell, you can try the command echo $SHELL. If it says something like /bin/bash or /usr/bin/zsh, that means you’re running the right program.
```bash
xialing@xialing-virtual-machine:~$ echo $SHELL
/bin/bash
```

2. Create a new directory called `missing` under `/tmp`.

```bash
xialing@xialing-virtual-machine:~$ cd /tmp
xialing@xialing-virtual-machine:/tmp$ mkdir missing
xialing@xialing-virtual-machine:/tmp$ ls
missing
.
.
```

3. Look up the `touch` program. The `man` program is your friend.

```bash
xialing@xialing-virtual-machine:~$ man touch
NAME
       touch - change file timestamps

SYNOPSIS
       touch [OPTION]... FILE...
.
.
```

4. Use `touch` to create a new file called `semester` in `missing`.

```
xialing@xialing-virtual-machine:/tmp/missing$ touch semester
xialing@xialing-virtual-machine:/tmp/missing$ ls
semester
```

5. Write the following into that file, one line at a time:

```bash
#!/bin/sh
curl --head --silent https://missing.csail.mit.edu
```

```bash
xialing@xialing-virtual-machine:/tmp/missing$ touch semester
xialing@xialing-virtual-machine:/tmp/missing$ echo '#!/bin/sh' > semester
xialing@xialing-virtual-machine:/tmp/missing$ echo curl --head --silent https://missing.csail.mit.edu >> semester
xialing@xialing-virtual-machine:/tmp/missing$ cat semester
#!/bin/sh
curl --head --silent https://missing.csail.mit.edu
```

**Note: **The first line might be tricky to get working. It’s helpful to know that `#` starts a comment in Bash, and `!` has a special meaning even within double-quoted (`"`) strings. Bash treats single-quoted strings (`'`) differently: they will do the trick in this case. See the Bash [quoting](https://www.gnu.org/software/bash/manual/html_node/Quoting.html) manual page for more information.

* 即，如果用双引号，感叹号有特殊含义，会报错：`bash: !/bin/sh: event not found`；
* 如果不用引号，`#` 会将后边的语句注释掉，文档内容为空；
* 所以，应该使用字面输入的单引号。

6. Try to execute the file, i.e. type the path to the script (`./semester`) into your shell and press enter. Understand why it doesn’t work by consulting the output of `ls` (hint: look at the permission bits of the file).

```bash
xialing@xialing-virtual-machine:/tmp/missing$ ./semester
bash: ./semester: Permission denied
xialing@xialing-virtual-machine:/tmp/missing$ ls -l
total 0
-rw-rw-r-- 1 xialing xialing 0 Feb 28 22:32 semester
```

对于文件，没有执行权限 `x` ，因此无法执行文件。

7. Run the command by explicitly starting the `sh` interpreter, and giving it the file `semester` as the first argument, i.e. `sh semester`. Why does this work, while `./semester` didn’t?

[有的网站](https://zacheller.dev/missing-semester0)指出了差别。 `sh semester` 可以忽略执行位。

8. Look up the `chmod` program (e.g. use `man chmod`).

```bash
xialing@xialing-virtual-machine:/tmp/missing$ man chmod
NAME
       chmod - change file mode bits
.
.
```

9. Use `chmod` to make it possible to run the command `./semester` rather than having to type `sh semester`. How does your shell know that the file is supposed to be interpreted using `sh`? See this page on the [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) line for more information.

```
xialing@xialing-virtual-machine:/tmp/missing$ chmod u+x semester
xialing@xialing-virtual-machine:/tmp/missing$ ls -l
total 0
-rwxrw-r-- 1 xialing xialing 0 Feb 28 22:32 semester
```

10. Use `|` and `>` to write the “last modified” date output by `semester` into a file called `last-modified.txt` in your home directory.
