---
title: 汇编语言学习总结
date: 2024-08-05
updated: 2024-08-05
"categories": Assembly
permalink: 2024/08/assembly-sum-up/
---
本文章是学习王爽老师的《汇编语言 第三版》积累的笔记，**不可以代替原书，只是起到总结的作用**。如果要入门汇编，还是要看王爽老师的教材，阅读完毕后，可以阅读本文章，查漏补缺。

## 寄存器

### 通用寄存器

#### AX

`AX`经常被用于中转，和运算相关。例如，要修改段寄存器的值，不可以使用`MOV, 立即数`，必须将立即数先存到`AX`中，再转移到段寄存器中。常见的一个用法是：

```assembly
; 代码的作用：将数据段地址设置为1000h
MOV AX, 1000h
MOV DS, AX
```

关于其在加减法、乘除法的作用，参见`DX`部分。

#### BX

`BX`和地址相关。如果要通过寄存器间接寻址，`AX`，`CX`，`DX`这三个通用寄存器在语法不支持，必须使用`BX`，这是8086硬件导致的。`BX`间接寻址的用法如下：

```assembly
MOV BX, 1000h
MOV AX, [BX]
```

`BX`默认的数据段是`DS`，也可以使用`ES:[BX]`进行段跨越。8086中，物理地址的构成是段地址+偏移地址，这里BX储存的就是偏移地址，构成真正的物理地址，要进行下列运算：例如`(DS)`=1000H，`(BX)`=0010H，则物理地址(`PA`)=10000+0010=10010H。

`BX`常用在循环`loop`中，作为可以改变的偏移地址。

#### CX

`CX`常作为计数器。`CX`存储了多少，循环就进行多少次。所以每次在执行循环之前，都要指定`CX`的值。看这个简单循环：

```assembly
; 作用：将FFFF:0开始的64字节空间复制到2000:0开始的空间
assume cs:code
    code segment
    
    MOV AX, 0FFFFh ; 0开头，因为汇编不允许字母开头的数字
    MOV DS, AX

    MOV AX, 2000H
    MOV ES, AX

    MOV BX, 0

    MOV CX, 64
s:
	; MOV ES:[BX], DS:[BX] 错误：不可同时对两块内存空间操作！
	MOV DL, DS:[BX]
	MOV ES:[BX],DL
	INC BX
	LOOP s

code ends

end
```

#### DX

`DX`在运算部分很有用，根据运算数的宽度，决定只使用`AX`或者同时使用`AX`和`DX`。就像在字母表的位置上，D比A大一样，`DX`一般存储结果的高位。

### 段寄存器

#### CS（IP）

`CS`的中文名叫指令指针寄存器，它和`IP`（指令指针寄存器）两个是8086最重要的两个寄存器，和指令的执行有关。**注意：IP不是段寄存器，但为了方便，在这里一同介绍。**

8086执行指令，按照`(CS)`*16+`(IP)`的结果来确定执行指令的地址。这个其他地址的确定方法类似，都是**段地址+偏移地址**的形式。上面提到的循环指令得以执行，和`IP`密切相关。

**方法一：**

要修改`CS`和`IP`，当然可以借助某个寄存器，使用`MOV`指令。8086大部分寄存器的值（思考：哪些不行呢。提示：段寄存器。），都可以通过`MOV`改变，`MOV`又被称作传送指令。

**方法二：**

可以使用`JMP`指令。

同时修改`CS`、`IP`的内容，使用`JMP 段地址:偏移地址`，如`JMP 2AE3:3`，执行后，`CS`变成2AE3H，`IP`变成0003H。

只修改`IP`的内容，使用`JMP 某一合法寄存器`，如`JMP AX`，若`(AX)`=1000H，则执行后，`(CS)`不变，而`(IP)`=1000H。

#### DS

`DS`是数据段寄存器，默认用来存放数据的段地址。例如，上面出现过的一个例子：

```assembly
MOV BX, 1000h
MOV AX, [BX]
```

