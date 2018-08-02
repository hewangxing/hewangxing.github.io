---
title: STM32之寄存器访问
date: 2018-05-07 14:51:05
categories: 电子
tags: STM32
comments: true
---

一般的寄存器访问需要通过**读-改-写三步曲** 和 **位运算的清0置1**来实现，但在stm32的编程中，通过利用它的一些优秀的特性如端口位设置/复位寄存BSRR、位绑定等，我们可以大大提升寄存器的访问速度和简化寄存器的操作。

```
//一般寄存器操作：
GPIOx->ODR |= 0x10;  //Pin4置1
GPIOx->ODR &= ~0x10; //Pin4清0
```

**BSRR/BRR寄存器**
![这里写图片描述](/images/20171127134008645.png)
![这里写图片描述](/images/20171127134102899.png)

```
GPIOx->BSRR //对BSRR的低16位写1置位，对BSRR的高16位写1清零
GPIOx->BRR  //对BRR的低16位写1清零，BRR的高16位保留
```
由此可见，通过BSRR/BRR寄存器来操作ODR寄存器， 不需要 **读-改-写三步曲**， 仅通过 **写** 就可一步到位，方便不少。

**位绑定**
当然了，stm32还有一个更牛X的特性--位绑定，仅仅只要1个时钟周期就能实现单独的位操作。位绑定，是通过简单的地址变换将寄存器中的某一个位映射到内存中的某一个存储单元。这样通过对一个内存单元的读写就能间接访问相应寄存器的某个位了，当然此时该32位的内存单元也只有最低位是有效的啦！

```
但是整个M3内核并没有全部允许位绑定，只有两个区有，分别是
SARM：0x20000000~0x2000FFFF 1M大小 
这个区绑定的地址是从0x22000000开始的；  
PERIPHERALS：0x40000000~0x4000FFFFF 1M大小
这个区绑定的地址是从0x42000000开始的；

对应的位绑定公式为：  
SRAM：AliasADDr = 0x22000000+((A-0x20000000)*32+n*4)
其中A：0x20000000~0x2000FFFF n：0~31
PERIPHERALS: AliasADDr = 0x42000000+((A-0x40000000)*32+n*4)
其中A：0x40000000~0x4000FFFFF n：0~31
```
下面就可以通过位绑定来快速实现位操作
![这里写图片描述](/images/20171127141437543.png)
![这里写图片描述](/images/20171127141452482.png)

```
#define GPIOA_ODR_ADDR (GPIOA_BASE + 0x0C)
#define GPIOA_IDR_ADDR (GPIOA_BASE + 0x08)

#define BitBind(Addr, bitNum) (*(volatile unsigned long *)((Addr&0xF0000000)+0x2000000+((Addr&0xFFFFF)<<5)+(bitNum<<2)))
//Addr&0xF0000000是为了区分SRAM还是PERIPHERALS
//Addr&0xFFFFF相当于(A-0x20000000)或者(A-0x40000000)
//左移是为了实现快速的乘法操作：左移n位相当于乘以2^n

#define PAout(n) BitBind(GPIOA_ODR_ADDR, n) 
#define PAin(n) BitBind(GPIOA_IDR_ADDR, n)
```
这样就实现了类似51单片机访问I/O的操作方式

```
sbit P10 = P1^0
P10 = 0; 或 P10 = 1;

PAout(3) = 1; 或 PAout(3) = 0;	
```
Pretty cool, huh!