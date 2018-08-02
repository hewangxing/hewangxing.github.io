---
title: STM32之SPI通信
date: 2018-05-07 14:42:22
categories: 电子
tags: STM32
comments: true
---

之前一直对SPI通信一知半解，所以想抽空把它搞得明白一些。考虑到之前是结合Flash芯片来学的，十分不直观，而且主要把时间和精力都花在Flash芯片的datasheet和驱动上了，SPI通信也没学好。所以这次就考虑用4位数码管显示模块，模块是直接买的现成的，如下图所示，这样可以简化操作，把精力聚焦到学习的核心--SPI通信本身上来。
![这里写图片描述](/images/20171215131641714.png)
该模块是用2片74HC595串联驱动的，一片用来控制数码管的位选(U1)，一片用来控制数码管的段选(U2)。接口比较简单，总共5个引脚，2个引脚分别接VCC和GND，DIO用来接收串行数据的输入，SCLK用来接收同步时钟，每个SCLK上升沿74HC595内部的移位寄存器会移一位，RCLK用来控制数据的输出，每个RCLK上升沿74HC595内部的移位寄存器的数据会被放进存储寄存器并输出到外部引脚QA~QH上。而QH'是串行输出引脚，该引脚会接收最高位的溢出，从而实现多片74HC595的级联。
![这里写图片描述](/images/20171215141518010.png)
当两片74HC595串联时，先发八位数据用于段选，再发八位数据用于位选，然后RCLK上升沿，就可以驱动某位数码管显示某个字符，通过动态扫描数码管，由于人眼的视觉暂停效果，就可以实现4位数码管的同时显示。先用通用I/O来实现该数码管的驱动，程序如下：
*头文件74HC595.h*
```
#ifndef __74HC595_H__
#define __74HC595_H__

#include"stm32f10x_lib.h"		//包含所有的头文件
#include<stdio.h>

// 4-Bit LED Digital Tube Module
#define  HC595_SCLK_PIN  GPIO_Pin_5   // SPI1_SCK  PA5
#define  HC595_RCLK_PIN  GPIO_Pin_12   // SPI1_NSS  PA4
#define  HC595_DIO_PIN   GPIO_Pin_7   // SPI1_MOSI PA7
#define  HC595_GPIO           GPIOA 
#define  HC595_RCLK_GPIO      GPIOB 
#define  HC595_RCC            RCC_APB2Periph_GPIOA 
#define  HC595_RCLK_RCC       RCC_APB2Periph_GPIOB 

void HC595_Init(void);
void HC595_SendByte(u8 data);
u8 HC595_Display(u16 num, u8 dp);

#endif
```
*源文件74HC595.c*
```
// 用于HC595实现的4Bit-LED Digit Tube Module
// 注意：该4位数码管是共阳的！
#include "74HC595.h"

// 码表
const u8 digitTable[] = 
{
// 0	   1	   2	   3	   4	   5	   6	   7	   8	   9
	0xC0, 0xF9, 0xA4, 0xB0, 0x99, 0x92, 0x82, 0xF8, 0x80, 0x90,
// A	    b	    C     d	    E     F    -
	0x8C, 0xBF, 0xC6, 0xA1, 0x86, 0xFF, 0xbf
};

/*******************************************************************************
* Function Name  : HC595_Init
* Description    : 初始化HC595
* Input          : None
* Output         : None
* Return         : None
*******************************************************************************/
void HC595_Init(void)
{
	GPIO_InitTypeDef GPIO_InitStructure;		    //声明一个结构体变量
	RCC_APB2PeriphClockCmd(HC595_RCC | HC595_RCLK_RCC, ENABLE);	//使能HC595的时钟	
	
	//74HC595, SCLK RCLK DIO 推挽输出
	GPIO_InitStructure.GPIO_Pin = HC595_SCLK_PIN| HC595_DIO_PIN;       
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;	 //管脚频率为50MHZ
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;	 //输出模式为推挽输出
	GPIO_Init(HC595_GPIO, &GPIO_InitStructure);				 //初始化寄存器	
	
	GPIO_InitStructure.GPIO_Pin = HC595_RCLK_PIN;       
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;	 //管脚频率为50MHZ
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;	 //输出模式为推挽输出
	GPIO_Init(HC595_RCLK_GPIO, &GPIO_InitStructure);	 //初始化寄存器	
	
}

/*******************************************************************************
* Function Name  : HC595_SendByte
* Description    : 发送一个字节
* Input          : data
* Output         : None
* Return         : None
*******************************************************************************/
void HC595_SendByte(u8 data)
{
	u8 i;
	for (i=8; i>=1; i--)
	{
		// 高位在前
		if (data&0x80) 
			GPIO_SetBits(HC595_GPIO, HC595_DIO_PIN); 
		else 
			GPIO_ResetBits(HC595_GPIO, HC595_DIO_PIN);
		data <<= 1;
		// SCLK上升沿
		GPIO_ResetBits(HC595_GPIO, HC595_SCLK_PIN);
		GPIO_SetBits(HC595_GPIO, HC595_SCLK_PIN);
	}
}

/*******************************************************************************
* Function Name  : HC595_Display
* Description    : 显示4位数字(包括小数点)
* Input          : num: 0000 - 9999 
*                  dp: 小数点的位置1-4 
* Output         : None
* Return         : 正常返回0，错误返回1
*******************************************************************************/
u8 HC595_Display(u16 num, u8 dp)
{
	u8 qian = 0, bai = 0, shi = 0, ge = 0;
	
	// 对显示的参数范围进行检查
	if (num > 9999 || dp > 4)
		//报错
		return 1;
	
	// 对num进行分解
	qian = num / 1000;
	bai = num % 1000 / 100;
	shi = num % 100 / 10;
	ge = num % 10;
	
	// 千位
	if(dp == 1)
		HC595_SendByte(digitTable[qian] & 0x7F);
	else
		HC595_SendByte(digitTable[qian]);
	HC595_SendByte(0x08);		
	GPIO_ResetBits(HC595_RCLK_GPIO, HC595_RCLK_PIN);
	GPIO_SetBits(HC595_RCLK_GPIO, HC595_RCLK_PIN);
	
	// 百位
	if(dp == 2)
		HC595_SendByte(digitTable[bai] & 0x7F);
	else
		HC595_SendByte(digitTable[bai]);
	HC595_SendByte(0x04);		
	GPIO_ResetBits(HC595_RCLK_GPIO, HC595_RCLK_PIN);
	GPIO_SetBits(HC595_RCLK_GPIO, HC595_RCLK_PIN);
	
	// 十位
	if(dp == 3)
		HC595_SendByte(digitTable[shi] & 0x7F);
	else
		HC595_SendByte(digitTable[shi]);
	HC595_SendByte(0x02);		
	GPIO_ResetBits(HC595_RCLK_GPIO, HC595_RCLK_PIN);
	GPIO_SetBits(HC595_RCLK_GPIO, HC595_RCLK_PIN);
	
	// 个位
	if(dp == 4)
		HC595_SendByte(digitTable[ge] & 0x7F);
	else
		HC595_SendByte(digitTable[ge]);
	HC595_SendByte(0x01);		
	GPIO_ResetBits(HC595_RCLK_GPIO, HC595_RCLK_PIN);
	GPIO_SetBits(HC595_RCLK_GPIO, HC595_RCLK_PIN);
	
	return 0;
}

```