此时，如果`(DS)`=2000H，`(2100H)`=03AEH，则存入`AX`的值为03AEH。

对于数据的访问，可以按字访问或者按字节访问。到底按哪种方式呢？

**方法一：根据目标寄存器的宽度**

如`MOV AX, 255`就是按字访问；`MOV AL, 255`就是按字节访问。

**方法二：强制类型转换**

C语言中有强制类型转换，汇编也有。如`WORD PTR`或者`BYTE PTR`，加到目的前即可。

#### SS（SP）

`SS`是栈段寄存器。在讲`SS`的同时，就把栈顺带说了吧。

栈就是一个杯子，`SS`存储栈段的段地址，`SP`存储栈顶指针的偏移地址。比如，如果把1000H;00-1000H:FF作为栈使用，则`(SS)`=1000H，在栈为空时，`(SP)`=100H。为什么呢，因为当进栈的时候，要先将`SP`的值-2，然后将一个字的数据写入内存。栈顶从高字节开始，逐渐向低字节增长。

有关`POP`和`PUSH`的作用，请参见**常用指令**。

栈顶当然有可能越界，这个是要人为通过高级语言来保证的。换句话说，汇编语言并不能检测栈顶越界的错误。

#### ES

`ES`又叫扩展寄存器，通常是作为扩展段来使用。比如，要从`1000H:0`开始，复制16个字节的内容到`2000H:0`。此时就可以将`DS`和`ES`分别设置为`1000H`和`2000H`，`BX`设置为偏移地址，进行循环操作，每次`(BX)`+1，便可以实现内容的复制，很方便。

### 地址寄存器（BX、BP、SI、DI）

和寻址有关的寄存器，有`BX`、`BP`、`SI`、`DI`四个。如果理解了各自的英文全称，就可以区分它们的功能了

<table>
    <thead>
    	<tr>
        	<th>名称</th>
            <th>全称</th>
            <th>用途</th>
            <th>默认段</th>
        </tr>
    </thead>
    <tbody>
    	<tr>
        	<td>BX</td>
            <td>Base Register</td>
            <td>基址寄存器</td>
            <td>DS</td>
        </tr>
        <tr>
        	<td>BP</td>
            <td>Base Pointer</td>
            <td>通常和访问堆栈有关</td>
            <td>SS</td>
        </tr>
        <tr>
        	<td>SI</td>
            <td>Source Index</td>
            <td>通常作为源的变址</td>
            <td>DS</td>
        </tr>
        <tr>
        	<td>DI</td>
            <td>Destination Index</td>
            <td>通常作为目标的变址</td>
            <td>DS（字符串操作：ES）</td>
        </tr>
    </tbody>
</table>

`BX`和数据段绑定，`BP`和栈段绑定。而关于`SI`和`DI`，就像在段寄存器-`ES`部分阐述过的一样，对于字符串的复制，使用`SI`和`DI`很方便。那他们当然默认的段就是`DS`和`ES`了啊！但是对于一般的内存访问，`DI`的默认段寄存器还是`ES`。

### 标志寄存器

标志寄存器和其他寄存器不同，它每个位都表示各自的含义，组合起来的2`Byte`并没有含义。下面介绍几个常用的标志，这些标志了解即可，因为后续我们不需要对标志进行操作，只需要在逻辑上调用使用了这些标志的指令即可。

#### `ZF`标志

零（_Zero_）标志位。如果指令执行后的结果为0，则`zf`=1，否则为0。

#### `PF`标志

奇偶（_Parity_）标志位。如果结果的所有bit位1的个数为偶数，`pf`=1，否则位0。

#### `SF`标志

符号（_Sign_）标志位。**对有符号数而言**，如果结果为负，则`sf`=1，否则为0。和补码的符号位是不是很相似？

#### `CF`标志

进位（_Carry_）标志位。**对无符号数而言**，如果结果有进位，则`cf`=1，否则为0。

#### `OF`标志

