---
title: C Primer Plus 第四章编程练习笔记
date: 2024-02-19
"categories": C
permalink: 2024/02/cpp-c4-pe/
---

## 序
今天完成了第四章的习题，把自己的解题代码分享在这里。就不像上次那么啰嗦了，这回只展示源代码，需要注意的地方，在注释里已经注明。

## C4Q1
### 题目
>Write a program that asks for your first name, your last name, and then prints the names
in the format _last, first_ .

### 代码
```C
// C4Q1.c 姓名的输入输出

# define _CRT_SECURE_NO_WARNINGS
# include <stdio.h>

int main(void)
{
	char first[40];
	char last[40];

	printf("Please input your first name: ");
	scanf("%s", first); // no need to add '&' prefix
	printf("Please input your last name: ");
	scanf("%s", last); // no need to add '&' prefix

	printf("%s, %s", last, first);

	return 0;
}

/*
    output:
    -------
    Please input your first name: Ling
    Please input your last name: Xia
    Xia, Ling
*/
```

## C4Q2
### 题目
>Write a program that requests your first name and does the following with it:  
>&nbsp;&nbsp;&nbsp;&nbsp;a. Prints it enclosed in double quotation marks  
>&nbsp;&nbsp;&nbsp;&nbsp;b. Prints it in a field 20 characters wide, with the whole field in quotes and the name at the right end of the field  
>&nbsp;&nbsp;&nbsp;&nbsp;c. Prints it at the left end of a field 20 characters wide, with the whole field enclosed in quotes  
>&nbsp;&nbsp;&nbsp;&nbsp;d. Prints it in a field three characters wider than the name

### 代码
```C
// C4Q2.c 名字的多种输出格式

# define _CRT_SECURE_NO_WARNINGS

# include <stdio.h>
# include <string.h>
int main(void)
{
	char first[40];

	printf("Please input your first name: ");
	scanf("%s", first);

	printf("\"%s\"\n", first); // in double quotation marks
	printf("\"%20s\"\n", first);
	printf("\"%-20s\"\n", first);
	printf("%*s\n", strlen(first)+3, first); // 3 characters wider than the name

	return 0;
}

/*
    output:
    -------
    Please input your first name: Ling
    "Ling"
    "                Ling"
    "Ling                "
        Ling
*/
```

## C4Q3
### 题目
>Write a program that reads in a floating-point number and prints it first in decimal-point notation and then in exponential notation. Have the output use the following formats (the number of digits shown in the exponent may be different for your system):\
>a. The input is ```21.3``` or ```2.1e+001``` .\
>b. The input is ```+21.290``` or ```2.129E+001``` .

### 代码
```C
// C4Q3.c 浮点数的输出

# define _CRT_SECURE_NO_WARNINGS

# include <stdio.h>

int main(void)
{
	double num;

	printf("Input a floating-point number: "); //21.290
	scanf("%lf", &num); // remember '&' mark!

	printf("The input is %.1f or %.1le.\n", num, num);
	printf("The input is %+.3f or %.3E.\n", num, num);

	return 0;
}

/*
    output:
    --------
    Input a floating-point number: 21.290
    The input is 21.3 or 2.1e+01.
    The input is +21.290 or 2.129E+01.
*/
```
## C4Q4
### 题目
>Write a program that requests your height in inches and your name, and then displays the information in the following form:
```Dabney, you are 6.208 feet tall```
>Use type ```float``` , and use / for division. If you prefer, request the height in centimeters and display it in meters.

### 代码
```C
// C4Q4.c 输出名字和身高

# define _CRT_SECURE_NO_WARNINGS

# include <stdio.h>

int main(void)
{
	char name[40];
	double height;

	printf("Please enter your name: ");
	scanf("%s", name);
	printf("Please enter your height in inches: ");
	scanf("%lf", &height); // remember '&' mark!

	printf("%s, you are %.3f feet tall\n", name, height);

	return 0;
}

/*
    output:
    --------
    Please enter your name: Dabney
    Please enter your height in inches: 6.208
    Dabney, you are 6.208 feet tall
*/
```

## C4Q5
### 题目
>Write a program that requests the download speed in megabits per second (Mbs) and the size of a file in megabytes (MB). The program should calculate the download time for the file. Note that in this context one byte is eight bits. Use type ```float``` , and use / for division. The program should report all three values (download speed, file size, and download time) showing two digits to the right of the decimal point, as in the following:
```At 18.12 megabits per second, a file of 2.20 megabytes```
```downloads in 0.97 seconds.```

### 代码
```C
// C4Q5.c 下载

# define _CRT_SECURE_NO_WARNINGS

# include <stdio.h>

int main(void)
{
	float speed, size, time;

	printf("Please give me current download speed(Mbs) and the size of a file(MB) in order: ");
	scanf("%f %f", &speed, &size); // '&' mark!

	time = (size * 8) / speed;
	
	printf("At %.2f megabits per second, a file of %.2f megabytes\ndownloads in %.2f seconds.", speed, size, time);
	return 0;
}

/*
    output:
    ---------
    Please give me current download speed(Mbs) and the size of a file(MB) in order: 18.12 2.20
    At 18.12 megabits per second, a file of 2.20 megabytes
    downloads in 0.97 seconds.
*/
```

## C4Q6
### 题目
>Write a program that requests the user’s first name and then the user’s last name. Have it print the entered names on one line and the number of letters in each name on the following line. Align each letter count with the end of the corresponding name, as in the following:
```C
Melissa Honeybee
      7        8
```
>Next, have it print the same information, but with the counts aligned with the beginning of each name.
```C
Melissa Honeybee
7       8
```

