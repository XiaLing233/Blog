---
title: C Primer Plus 第三章编程练习笔记
date: 2024-02-09
updated: 2024-02-19
"categories": C
permalink: 2024/02/cpp-c3-pe/
---

## 前言
最近假期重拾了C Primer Plus，做了做第三章的编程练习题。因为才是第三章，所以难度不是很大，不过有一些小地方还是值得注意的。因为中文版翻译的水平实在难以恭维，而且英文版确实上手不难，所以就用英文版的题目了。

## 题目1
>Find out what your system does with integer overflow, floating-point overflow, andfloating-point underflow by using the experimental approach; that is, write programs having these problems. (You can check the discussion in Chapter 4 of ```limits.h``` and ```float.h``` to get guidance on the largest and smallest values.)

### 思路
这题比较简单，在于考察底层的知识。可以参考注释，找找自己电脑的 ```limits.h``` 头文件，找到相关的定义。像我这台64位的电脑，```int``` 和 ```long``` 就都占了4个字节。还能获取到具体的数值。

### 源代码

```C
// <C3Q1> integer overflow and floating-point overflow / underflow

# include <stdio.h>

int main(void)
{
	// integer overflow
	int max_int = 2147483647;
	max_int++;
	printf("%d\n", max_int);
	max_int++;
	printf("%d\n", max_int);

	// floating-point overflow
	float max_float = 3.402823466e+38F;
	float min_positive_float = 1.401298464e-45F;

	printf("%f\n", max_float + 1);
	printf("%f\n", max_float - 1);
	printf("%f\n", min_positive_float - 1);
	printf("%f\n", min_positive_float / 2);
	return 0;
}

/*
	在VS2022的输出结果为：
	------------------------------
	-2147483648
	-2147483647
	340282346638528859811704183484516925440.000000
	340282346638528859811704183484516925440.000000
	-1.000000
	0.000000
	------------------------------
*/
```

## 题目2
>Write a program that asks you to enter an ASCII code value, such as 66, and then prints the character having that ASCII code.

### 思路
本题涉及到之前讲的，和转义字符有关的知识。注意事项有输入输出的类型等等，注释里写的很详尽。为了让代码更有整体性，注意的地方和疑难在注释中更全面，这是本文的一个约定。

### 源代码
```C
// <C3Q2.c> print a character based on given ASCII code
# define _CRT_SECURE_NO_WARNINGS

# include <stdio.h>

int main(void)
{
	int ASCII_ch; // char type will result in runtime debug error, however you can ignore it; an integer type is richer in type
	scanf("%d", &ASCII_ch); // must be %d, %c will cause receiving only the first number as a character

	printf("%c\n", ASCII_ch); // must be %c to reach your goal, output binary code as a character type

	return 0;
}

```

## 题目3
>Write a program that sounds an alert and then prints the following text:
>```Startled by the sudden sound, Sally shouted,```
>```"By the Great Pumpkin, what was that!"```


### 思路
还是转义字符的知识，如何打印出\a，以及如何打印出有特殊含义的转义字符，如 \、"等。

### 源代码
```C
/* <C3Q3.c> using printf */

# include <stdio.h>

int main(void)
{
	printf("\aStartled by the sudden sound, Sally shouted,\n");
	printf("\"By the Great Pumpkin, what was that!\"");

	return 0;
}

/*
	在VS2022的输出结果为：
	------------------------------
	Startled by the sudden sound, Sally shouted,
	"By the Great Pumpkin, what was that!"
	------------------------------
*/
```

## 题目4
>Write a program that reads in a floating-point number and prints it first in decimal-point notation, then in exponential notation, and then, if your system supports it, p notation.
Have the output use the following format (the actual number of digits displayed for the exponent depends on the system):
```Enter a floating-point value: 64.25```
```fixed-point notation: 64.250000```
```exponential notation: 6.425000e+01```
```p notation: 0x1.01p+6```

### 思路
浮点数的格式化输出，还挺有用的。之前不太喜欢 ```%e``` 格式的输出，用习惯了还觉得挺顺眼的。