溢出(_Overflow_)标志位。如果**有符号数**超过了机器可以表示的范围，则`of`=1，否则为0。什么叫溢出？什么叫超过表示范围？例如：

```assembly
mov al, 98
add al, 99
; 得到的结果是197
```

得到的结果197无法在8位寄存器中正确表示，因为`al`的表示范围是-128~127。因此，最终得到所谓的98+99=-59，这是不可接受的。

**注意：**`SF`、`CF`、`OF`的改变，和逻辑上正在进行哪种运算无关。比如上面的例子，假设逻辑上我们在执行无符号运算，按理说，我们不在乎`SF`和`OF`的值，但是`SF`和`OF`也可能改变。换句话说，对于机器而言，它操作的数据就是一串二进制数据，得到的结果也是一串二进制数据。在操作过程中和得到的结果中，根据有符号和无符号的两种理解方式，修改标志的值，与人对于数字的理解无关。

#### `DF`标志

方向（_Direction_）标志位。在串处理指令中，控制每次操作后`si`、`di`的增减。和它有关的指令为：

* `std`，_Set Direction_，将`df`=1，递减；
* `cld`，_Clear Direction_，将`df`=0，递增。

#### `TF`标志

陷阱（_Trap_）标志位。`(tf)`=1时，单步调试。

#### `IF`标志

中断允许（_Interrupt Enable_）标志位。当`(if)`=1时，是不可屏蔽中断；否则为可屏蔽中断。


## 常用指令

### MOV

`MOV`叫传送指令，就是把源的值赋给目标。

### ADD

`ADD`是简单加法，很简单。

用法：`ADD AX, BX`，意思是把`AX`和`BX`储存的值相加，并把结果储存在`AX`中。

可以看出，寄存器中存的内容，到底是当做值，还是当做地址，完全看我们用什么样的指令来操作。不同的指令，对于二进制内容会有不同的理解方式。

注意，两个寄存器的位数要对应，和`MOV`一样。

### SUB

和`ADD`的用法类似，不赘述了。

### ADC

`ADC`执行带进位的加法。利用`CF`上记录的进位值。这样，理论上可以实现无穷长度的数据的加法计算。

### SBB

`SBB`执行带借位的减法。和`ADC`的原理类似，藉此可以实现对无穷长度数据的减法运算。

### MUL

* 两个相乘的数，要么都是8位，要么都是16位；
* 如果都是8位，一个默认在`AL`中，另一个放在8位`reg`或者内存单元中；结果默认放在`AX`中；
* 如果都是16位，一个默认放在`AX`中，另一个放在16位`reg`或者内存单元中；结果高位默认在`DX`存放，低位在`AX`中存放。

**注意：**

* 对于内存单元的寻址方式，需要给出数据的宽度，即需要强制类型转换；
* 注意到乘法不支持立即数。

### DIV

`DIV`是除法指令。

* 如果除数为8位，则被除数是16位；除数在一个`reg`或内存单元中，被除数默认在`AX`中；结果`AL`存放商，`AH`存放余数；
* 如果除数为16位，则被除数为32位；除数在一个`reg`或内存单元中，被除数在`DX`存放高16位，`AX`存放低16位；结果`AX`存放商，`DX`存放余数。

### POP

`POP`用来出栈。先将内容写到目标地址，然后将`(IS)`+2。

### PUSH

`PUSH`用来压栈，或者叫进栈。先将`(IS)`-2，然后将内容写到栈中。

可以看到，`POP`和`PUSH`只能一次修改一个字(2Byte)的内容。

### PUSHF与POPF

`pushf`将标志寄存器的值压栈，`popf`将栈中的值压入标志寄存器。它们不接受参数，但可以通过修改栈的值，间接改变标志寄存器的值。

### AND

`AND`用来执行`与`运算，例如，

```assembly
mov al, 01100011B
and al, 00111011B
```

的结果为：`00100011B`。

达成的效果是：将可操作对象的相应位（掩码为0的位）设为0，其他位不变。

### OR

`OR`用来执行`或`运算，例如

```assembly
mov al, 01100011B
or al,  00111011B
```