### 代码
```C
// C4Q6.c 姓名与字符数的输出

# define _CRT_SECURE_NO_WARNINGS

# include <stdio.h>
# include <string.h>

int main(void)
{
	char first[40], last[40];

	printf("Please enter your first name: ");
	scanf("%s", first);
	printf("Please enter your last name: ");
	scanf("%s", last);

	printf("%s %s\n", first, last);
	printf("%*d %*d\n", strlen(first), strlen(first), strlen(last), strlen(last));

	printf("%s %s\n", first, last);
	printf("%-*d %-*d\n", strlen(first), strlen(first), strlen(last), strlen(last));

	return 0;
}

/*
    output:
    --------
    Please enter your first name: Melissa
    Please enter your last name: Honeybee
    Melissa Honeybee
        7        8
    Melissa Honeybee
    7       8
*/
```

## C4Q7
### 题目
>Write a program that sets a type ```double``` variable to 1.0/3.0 and a type ```float``` variable to 1.0/3.0. Display each result three times—once showing four digits to the right of the decimal, once showing 12 digits to the right of the decimal, and once showing 16 digits to the right of the decimal. Also have the program include ```float.h``` and display the values of ```FLT_DIG``` and ```DBL_DIG``` . Are the displayed values of 1.0/3.0 consistent with these values?

### 代码
```C
// C4Q7.c float 和 double 类型对数值的存储精度

# define _CRT_SECURE_NO_WARNINGS

# include <stdio.h>
# include <float.h>

int main(void)
{
	//float f_num = 1.0 / 3.0; // error
	//double d_num 1.0 / 3.0; // error

	float f_num;
	double d_num;

	f_num = 1.0 / 3.0;
	d_num = 1.0 / 3.0;

	printf("%25.4f %25.4f\n", f_num, d_num);
	printf("%25.12f %25.12f\n", f_num, d_num);
	printf("%25.16f %25.16f\n", f_num, d_num);
	printf("Further:\n%25.20f %25.20f\n", f_num, d_num);

	printf("FLT_DIG:%d\nDBL_DIG:%d\n", FLT_DIG, DBL_DIG); // FLT_DIG: Minimum number of significant decimal digits for a float
	                                                      // 一个浮点数的最小有效数字位数
	return 0;
}

/*
	output:
	----------
                   0.3333                    0.3333
           0.333333343267            0.333333333333
       0.3333333432674408        0.3333333333333333
Further:
   0.33333334326744079590    0.33333333333333331483
FLT_DIG:6
DBL_DIG:15
*/
```

## C4Q8
### 题目
>Write a program that asks the user to enter the number of miles traveled and the number of gallons of gasoline consumed. It should then calculate and display the miles-per-gallon value, showing one place to the right of the decimal. Next, using the fact that one gallon is about 3.785 liters and one mile is about 1.609 kilometers, it should convert the miles-per-gallon value to a liters-per-100-km value, the usual European way of expressing fuel consumption, and display the result, showing one place to the right of the decimal. Note that the U. S. scheme measures the distance traveled per amount of fuel (higher is better), whereas the European scheme measures the amount of fuel per distance (lower is better). \
>Use symbolic constants (using ```const``` or ```#define``` ) for the two conversion factors.

### 代码
```C
// C4Q8.c 燃油效率

# define _CRT_SECURE_NO_WARNINGS

# include <stdio.h>

# define GALLONTOLITER 3.785 // no ';'
# define MILETO100KM 1.609 / 100 // no ';' & '/' not '*'!

int main(void)
{
	double m_distance, g_fuel, km_distance, l_fuel;
	double mpg, lp100km; // miles-per-gallon, liters-per-100km
	printf("Please enter miles travelled and gallons of gasoline consumed: ");
	scanf("%lf %lf", &m_distance, &g_fuel);

	km_distance = m_distance * MILETO100KM; // distance in kilometers
	l_fuel = g_fuel * GALLONTOLITER; // fuel in liters

	mpg = m_distance / g_fuel;
	lp100km = l_fuel / km_distance;

	printf("Miles-per-gallon: %.1f\n", mpg);
	printf("Liters-per-100km: %.1f\n", lp100km);

	return 0;
}

/*
    output:
    -----------
    Please enter miles travelled and gallons of gasoline consumed: 10 10
    Miles-per-gallon: 1.0
    Liters-per-100km: 235.2
    -----------
    Please enter miles travelled and gallons of gasoline consumed: 10 5
    Miles-per-gallon: 2.0
    Liters-per-100km: 117.6
*/
```

## 跋
本节题目不难，关键是掌握 ```printf()``` 和 ```scanf()``` 的修饰符，以合乎期望地显示输出内容。

其中，第六题用到了 ```%*d``` 修饰符，可以动态地调整 field 的宽度，很有应用价值；\
第七题对 ```float``` 和 ```double``` 类型的变量进行了不同小数位数的输出，结合 ```floats.h``` 头文件，可以对浮点型数据的存储有更深的理解；\
第八题使用了 ```#define``` 定义 _symbolic constant_ ，同时要求对数据进行有效处理，具有应用价值。

对于这些题目，有共同点需要注意：
* 对于以数组类型存储的字符串，在 ```scanf()``` 中不需要在变量名前加 '&'；
* 对于其他类型的变量，则需要加 '&'。