接下来就可以把重心都放在STM32的SPI外设上了，首先需要读一下*STM32F10x的参考手册*的SPI（串行外设接口）部分，这样对SPI就可以有较好的理解，比较重要的是要看懂SPI的结构框图和主从机通信的示意图，如下：
    ![这里写图片描述](/images/20171215144420183.png)
    ![这里写图片描述](/images/20171215144438974.png)
这个理解以后，我们就可以参考*《STM32F103XX固件库用户手册》*的SPI部分来实现STM32的SPI外设配置和收发数据了，具体代码如下：
*头文件74HC595_SPI.h*
```
#ifndef __74HC595_SPI_H__
#define __74HC595_SPI_H__

#include"stm32f10x_lib.h"		//包含所有的头文件
#include<stdio.h>

// 4-Bit LED Digital Tube Module
// 引脚                                // SPI1       4位数码管            
#define  HC595_NSS_PIN   GPIO_Pin_4    // SPI1_NSS   未用
#define  HC595_SCK_PIN   GPIO_Pin_5    // SPI1_SCK   SCLK
#define  HC595_MISO_PIN  GPIO_Pin_6    // SPI1_MISO  未用 
#define  HC595_MOSI_PIN  GPIO_Pin_7    // SPI1_MOSI  DIO
#define  HC595_RCLK_PIN  GPIO_Pin_12   //            RCLK

// 端口
#define  HC595_SPI1_GPIO      GPIOA  
#define  HC595_RCLK_GPIO      GPIOB   
// 时钟
#define  HC595_SPI1_RCC  RCC_APB2Periph_GPIOA
#define  HC595_RCLK_RCC  RCC_APB2Periph_GPIOB 

void HC595_Init(void);
void HC595_SendByte(u8 data);
u8 HC595_Display(u16 num, u8 dp);

#endif

```
*源文件74HC595_SPI.c*
```
/************************省略部分代码见(74HC595.c)************************/
/*******************************************************************************
* Function Name  : HC595_Init
* Description    : 初始化HC595
* Input          : None
* Output         : None
* Return         : None
*******************************************************************************/
void HC595_Init(void)
{
	GPIO_InitTypeDef GPIO_InitStructure;	
  SPI_InitTypeDef SPI_InitStructure;	                              // 声明一个结构体变量
	// 不需要开启AFIO时钟
	RCC_APB2PeriphClockCmd(HC595_SPI1_RCC | HC595_RCLK_RCC | RCC_APB2Periph_SPI1, ENABLE);	// 使能HC595及SPI1的时钟	
	
	//74HC595, SPI1_NSS、SPI1_SCK、SPI1_MOSI 
	GPIO_InitStructure.GPIO_Pin = HC595_NSS_PIN | HC595_SCK_PIN |HC595_MOSI_PIN;       
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;	    // 管脚频率为50MHZ
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;	        // 输出模式为复用推挽输出
	GPIO_Init(HC595_SPI1_GPIO, &GPIO_InitStructure);	    // 初始化寄存器	
	
	//74HC595, SPI1_MISO
	GPIO_InitStructure.GPIO_Pin = HC595_MISO_PIN;       
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING;	// 输入模式为浮空输入
	GPIO_Init(HC595_SPI1_GPIO, &GPIO_InitStructure);	    // 初始化寄存器	
	
	//74HC595, RCLK
	GPIO_InitStructure.GPIO_Pin = HC595_RCLK_PIN;       
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;	    // 管脚频率为50MHZ
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;	    // 输出模式为复用推挽输出
	GPIO_Init(HC595_RCLK_GPIO, &GPIO_InitStructure);	    // 初始化寄存器	
	
	/* Initialize the SPI1 according to the SPI_InitStructure members */
	SPI_InitStructure.SPI_Direction = SPI_Direction_2Lines_FullDuplex;
	// 第一步：设置主从模式和通信速率
	SPI_InitStructure.SPI_Mode = SPI_Mode_Master;
	// SPI_NSS_Hard时需要外部电路把NSS接VCC, SPI_NSS_Soft时SPI外设会将SSM和SSI置位
	SPI_InitStructure.SPI_NSS = SPI_NSS_Soft;
	// 实测波特率最低为SPI_BaudRatePrescaler_8，否则出错
	SPI_InitStructure.SPI_BaudRatePrescaler = SPI_BaudRatePrescaler_4;
	
	// 第二步：设置数据格式
	SPI_InitStructure.SPI_DataSize = SPI_DataSize_8b;
	// MSB在前还是LSB在前要根据码表和数码管与74HC595的接法来定
	SPI_InitStructure.SPI_FirstBit = SPI_FirstBit_MSB;
	
	// 第三步：设置时钟和极性
	// 当SPI_CPOL_Low且SPI_CPHA_2Edge出错
	SPI_InitStructure.SPI_CPOL = SPI_CPOL_Low;
	SPI_InitStructure.SPI_CPHA = SPI_CPHA_1Edge;
	//第四步：其它，CRC校验，可靠通信，这步可以不设置
	SPI_InitStructure.SPI_CRCPolynomial = 7;
	SPI_Init(SPI1, &SPI_InitStructure);
	
	/* Enable SPI1 */
  SPI_Cmd(SPI1, ENABLE);
}

/*******************************************************************************
* Function Name  : HC595_SendByte
* Description    : 发送一个字节
* Input          : data
* Output         : None
* Return         : None
*******************************************************************************/
void HC595_SendByte(u8 data)
{
	SPI_I2S_SendData(SPI1, data);
	while(!SPI_I2S_GetFlagStatus(SPI1, SPI_I2S_FLAG_TXE));
}
/************************省略部分代码(见74HC595.c)************************/
```
这样就大工告成啦，STM32的SPI外设还是比较简单的，尤其是通过库函数来调用。用数码管模块这种简单的可视化工具，我们就可以更好的研究通信协议本身的特性啦，后续我还会用这种方式来学习其它的通信协议，好了，"talk is cheap,  show me the code"!