的结果为：`01111011B`。

达成的效果是：将可操作对象的相应位（掩码为1的位）设为1，其他位不变。

### SHL

`SHL`的作用是左移位，把最高位存储到`CF`标志中，低位补0。

### SHR

`SHR`的作用是右移位，把最低位存储到`CF`标志中，高位补0。

**注意：**

* 在`SHL`和`SHR`的使用中，如果移动的位数为1，可以使用立即数，形如`shl, ax, 1`；

* 如果移动的位数大于1，则需要使用`CL`存储移动的位数。

### CMP

`cmp`指令对两个操作对象进行比较，通过比较的结果修改标志寄存器中对应的值。它通常和条件跳转指令连用。

### LEA

`LEA`指令获得某个标签对应的偏移地址。用法：`LEA BX, 标号`。

### SEG

`SEG`指令获得某个标签对应的段地址。用法：`mov ax, seg datasg`。

## 转移指令

### 循环

在汇编语言中使用循环，需要涉及到`loop`伪指令。

一般的循环结构如下：

```assembly
; Basic Circulation
		mov cx, 10
s: 		add ax, ax
		loop s
```

进入循环之前，先给`CX`赋值，因为`(CX)`决定了循环的执行次数。

循环的标志是标号，标号开头是循环的内容，一直到`loop`前。

`loop`的执行逻辑是：

* 先将`(CX)`-1
* 判断`(CX)`是否不为0，如果为0，则执行`loop`下方语句，否则跳转到标号语句

`loop`通常可以和`[BX]`联合使用，从而方便地进行内容的复制等操作。

注意`loop`只能实现短转移喔！

### 跳转

无论哪种跳转，实现的底层逻辑都是修改`CS、IP`的值。

#### 无条件跳转

无条件跳转的指令是`jmp`，可以只修改`IP`，也可以同时修改`CS`和`IP`。

* `jmp short s`实现的是段内短转移，对`IP`修改的范围是-128~127，机器码中**不包含转移的目的地址，而是位移**；
* `jmp near ptr`实现段内近转移，对`IP`的修改范围是-32768~32767，机器码也包含位移；

**关于位移的计算，是(目的偏移地址)-(源的下一条指令的偏移地址)。**

* `jmp far ptr s`实现远转移，机器码中**包含转移的目的地址**。
* `jmp 2AE3:3`会同时修改`CS`和`IP`的值；
* `jmp AX`会只修改`IP`的值，用寄存器的值覆盖；
* 如果转移地址在内存中，根据宽度的不同，也分成段内转移和段间转移
  * `jmp word ptr 内存单元地址`只修改`IP`的值；
  * `jmp dword ptr 内存单元地址`修改`CS`和`IP`的值，`CS`的值在高位；

有必要对`jmp short s`和`jmp far ptr s`进行辨析。

