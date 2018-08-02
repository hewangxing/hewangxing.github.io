---
title: C语言之递归
date: 2018-05-06 11:42:24
categories: 编程
tags: C语言
comments: true 
---

我将这篇博文的代码托管在了Github上，分别是fabonacci.c(斐波那契数列问题） 和 hanoi.c（汉诺塔问题）。  
获取源码：

    git clone git@github.com:hewangxing/data-struct.git
    

递归理解：递归是一个自相似并不断重复的过程，在程序中呈现为一种函数在定义时调用自身的方法。
递归的特征：  
1.有一个参考值作为基准，使得递归过程可以在有限步骤结束并获得确定的结果。
2.问题可以分解为更小且相同的部分。（难点）  
递归常用在有递推公式、迭代方程等的类似问题中。

递归的大致过程：

    递归函数（输入）  
    {
        if 符合基准情况  
            return 基准情况下问题的解
        输入 = 分解成更小但重复的问题
        递归函数（输入）
    }
    
一、斐波那契数列问题：

观察以下数列：

    0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233 ……
    
数列的第0项和第1项的值分别为：0和1。后续第N项的值的为第N-1项的值与第N-2的值之和。 求一个递归函数来求斐波那契数列上第N项的值。

    /* 
     * descrip:  a recursion exercise, fabonacciv sequence 
     * author:   xingyes
     * date:     2017-05-01
     */
    
    #include <stdio.h>
    #include <stdlib.h>
    
    /*long long fabonacciv(int n)
    {
        if(n == 0) return 0;
        else if(n == 1) return 1;
        else return fabonacciv(n-1) + fabonacciv(n-2);
    }*/
    
    long long sequence(int n, long long t0, long long t1)
    {
        if(n == 0) return t0;
        else if(n == 1) return t1;
        else return sequence(n-1, t1, t0+t1);
    }
    
    long long fabonacciv(int n)
    {
        return sequence(n, 0, 1);
    }
    
    int main(int argc, char *argv[])
    {
        if(argc != 2)
        {
            fprintf(stderr, "Usage: %s N.\n", argv[0]);
            return 0;
        }
        int n = atoi(argv[1]);
        fprintf(stdout, "%d  %lld\n", n, fabonacciv(n));
        return 0;
    }
    

二、汉诺塔问题

有三根杆子A，B，C。A杆上有N个(N>1)穿孔圆盘，盘的尺寸由下到上依次变小。要求按下列规则将所有圆盘移至C杆：

1.每次只能移动一个圆盘。
2.大盘不能叠在小盘上面。

问题可以分解成三步：（以三个盘为例）

<img src="http://123.206.132.71/wp-content/uploads/2017/05/P70501-225214.jpg" alt="" width="768" height="569" class="alignnone size-full wp-image-183" />

    /* 
    *descrip: a recursion exercise, solution for hanoi 
    *author: xingyes 
    *date: 2017-05-01 
    */ 
    #include <stdio.h> 
    #include <stdlib.h>
    
    void moveSingleStep(int disk, char A, char B, char C)
    {
        static int step = 1;
        fprintf(stdout, "step%d: disk%d %c-->%c\n", step++, disk, A, C);
    }
    
    void hanoi(int n, int disk, char A, char B, char C)
    {
        if(n == 1) 
            moveSingleStep(disk, A, B, C);
        else 
        {
            hanoi(n-1, disk-1, A, C, B);
            hanoi(1, disk, A, B, C);
            hanoi(n-1, disk-1, B, A, C);
        }
    }
    
    int main(int argc, char *argv[])
    {
        if(argc != 2)
        {
            fprintf(stderr, "Usage: %s N.\n", argv[0]);
            return 0;
        }
        int n = atoi(argv[1]);
        fprintf(stdout, "========hanoi(%d)\n", n);
        hanoi(n, n, 'A', 'B', 'C');
        fprintf(stdout, "========hanoi finished\n");
        return 0;
    }