### 源代码
```C
/* <C3Q4.c> handle floats */
# define _CRT_SECURE_NO_WARNINGS

# include <stdio.h>

int main(void)
{
	float f_val;

	printf("Enter a floating-point value: ");
	scanf("%f", &f_val);
	printf("fixed-point notation: %f\n", f_val);
	printf("exponential notation: %e\n", f_val);
	printf("p notation: %a\n", f_val); // Note the specifier for p notation is %a
	
	return 0;
}
```

## 题目5
>There are approximately 3.156 × 10<sup>7</sup> seconds in a year. Write a program that requests your age in years and then displays the equivalent number of seconds.

### 思路
下边几道题都是单位换算，比较简单。

### 源代码

```C
// <C3Q5.c> year to seconds

# define _CRT_SECURE_NO_WARNINGS


# include <stdio.h>

int main (void)
{
	int year_age;
	int sec_age; // Though the number is in a e notation, it's a integer actually, so int type will be fine

	printf("Please enter your age: ");
	scanf("%d", &year_age);

	sec_age = 3.156e+7 * year_age; 
	printf("That's equivalent to %d seconds!\n", sec_age);
	return 0;
}
```

## 题目6

>The mass of a single molecule of water is about 3.0×10<sup>-23</sup> grams. A quart of water is about 950 grams.Write a program that requests an amount of water, in quarts, and displays the number of water molecules in that amount.

### 源代码

```C
/* <C3Q6.c> quarts to molecules */

# define _CRT_SECURE_NO_WARNINGS

# include <stdio.h>

int main(void)
{
	double water_quarts, molecule_num;
	
	printf("Please enter an amount of water in quart(s): ");

	// to receive a double value in C, you MUST use "%lf"! 
	// In printf(), both "%lf" and "%f" will work, 
	// because float arguments in prinf() will be automatically transferred into double type.
	scanf("%lf", &water_quarts); 

	//printf("water quarts: %f\n", water_quarts);

	molecule_num = water_quarts * 950 / 3.0e-23;

	printf("There are about %.2e molecules.\n", molecule_num); // %d will result in trash values, and e notation is more clear

	return 0;
}
```

## 题目7
>There are 2.54 centimeters to the inch. Write a program that asks you to enter your height in inches and then displays your height in centimeters. Or, if you prefer, ask for the height in centimeters and convert that to inches.

### 源代码

```C
/* <C3Q7.c> centimeter to inch */

# define _CRT_SECURE_NO_WARNINGS

# include <stdio.h>

int main(void)
{
	double cm_height, inch_height;

	printf("Enter your height in centimeters: ");
	scanf("%lf", &cm_height);
	
	inch_height = cm_height / 2.54;

	printf("So you are %.2f in inch!\n", inch_height);

	return 0;
}
```

## 题目8

>In the U.S. system of volume measurements, a pint is 2 cups, a cup is 8 ounces, an ounce is 2 tablespoons, and a tablespoon is 3 teaspoons. Write a program that requests a volume in cups and that displays the equivalent volumes in pints, ounces, tablespoons, and teaspoons. Why does a floating-point type make more sense for this application than an integer type?

### 源代码
```C
/* <C3Q8.c> volume measurements */

# define _CRT_SECURE_NO_WARNINGS

# include <stdio.h>

int main(void)
{
	float vol;

	printf("Enter a volumn in cups: ");
	scanf("%f", &vol);

	printf("== %.2f pint(s)\n", vol / 2);
	printf("== %.2f ounce(s)\n", vol * 8);
	printf("== %.2f tablespoon(s)\n", vol * 8 * 2);
	printf("== %.2f teaspoon(s)\n", vol * 8 * 2 * 3);

	return 0;
}

/*
	在VS2022的输出结果为：
	------------------------------
	Enter a volumn in cups: 2.3
	== 1.15 pint(s)
	== 18.40 ounce(s)
	== 36.80 tablespoon(s)
	== 110.40 teaspoon(s)
	------------------------------
*/
```

## 小结
本章的题目大多和数据类型的转换有关。在把代码整合为文章的过程中，又习得了 MarkDown 的另一个语法，用左右各三个`符号可以框住代码块。当然还有上标 ```<sup></sup> ``` 以及下标 ```<sub></sub>```。还能学到HTML，一举两得。