![摘自王爽《汇编语言第三版》P178](https://static.xialing.icu/img/202408031608099.webp)

一个常考的题目是：已知机器码`EB03`，判断跳转的目标地址；或者已知反汇编的指令`JMP 000B`，判断机器码为`EB??`。

要指出的是，内存当中存放的是跳转的偏移量，例如`03`就是偏移量。那为什么不跳转到`0BBD:0009`，而是`000B`？这就要回顾指令执行的顺序：

1. 从`CS:IP`指向内存单元读取指令，读取的指令进入指令缓冲器；
2. `(IP)`=`(IP)`+所给指令的长度，从而指向下一条指令；
3. 执行指令。转到1重复这个过程。

`EB03`就是根据指令中的`03`修改了`IP`的值。

读取`EB03`这个指令后，`IP`指向下一条指令，偏移地址是`0008H`。之后，再执行`EB03`，把`(IP)`再加3，就指向`000BH`啦！

反过来，如果知道指令是`JMP 000B`，知道`JMP`指令下一条指令的偏移地址是`0008H`，二者相减，得到的就是`03`啦！

但是对于`jmp far ptr s`来说，注意到机器码中直接包含了段地址和偏移地址。需要注意，段地址在高位置，并且在一个字中，按照高放高来安排字节的存储。谁在高位？按照最左侧一列的排布，原来一行中右侧的内容反而在高位。

![摘自王爽《汇编语言第三版》P181](https://static.xialing.icu/img/202408031617726.webp)

#### 条件跳转

* `jcxz 标号`，如果`(cX)`=0，则跳转；
* `loop 标号`，如果`(CX)`!=0，则跳转。

这两个指令可以一起记忆。**所有的条件跳转都只能实现短转移**。

* 检测比较结果的条件跳转指令（通常和`cmp`连用）

<table>
    <thead>
    	<tr>
        	<th>指令</th>
            <th>含义</th>
            <th>检测的相关标志位</th>
        </tr>
    </thead>
    <tbody>
    	<tr>
        	<td>je</td>
            <td>等于则转移</td>
            <td>zf=1</td>
        </tr>
    	<tr>
        	<td>jne</td>
            <td>不等于则转移</td>
            <td>zf=0</td>
        </tr>
    	<tr>
        	<td>jb</td>
            <td>低于则转移</td>
            <td>cf=1</td>
        </tr>      
    	<tr>
        	<td>jnb</td>
            <td>不低于则转移</td>
            <td>cf=0</td>
        </tr>    
    	<tr>
        	<td>ja</td>
            <td>高于则转移</td>
            <td>cf=0且zf=0</td>
        </tr>         
    	<tr>
        	<td>jna</td>
            <td>不高于则转移</td>
            <td>cf=1或zf=1</td>
        </tr>         
    </tbody>
</table>

### 函数调用

* `ret`把`IP`的值出栈，实现近转移；
* `retf`把`CS`和`IP`的值进栈/出栈，实现远转移。注意`CS`的值在高位。
* `call`不能实现短转移，以下几种转移方法，可以和`jmp`的方法类比
  * `call 标号`：令`IP`进栈，实现近转移（-32768~32767），仍然注意内存的存放问题；
  * `call far ptr 标号`，令`CS`和`IP`先后进栈，实现段间转移；
  * `call 16位reg`，压栈`IP`，用寄存器的值修改`IP`的值；
  * 转移地址在内存中：
    * `call word ptr 内存单元地址`，相当于实现近转移；
    * `call dword ptr 内存单元地址`，相当于实现段间转移。

## 环境配置

### 系统参数

为了防止提供的文件污染环境，选择使用VMware的虚拟机来运行。

* 系统版本：Windows XP SP2 64bit

* 虚拟机：VMware Workstation 17 Pro

多说一句，选XP是因为，支持运行DOS和masm，并且虚拟机的速度相对比较快。

### 所需软件

* DOSBox0.74-2-win32
* masmplus
* DEBUG32
* DEBUG

注：王爽老师的汇编语言教材使用的是`DEBUG`，而学校提供的是`DEBUG 32`。二者差不多，指令也差不多，但是`DEBUG32`不支持`MOV AX, [0]`这种指令，即指令中不能出现中括号。[下载地址](http://122.51.85.215/software/Assembly/)

### 具体步骤

1. 关闭防火墙
2. 双击安装DOSBox，地址可以用默认的，反正是虚拟机
3. 双击安装masmplus，如果发现显示乱码，可能是当前系统不支持中文，自行检索给对应系统安装中文的办法
4. 把DEBUG或DEBUG32放到`masmplus的安装目录/Project`下。例如，我的`masmplus`安装在`C:/Assembly/masmplus`下，则把两个DEBUG放到`C:/Assembly/masmplus/Project`下。
5. 打开DOSBox，输入`mount c: C:/Assembly/masmplus/Project`，每次重新打开都要输入。
6. 输入`c:`，切换到`c:`盘符，输入`debug`或`debug32`进入调试。
7. 使用`asmsplus`编写文件：`文件`->`新建`->`MASM工程`->`第一个DOS EXE`->`确定`->`修改文件名，注意以.asm结尾`->`Save`。最后会得到一个默认程序。可以完全删掉，因为默认程序的语法和王爽的书上的语法略有不同，后者也可以在DOS和masm的环境跑起来。

![](https://static.xialing.icu/img/202407312344601.webp)

![](https://static.xialing.icu/img/202407312344864.webp)

8. 编写好文件后，点击`编译`->`编译(ASM)`->`连接(OBJ)`，最终得到了一个同文件名，扩展名为`.exe`的文件。

![](https://static.xialing.icu/img/202407312347927.webp)

9. 要运行，在DOS下输入`文件名`即可
10. 要在`Debug`下调试，在DOS下输入`Debug 文件名`即可

### Debug的使用

Debug 有几个比较常用的指令：

<table>
    <thead>
    	<tr>
        	<th>命令</th>
            <th>功能</th>
            <th>助记</th>
        </tr>
    </thead>
    <tbody>
        <tr>
        	<td colspan="3">查看与修改内容</td>
        </tr>
        <tr>
            <td>r</td>
            <td>查看寄存器的内容</td>
            <td><strong>R</strong>egister</td>
        </tr>
        <tr>
        	<td>d</td>
            <td>查看一块内存空间的内容</td>
            <td><strong>D</strong>ump</td>
        </tr>
        <tr>
        	<td>e</td>
            <td>修改一块内存的内容</td>
            <td><strong>E</strong>dit</td>
        </tr>
        <tr>
        	<td colspan="3">程序的执行</td>
        </tr>        
        <tr>
        	<td>t</td>
            <td>单步执行某个程序</td>
            <td><strong>T</strong>race</td>
        </tr>        
        <tr>
        	<td>g</td>
            <td>跳转到某个偏移地址执行指令</td>
            <td><strong>G</strong>o</td>
        </tr>
        <tr>
        	<td>p</td>
            <td>执行循环，直到cx为0为止</td>
            <td><strong>P</strong>trong</td>
        </tr>
        <tr>
        	<td colspan="3">汇编相关</td>
        </tr>            
		<tr>
        	<td>a</td>
            <td>选定一块内存空间，进行汇编程序的编写</td>
            <td><strong>A</strong>ssemble</td>
        </tr>      
        <tr>
        	<td>u</td>
            <td>将内存的内容反汇编</td>
            <td><strong>U</strong>nassemble</td>
        </tr>
    </tbody>
</table>

具体的使用方法，这里就不赘述了，请参见王爽的教材。

## 在编辑器中编写一个`.asm`程序

示例使用的是masmplus，使用其他编辑器也可以。

程序的基本框架如下：

```assembly
; An example program

assume cs:codesg

datasg segment
	db 20 dup(?)
datasg ends

stacksg segment
	db 16 dup(?)
stacksg ends

codesg segments
start:	mov ax, datasg
		mov ds, ax
		mov ax, stacksg
		mov ss, ax
		
        mov cx, 10
s:		add ax, ax
		loop s
		
		mov ax, 4c00H
		int 21h
codesg ends

end start
```

其中，`datasg`、`stacksg`、`codesg`分别代表数据段、栈段以及代码段。因为一个段最大只能占64KB，因此把代码分段进行处理可以让看代码的人更容易理解、同时程序在执行过程中更不容易出错。

`assume`把寄存器和对应的段关联起来，我们不关心它具体的作用，就把它当成汇编源程序编译之前的必要吟唱吧。

`start:`表明了程序执行的第一条语句，`IP`会指向它。如果代码段的第一条语句并不是第一条想要执行的语句，就需要加上`start:`，同时，在`end`后也要把`start`写上。这里，`start`只是一个名字，换成`aa`也是可以的，只需要确保和`end`后面的名字对应。

注意，上边提到的`datasg`等段名仍然只是一个标号。如何让CPU知道各个段的含义呢？

* 对于代码段，`start:`标号指向的语句为程序的入口，`CS:IP`会指向这条语句的首地址，也就让其所在的`codesg`成为了代码段；
* 对于栈段和数据段，通过在代码段中，把段名送入对应的段寄存器中，就实现了寄存器和段的关联。

## 寻址

寻址的方法列举如下

* 立即数：`MOV AX, 1000H`，等价于`(AX)`=1000H。之前没有说过这个符号，`(寄存器名/地址)`指的是这个寄存器或地址存储的值。
* 寄存器：`MOV AX, BX`，等价于`(AX)`=`(BX)`。注意位数到对应，8bit对8bit，16bit对16bit。**错误示例**：`MOV AX, BL`。
* 直接寻址：`MOV AX, [1000H]`。如果`(1000H)`=2345H，则该语句使得`(AX)`=2345H。这里要说明，在`Debug`中，[0]理解为DS:0，而在`masm`中，[0]理解为数字0。
* 直接寻址——数据标号法：`MOV AX, VAL`。如果`(VAL)`=2345H，则该语句使得`(AX)`=2345H。也要注意位数的对应。
* 间接寻址：`MOV AX, [BX]`。如果`(BX)`=2345H，则该语句使得`(AX)`=2345H。注意，几个通用寄存器中，只有`[BX]`可以用来间接寻址。还可以间接寻址的有：`BP`，`SI`，`DI`，它们默认的段寄存器不全相同。
* 相对寻址：`MOV AX, VAL[COUNT]，(AX)=(DS:(VAL+COUNT))`。
* 基址变址寻址：`MOV AX,[BP][DI]`，`(AX)`=`(SS:(BP+DI))`。两层括号，强调存进去的是地址存储的值，而不是地址本身。且必须一个基址+一个变址，不可以两个寄存器类型相同。
* 相对基址变址寻址，是上边两个的结合，可以类比。`MOV AX, VAL[BI][SI]`

这几种寻址方法，各有各的应用场景。到底采取哪种寻址方法，和寻址的要求是密切相关的。换句话说，如果需要一个变量，`[BX]`就足够了，如果要两个变量，可能就需要`[BX]和[DI]`共同起作用，如果还需要一个常量，那就需要`idata`。这里不去阐述到底哪种数据类型适合哪种寻址方式，只是点到为止。**但需要强调的是，如果要采取基址变址的寻址方式，一定是一个基址寄存器+一个变址寄存器的形式！**

## 伪指令

### `db`、`dw`、`dd`

* `db`用来定义字节型数据，_Define Byte_
* `dw`用来定义字型数据，_Define Word_
* `dd`用来定义双字型数据，_Define Dword_

### `dup`

`dup`用来进行数据的重复。

如`db 3 dup (0)`定义了三个字节宽度的内容，每个字节的内容都是0。

### `offset`

`offset`用来计算标号的偏移地址，相对于谁的偏移地址呢？相对于段地址的！哪个段地址？写在`assume`伪指令里的。

看一个有些难度的例子：

```assembly
assume cs:codesg
codesg segment
s:		mov ax, bx			;mov ax, bx 的机器码占两个字节
		mov si, offset s	;si一般存储源偏移地址
		mov di, offset s0	;di一般存储目的偏移地址
		mov ax, cs:[si]		;思考：为什么要用ax作为中转
		mov cs:[di], ax		;答案：因为不可同时对两块内存操作
s0:		nop					;nop的机器码占一个字节
		nop
codesg ends

end s
```

这段代码的作用，是把第一条指令复制到`s0`标号处。

### 数据标号

之前的程序，使用的是地址标号来表示数据的地址：

```assembly
;例子摘自王爽《汇编语言 第三版》P287
assume cs:code

code segment
	a: db 1, 2, 3, 4, 5, 6, 7, 8
	b: dw 0

start:	mov si, offset a
		mov bx, offset b
		mov cx, 8
s:		mov al, cs:[si]
		mov ah, 0
		add cs:[bx], ax
		inc si
		loop s
		
		mov ax, 4c00h
		int 21h
		
code ends
end start
```

在上面的例子中，使用标号`a`、`b`指明了代码段中一些数据存放的地址。通过`offset`伪代码可以计算出对应地址，进行数据的操作。

**但如果去掉`:`呢？**

```assembly
;例子摘自王爽《汇编语言 第三版》P288
assume cs:code

code segment
	a db 1, 2, 3, 4, 5, 6, 7, 8
	b dw 0

start:	mov si, 0
		mov cx, 8
s:		mov al, a[si]
		mov ah, 0
		add b, ax
		inc si
		loop s
        
		mov ax, 4c00h
		int 21h
		
code ends
end start
```

此时的`a`、`b`具有两层含义：

1. 表示内存地址
2. 表示单元长度

例如，`a`标号表示了8个数字中第一个数字的字节地址，并且指明了每个数据占一个字节。

通过这样的方法，寻址表达更加简洁。

**问题是，计算机怎么知道一个数据编号到底代表哪里？换句话说，有多个段的情况，应该如何处理？**

知道偏移量，相对于哪个代码段的偏移量呢？编译器怎么知道？CPU又怎么知道呢？

* 对于编译器问题的解答：编译器通过`assume`伪指令把寄存器和代码段关联起来。例如，`assume cs:codesg`让编译器认为，`codesg`的段地址存放在`CS`中。
* 对于CPU问题的解答：CPU通过访问对应段寄存器的值寻找对应的段地址。

举个例子，

```assembly
; 自己想的例子
assume ds:datasg, cs:codesg

datasg segment
	a db 10 dup(?)
datasg ends

codesg segment
start:	mov ax, datasg
		mov ds, ax
		
		mov dl, a
codesg ends

end start
```

当写下`mov dl, a`的时候，CPU在编译器的帮助下，理解的指令是这样的：

`mov dl, ds:a`，也是`mov dl, ds:[0]`

* 编译器做了什么？在编译的时候，看到标号`a`。”`a`在哪里？“，编译器问道。由第二行的伪指令`assume`，它知道，应该到数据段中寻找，把`a`标号的段地址理解为数据段的；
* CPU做了什么？他通过编译器的翻译，知道了，`a`的偏移地址要到`DS`中寻找，把我们写的源程序理解为`mov dl, ds:a`；
* 我们应该做什么？写好`assume`伪指令，把寄存器和数据段对应正确，让编译器理解清楚；给对应的段寄存器通过`mov`指令正确赋值，使得CPU能够访问到正确的段地址。

### 直接定址表

如果要把内存当中的`0-F`输出为字符`0-F`，有什么好的方法吗？

当然可以注意到，数字`0-9`和字符`0-9`，数字`A-F`和字符`A-F`之间各自有对应的线性映射。但问题是，这两个映射并不是同一个映射，一个是`+30H`，另一个是`+37H`。怎么才好？分类讨论不够方便，如果能建立一个表格就好了。

`table db 0123456789ABCDEF`

这样，根据数字的偏移量，就可以通过`table[bx]`来访问对应的字符啦！这就是一个通用的线性映射。

## 中断

这里，我们只关注内中断。本来中断就不是授课内容，所以这部分是感兴趣才阅读下来的。和外中断相比，还是内中断更重要一些。为什么呢？因为理解了中断，才能明白程序最后的两句：

```assembly
mov ax, 4c00h
int 21h
```

到底是什么意思啊！

关于中断的总结，从略，总结的目的是为了理解上面两行代码。

中断的过程是：

1. CPU收到中断信息
2. 将`标志寄存器`压栈，设置`CS`和`IP`，并且置`TF`和`IF`为0
3. 根据中断向量表，寻找到相应中断编号的中断处理程序入口地址
4. 中断处理程序开始运行

**理解这两行的背景知识：**

* `AH`存放调用的子程序
* `AL`存放参数
* `int`意思是调用中断处理程序

所以，这两行的意思是：

* 传递`4c00h`给`AX`
* 调用`21h`号中断处理程序的`4c`号子程序
* 根据`al`的值，返回值为0