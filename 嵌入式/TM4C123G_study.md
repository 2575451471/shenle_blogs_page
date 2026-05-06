---
title: TM4C123G_study
date: 2025-01-05 01:27:00
tags: 嵌入式ti
hidden: false
top: false
layout: post
publish: true
---
<meta name="referrer" content="no-referrer"/>
<!-- more -->



# ***图库挂了***
# 时钟


## 时钟源

TM4C123内部共有4个时钟源，见下表

| 时钟                    | 简介                                                         |
| ----------------------- | ------------------------------------------------------------ |
| 内部高精度振荡器(PIOSC) | 内部振荡器，其频率为16MHz，精度为1%，可以用来驱动PLL         |
| 主振荡器 (MOSC)         | 外部高速振荡器，频率可在4-25M间选择，可以驱动PLL（此时频率在5-25M） |
| 低频内部振荡器 (LFIOSC) | 适用于深度睡眠省电模式，它的频率是会改变的，范围在10KHz-90KHz之间，标准值30KHz |
| 休眠模块时钟源          | 32.768KHz晶振，用于实时时钟源或睡眠时钟                      |

 ![img](https://img-blog.csdnimg.cn/20190708100616504.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d4Yzk3MTIzMQ==,size_16,color_FFFFFF,t_70) 

时钟树

 ![img](https://img-blog.csdnimg.cn/20190708100659281.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d4Yzk3MTIzMQ==,size_16,color_FFFFFF,t_70) 

1. MOSC和PIOSC可以用来驱动PLL
2. PLL输出锁定在400MHz，它可以在经过二分频和SYSDIV分频（这个可以程序配置）后提供系统时钟。注意TM4C123G的**最大主频为80MHz**，因此配置时钟的时候，**若使用的PLL，最小分频数只能是2.5分频**。

## 时钟配置

 使用函数 `void SysCtlClockSet(uint32_t ui32Config);` 进行系统时钟设置
这个函数参数是**4个部分做按位与**，包括 时钟分频SYSDIV设置，系统时钟来源（直接用振荡器，还是用PLL倍频过的），时钟源选择（对应上面表2.4），外接晶体频率

```c
1. 时钟分频SYSDIV设置
        #define SYSCTL_SYSDIV_1         0x07800000  // Processor clock is osc/pll /1
		#define SYSCTL_SYSDIV_2         0x00C00000  // Processor clock is osc/pll /2
		...
		#define SYSCTL_SYSDIV_62        0x9EC00000  // Processor clock is osc/pll /62
		#define SYSCTL_SYSDIV_63        0x9F400000  // Processor clock is osc/pll /63
		#define SYSCTL_SYSDIV_64        0x9FC00000  // Processor clock is osc/pll /64
		#define SYSCTL_SYSDIV_2_5       0xC1000000  // Processor clock is pll / 2.5
		#define SYSCTL_SYSDIV_3_5       0xC1800000  // Processor clock is pll / 3.5
		#define SYSCTL_SYSDIV_4_5       0xC2000000  // Processor clock is pll / 4.5
		...
		#define SYSCTL_SYSDIV_61_5      0xDE800000  // Processor clock is pll / 61.5
		#define SYSCTL_SYSDIV_62_5      0xDF000000  // Processor clock is pll / 62.5
		#define SYSCTL_SYSDIV_63_5      0xDF800000  // Processor clock is pll / 63.
```

```C
2.系统时钟来源（直接用振荡器，还是用PLL倍频过的）
	    #define SYSCTL_USE_PLL          0x00000000  // System clock is the PLL clock
		#define SYSCTL_USE_OSC          0x00003800  // System clock is the osc clock
```

```c
3.时钟源选择（对应上面表2.4）
		#define SYSCTL_OSC_MAIN         0x00000000  // Osc source is main osc
		#define SYSCTL_OSC_INT          0x00000010  // Osc source is int. osc
		#define SYSCTL_OSC_INT4         0x00000020  // Osc source is int. osc /4
		#define SYSCTL_OSC_INT30        0x00000030  // Osc source is int. 30 KHz
		#define SYSCTL_OSC_EXT32        0x80000038  // Osc source is ext. 32 KHz

```

```c
4.外接晶体频率
		#define SYSCTL_XTAL_1MHZ        0x00000000  // External crystal is 1MHz
		#define SYSCTL_XTAL_1_84MHZ     0x00000040  // External crystal is 1.8432MHz
		#define SYSCTL_XTAL_2MHZ        0x00000080  // External crystal is 2MHz
		#define SYSCTL_XTAL_2_45MHZ     0x000000C0  // External crystal is 2.4576MHz
		#define SYSCTL_XTAL_3_57MHZ     0x00000100  // External crystal is 3.579545MHz
		...
		#define SYSCTL_XTAL_24MHZ       0x00000640  // External crystal is 24.0 MHz
		#define SYSCTL_XTAL_25MHZ       0x00000680  // External crystal is 25.0 MHz

```

配置示例

```c
/* MOSC频率16M，SYSDIV5分频，系统时钟源自PLL锁相环倍频，时钟源使用MOSC */
/* 系统时钟频率400M/2/5=40M */
SysCtlClockSet(SYSCTL_SYSDIV_5|SYSCTL_XTAL_16MHZ|SYSCTL_USE_PLL|SYSCTL_OSC_MAIN);	

/* MOSC频率16M，SYSDIV2.5分频，系统时钟源自PLL锁相环倍频，时钟源使用MOSC */
/* 系统时钟频率400M/2/2.5=80M , 注意这是上限了*/
SysCtlClockSet(SYSCTL_SYSDIV_2_5 | SYSCTL_USE_PLL | SYSCTL_OSC_MAIN | SYSCTL_XTAL_16MHZ);

/* MOSC频率16M，SYSDIV不分频，系统时钟来自时钟源，时钟源使用MOSC */
/* 系统时钟频率16M/1=16M */
SysCtlClockSet(SYSCTL_SYSDIV_1 | SYSCTL_USE_OSC | SYSCTL_OSC_MAIN | SYSCTL_XTAL_16MHZ);
```

## 延时函数

 TM4C库提供了一个延时函数，它利用汇编，提供了跨越工具链时恒定的延迟。延时3*ui32Count个时钟周期 

```c
__asm void SysCtlDelay(uint32_t ui32Count);
```

 但若系统时钟频率不同，一个时钟周期的长度也不同，一旦改了系统时钟频率，延时就会变化，需要改进

  利用以下函数获取系统时钟频率（单位Hz） 

```c
uint32_t SysCtlClockGet(void);
```

假设系统时钟频率为nHz，即(n/1000)KHz，设cnt=(n/1000)，每秒有1000*cnt个周期，每个cnt长1ms。
SysCtlDelay(Count)可以延时3*Count个周期，令Count=cnt/3，即可延时1个cnt长（即1ms）

- ```c
  //延时n毫秒，不用考虑时钟频率
  		#define delay_ms(n); SysCtlDelay(n*(SysCtlClockGet()/3000));
  
  ```

不管用哪个时钟源，只要工作频率高于40MHz，就会导致实际延时时间大于设置值。原因好像是芯片内部Flash的读取频率最大只能达到40M，
当工作频率大于40MHz时，通过预取两个字的指令来达到80M的运行主频。但是，当遇到SysCtlDelay函数这种短跳转时这个特性并不能很好的工作，每次都需要读取指令，所以时间就延长了
也就是说如果主频大于40M，SysCtlDelay(n*(SysCtlClockGet()/3000))这个方法也不是很准，可以考虑用ROM_SysCtlDelasy()

# GPIO

## RGB_LED

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190708114310667.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d4Yzk3MTIzMQ==,size_16,color_FFFFFF,t_70) 

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/201907081143272.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d4Yzk3MTIzMQ==,size_16,color_FFFFFF,t_70) 

 控制LED是一个三极管开关电路，单片机PF1/PF2/PF3连接到LED_R/LED_B/LED_G，GPIO输出高电平即可点亮二极管 

## 相关库函数

### 1.使能外设时钟

```c
（1）void SysCtlPeripheralEnable(uint32_t ui32Peripheral)
```

功能：使能外设时钟
参数：uint32_t ui32Peripheral 要使能的外设
说明：从写外设使能操作完成到实际上的外设使能间有5个时钟周期的延迟，这期间内访问外设将导致一个总线错误。应注 意确保在这段时间内不访问该外设。

### 2.引脚配置为输出模式

```c
（2）void GPIOPinTypeGPIOOutput(uint32_t ui32Port, uint8_t ui8Pins)
```

功能：引脚配置为输出模式
参数：
ui32Port GPIO口的基地址
ui8Pins bit-packed格式表示的引脚
说明：要使GPIO引脚做为GPIO输出，必须正确地配置引脚。本函数提供这些引脚的典型配置。引脚使用bit-packed 字节格式表示，每一位表示一个要访问的引脚，位0表示引脚0，位1表示引脚1，以此类推。
底层：

```c
void GPIOPadConfigSet(uint32_t ui32Port, uint8_t ui8Pins,uint32_t ui32Strength, uint32_t ui32PinType);//设置输出类型和强度
void GPIODirModeSet(ui32Port, ui8Pins, GPIO_DIR_MODE_OUT);//设置方向（输入or输出）
```

### 3.写值到指定引脚

```c
void GPIOPinWrite(uint32_t ui32Port, uint8_t ui8Pins, uint8_t ui8Val);
```

功能：写值到指定引脚.
参数：
ui32Port GPIO口的基地制作.
ui8Pins bit-packed 格式表示的引脚
ui8Val 将要写入引脚的值.
说明：写相应位的数值到ui8Pins参数指定的引脚，写数值时不影响配置为输入的引脚状态。引脚用 bit-packed 字节格式表示, 每一个位代表一个引脚，位0表示GPIO口的引脚0，位1表示GPIO口的引脚1，以此类推。

### 4.不受频率影响的延时函数

```c
SysCtlDelay(100*(SysCtlClockGet()/3000));
```

## 示例代码

```c
#include <stdint.h>
#include <stdbool.h>
#include "inc/hw_types.h"					//通用类型和宏
#include "inc/hw_memmap.h"					//外设和存储器的基地址
#include "driverlib/sysctl.h"				//API函数中外设、状态等的标志
#include "driverlib/gpio.h"

int main(void)
{
	uint8_t ui8LED = 2;	//2 = 0010
                        //1 = 0001
                        //8 = 0100
    
	//系统时钟分频器系数|选择外部晶振频率|使用PLL锁相环作为系统时钟源
SysCtlClockSet(SYSCTL_SYSDIV_6|SYSCTL_XTAL_16MHZ|SYSCTL_USE_PLL|SYSCTL_OSC_MAIN); 
	//使能PF时钟
	SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOF);	
   //配置引脚为GPIO输出，底层是GPIOPadConfigSet和GPIODirModeSet	
	GPIOPinTypeGPIOOutput(GPIO_PORTF_BASE, GPIO_PIN_1|GPIO_PIN_2|GPIO_PIN_3);						
	
	while(1)
	{
		// Turn on the LED ：PF1(R),PF2(B),PF3(G)
		GPIOPinWrite(GPIO_PORTF_BASE, GPIO_PIN_1|GPIO_PIN_2|GPIO_PIN_3, ui8LED);	

		// Delay for a bit		
		SysCtlDelay(100*(SysCtlClockGet()/3000));
		
		// Cycle through Red, Green and Blue LEDs
		//写入2->4->8(0010->0100->1000)（R->B->G）
		if (ui8LED == 8) 
			ui8LED = 2; 
		else 
			ui8LED = ui8LED*2;								
	}
}

```

# EXTI

## 相关库函数

### 1.设置指定引脚的中断触发类型.

```c
void GPIOIntTypeSet(uint32_t ui32Port, uint8_t ui8Pins,uint32_t ui32IntType)
```

功能：设置指定引脚的中断触发类型.
参数:
（1）ui32Port： GPIO口的基地址
（2）ui8Pins： 多个bit-packed格式表示的引脚
（3）ui32IntType： 中断触发类型(有以下类型)
		

```c
#define GPIO_FALLING_EDGE       0x00000000  // Interrupt on falling edge
		#define GPIO_RISING_EDGE        0x00000004  // Interrupt on rising edge
		#define GPIO_BOTH_EDGES         0x00000001  // Interrupt on both edges
		#define GPIO_LOW_LEVEL          0x00000002  // Interrupt on low level
		#define GPIO_HIGH_LEVEL         0x00000006  // Interrupt on high level
		//前5个都可和下面这个或运算一起作为ui32IntType参数，但不是所有引脚都支持离散中断，需要查手册
		#define GPIO_DISCRETE_INT       0x00010000  // Interrupt for individual pins
```

说明： 为了避免毛刺引发的中断，用户必须确保GPIO口处于稳定状态时执行本函数

### 2.注册GPIO中断的中断处理程序

```c
void GPIOIntRegister(uint32_t ui32Port, void (*pfnIntHandler)(void))

```

功能：注册GPIO中断的中断处理程序
参数:
（1）ui32Port ：GPIO口的基地址
（2）pfnIntHandler： 是GPIO中断服务程序入口地址指针。
说明：
（1）不管是什么外设触发的中断，都要先注册中断服务函数，告诉程序中断发生时去哪里，类似的函数有SysCtlIntRegister、ADCIntRegister等
（2）如果不利用这些中断注册函数，也可以在启动文件中修改中断向量表进行手动注册
（3）GPIOIntRegister只能以GPIO组为单位注册，不能精确到判断哪个引脚发生中断，因此要在中断服务函数中判断触发中断的引脚，以下为一个示例

```c
		//GPIOF中断服务函数
		void io_interrupt(void)
		{	
			//获取中断状态
			uint32_t s = GPIOIntStatus(GPIO_PORTF_BASE, true);
			//如果PF4触发中断
  			if((s&GPIO_PIN_4) == GPIO_PIN_4)
			{...}
		}
```

### 3.使能指定引脚的中断.

void GPIOIntEnable(uint32_t ui32Port, uint32_t ui32IntFlags)
功能：使能指定引脚的中断.
参数:
（1）ui32Port ：GPIO口的基地址
（2）ui32IntFlags： 被禁止的中断源中断屏蔽位(指示哪些引脚中断被开启，是以下参数的逻辑或)

```C
	#define GPIO_INT_PIN_0          0x00000001
		#define GPIO_INT_PIN_1          0x00000002
		#define GPIO_INT_PIN_2          0x00000004
		#define GPIO_INT_PIN_3          0x00000008
		#define GPIO_INT_PIN_4          0x00000010
		#define GPIO_INT_PIN_5          0x00000020
		#define GPIO_INT_PIN_6          0x00000040
		#define GPIO_INT_PIN_7          0x00000080
```

说明：这个函数是中断源级的中断使能控制

### 4.使能一个中断

```C
void IntEnable(uint32_t ui32Interrupt)
```

功能：使能一个中断
参数:
（1）ui32Interrupt 指定的被允许的中断.
说明：这个函数是中断控制器级的中断使能控制

### 5.使能处理器中断

```c
bool IntMasterEnable(void)
```

功能：使能处理器中断.
参数：无
说明：
（1）这是处理器级的中断使能控制，它决定处理器要不要处理中断控制器的请求
（2）以上三个函数，从低级到高级对应了中断处理通路的三道“开关”，如下图所示

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190709124218959.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d4Yzk3MTIzMQ==,size_16,color_FFFFFF,t_70) 

### 6.读取指定GPIO口的中断状态

说明：这个函数是中断源级的中断使能控制
（4）void IntEnable(uint32_t ui32Interrupt)
功能：使能一个中断
参数:
（1）ui32Interrupt 指定的被允许的中断.
说明：这个函数是中断控制器级的中断使能控制
（5）bool IntMasterEnable(void)
功能：使能处理器中断.
参数：无
说明：
（1）这是处理器级的中断使能控制，它决定处理器要不要处理中断控制器的请求
（2）以上三个函数，从低级到高级对应了中断处理通路的三道“开关”，如下图所示

（3）void GPIOIntEnable(uint32_t ui32Port, uint32_t ui32IntFlags)
功能：使能指定引脚的中断.
参数:
（1）ui32Port ：GPIO口的基地址
（2）ui32IntFlags： 被禁止的中断源中断屏蔽位(指示哪些引脚中断被开启，是以下参数的逻辑或)
		#define GPIO_INT_PIN_0          0x00000001
		#define GPIO_INT_PIN_1          0x00000002
		#define GPIO_INT_PIN_2          0x00000004
		#define GPIO_INT_PIN_3          0x00000008
		#define GPIO_INT_PIN_4          0x00000010
		#define GPIO_INT_PIN_5          0x00000020
		#define GPIO_INT_PIN_6          0x00000040
		#define GPIO_INT_PIN_7          0x00000080
1
2
3
4
5
6
7
8
说明：这个函数是中断源级的中断使能控制
（4）void IntEnable(uint32_t ui32Interrupt)
功能：使能一个中断
参数:
（1）ui32Interrupt 指定的被允许的中断.
说明：这个函数是中断控制器级的中断使能控制
（5）bool IntMasterEnable(void)
功能：使能处理器中断.
参数：无
说明：
（1）这是处理器级的中断使能控制，它决定处理器要不要处理中断控制器的请求
（2）以上三个函数，从低级到高级对应了中断处理通路的三道“开关”，如下图所示

```c
uint32_t GPIOIntStatus(uint32_t ui32Port, bool bMasked)
```

功能：读取指定GPIO口的中断状态
参数：
（1）ui32Port： GPIO口的基地址.
（2）bMasked： 指定返回屏蔽的中断状态还是原始的中断状态
说明： 如果bMasked被设置为真，则函数返回被屏蔽的中断状态，否则返回原始的中断状态。解释一下所谓“被屏蔽的中断状态”。在GPIOIntEnable这个函数中，没有写在第二个参数ui32IntFlags中的引脚是被屏蔽的（即不处理它们的中断事件）。当bMasked为真时，返回GPIOMIS寄存器值，所有被屏蔽的位都是0，否则返回GPIORIS寄存器值，被屏蔽的位也可能是1（因为虽然不处理这些引脚的中断事件，但它们的输入也可能符合中断特征）

返回值：返回指定GPIO口当前的中断状态，返回值是当前有效的GPIO_INT_∗values的逻辑或. ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190709125622852.png) 

### 7.清除指定中断源标志

```C
void GPIOIntClear(uint32_t ui32Port, uint32_t ui32IntFlags)
```

功能：清除指定中断源标志
参数:
（1）ui32Port ：GPIO口的基地址
（2）ui32IntFlags ：被清除的中断源中断屏蔽位
发生中断后，对应的中断标志位置1，进入中断服务函数，在服务函数中务必清除中断标志，否则程序将不停地进入中断服务函数

## 示例代码

```c
#include <stdint.h>
#include <stdbool.h>
#include "inc/tm4c123gh6pm.h"				//Register Definitions
#include "inc/hw_memmap.h"
#include "inc/hw_types.h"
#include "driverlib/sysctl.h"
#include "driverlib/interrupt.h"
#include "driverlib/gpio.h"
#include "driverlib/uart.h"
#include "uartstdio.h"
#include "driverlib/systick.h"
#include "driverlib/pin_map.h"


#define delay_ms(n); SysCtlDelay(n*(SysCtlClockGet()/3000));

void ButtonsInit(void);
void io_interrupt(void);

int main()
{
	//使能时钟
	SysCtlClockSet(SYSCTL_SYSDIV_4|SYSCTL_USE_PLL|SYSCTL_XTAL_16MHZ|SYSCTL_OSC_MAIN);
	SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOF);
	
	//配置PF1/PF3为输出,点亮绿灯（PF3）
	GPIOPinTypeGPIOOutput(GPIO_PORTF_BASE,GPIO_PIN_3|GPIO_PIN_1);
	GPIOPinWrite(GPIO_PORTF_BASE,GPIO_PIN_3|GPIO_PIN_1,1<<3);
	
	//按键使能
	ButtonsInit();

	while(1)
	{
		;	
	}		
}

/******************************************************************************************************************
*函数名: ButtonsInit
*描	 述：按键初始化函数
                                                                                                                                                                                                                                                                                              *输	 入：无
*线  路：PF0<->SW2
	     PF4<->SW1
******************************************************************************************************************/
void ButtonsInit(void)
{
	//使能PF时钟
	SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOF);
	
	//设置PF4为输入，上拉（没按就是高电平，按下就是低电平）
	GPIODirModeSet(GPIO_PORTF_BASE, GPIO_PIN_4, GPIO_DIR_MODE_IN);	
	
	//方向为输入
	GPIOPadConfigSet(GPIO_PORTF_BASE, GPIO_PIN_4, GPIO_STRENGTH_2MA, GPIO_PIN_TYPE_STD_WPU);//推挽上拉

	//PF4配置为下降沿中断
	GPIOIntTypeSet(GPIO_PORTF_BASE, GPIO_PIN_4, GPIO_FALLING_EDGE);//下降沿
	
	//给PF组注册一个中断函数
	GPIOIntRegister(GPIO_PORTF_BASE, io_interrupt);
	
	//开启PF4的中断
	GPIOIntEnable(GPIO_PORTF_BASE, GPIO_INT_PIN_4);
	IntEnable(INT_GPIOF);
	IntMasterEnable();
}


/******************************************************************************************************************
*函数名: io_interrupt
*描	 述：GPIO外部中断处理函数
*输	 入：无
*线  路：PF4<->SW1
******************************************************************************************************************/
void io_interrupt(void)
{
	static uint8_t trigger=0;
	
	//获取中断状态
	uint32_t s = GPIOIntStatus(GPIO_PORTF_BASE, true);
	//清除发生的中断标志
 	GPIOIntClear(GPIO_PORTF_BASE, s);
    
  	if((s&GPIO_PIN_4) == GPIO_PIN_4)
  	{
		while(!GPIOPinRead(GPIO_PORTF_BASE, GPIO_PIN_4));//等待按键松开
		
		if(!trigger)
			trigger=1,GPIOPinWrite(GPIO_PORTF_BASE,GPIO_PIN_3|GPIO_PIN_1,1<<1);//点亮红灯（PF1）
		else
			trigger=0,GPIOPinWrite(GPIO_PORTF_BASE,GPIO_PIN_3|GPIO_PIN_1,1<<3);//点亮绿灯（PF3）
	}
}

```

# UART

## 1.Tiva控制器的UART特征

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190712154843399.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d4Yzk3MTIzMQ==,size_16,color_FFFFFF,t_70) 

## 2.UART结构图

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190712155058934.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d4Yzk3MTIzMQ==,size_16,color_FFFFFF,t_70) 

UART和引脚的复用映射表

 这个表非常重要，编程时要根据这个参考 

（TM4C123GXL板子：串口0的接收端不能用，串口1正常，串口2也不正常，涉及到芯片部分IO解锁的东西）

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190712155235173.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d4Yzk3MTIzMQ==,size_16,color_FFFFFF,t_70) 

## 3.FIFO操作

 看来许多人还没有真正理解FIFO的作用和优点，仍然停留在每收发一个字符就要中断处理一次的老思路上。UART收发FIFO主要是为了解决收发中断过于频繁而导致的CPU效率不高的问题。

　　FIFO的必要性。在进行UART通信时，中断方式比轮询方式要简便且效率高。但是，如果没有收发FIFO，则每传输一个数据（5～8位）都要中断处理一次，效率仍然不高。如果有了收发FIFO，则可以在连续收发若干个数据（可多至14个）后才产生一次中断，然后一起处理。这就大大提高了收发效率。

　　接收超时问题。如果没有接收超时功能，则在对方已经发送完毕而接收FIFO未填满时并不会触发中断（FIFO满才会触发中断），结果造成最后接收的有效数据得不到处理的问题。有了接收超时功能后，如果接收FIFO未填满而对方发送已经停，则在不超过3个数据的接收时间内就会触发超时中断，因此数据会照常得到处理。

　　总之，FIFO的设计是优秀而合理的，它已经帮你想到了收发过程中存在的任何问题，只要初始化配置UART后，就可以放心收发了，FIFO和中断例程会自动搞定一切！

　　完全不必要担心FIFO大大减少了中断产生的次数而“可能”造成数据丢失的问题！

　　发送时，只要发送FIFO不满，数据只管往里连续放，放完后就直接退出发送子程序。随后，FIFO真正发送完成后会自动产生中断，通知主程序说：我已经完成真正的发送。

　　接收时，如果对方是连续不间断发送，则填满FIFO后会以中断的方式通知主程序说：现在有一批数据来了，请处理。

　　如果对方是间断性发送，也不要紧，当间隔时间过长时（2～3个字符传输时间），也会产生中断，这次是超时中断，通知主程序说：对方可能已经发送完毕，但FIFO未满，也请处理。


每个UART有两个16x8的缓冲区，一个用来发送，一个用来接收

FIFO状态通过UART标志寄存器UARTFR和UART接收状态寄存器（UARTRSR）显示。硬件监视空、满和溢出情况。

FIFO产生中断的触发条件由 UART中断FIFO深度选择（UARTIFLS）控制。两个FIFO可以单独配置为不同的电平情况下触发中断。可以选择如下配置：1/8、1/4、1/2、3/4、7/8。（例如：1/4代表连续FIFO装入16*1/4=4个字节产生一个接受中断）

复位后FIFO都是禁用的并作为1字节的保留寄存器，FIFO中断默认为两个都是1/2选项。通过UARTLCRH的FEN位启用FIFO。

注意：关于复位后FIFO是默认使能还是禁用，似乎TI手册之间有矛盾，总之不要用默认设置，自己手动设置一下吧
以下内容来自ti的getting start手册

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190712164442900.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d4Yzk3MTIzMQ==,size_16,color_FFFFFF,t_70) 

### 相关库函数

 \1. UARTConfigSet()
　　配置UART，例如：
  // 配置UART2：波特率9600，数据位8，停止位1，无校验
  UARTConfigSet(UART2_BASE, 9600, UART_CONFIG_WLEN_8 |
                  UART_CONFIG_STOP_ONE |
                  UART_CONFIG_PAR_NONE);

\2. UARTFIFOLevelSet()
　　设置UART收发FIFO的深度，可以设置的深度有2、4、8、12、14

\3. UARTSpaceAvail()
　　确认在发送FIFO里是否有可利用的空间。

\4. UARTCharsAvail()
　　确认在接收FIFO里是否存在字符。

\5. UARTCharPutNonBlocking()
　　该函数要与UARTSpaceAvail()配合使用，如果已确认发送FIFO里有可用空间，则将字符直接放入发送FIFO，不等待。

\6. UARTCharGetNonBlocking()
　　该函数要与UARTCharsAvail()配合使用，如果已确认接收FIFO里有字符，则直接从接收FIFO里读取字符，不等待。

\7. UARTCharPut()
　　将字符放到发送FIFO里，如果没有可用空间则一直等待。

\8. UARTCharGet()
　　从接收FIFO里读取字符，如果没有字符则一直等待。

\9. UARTIntEnable()
　　使能一个或多个UART中断，例如：
  // 同时使能接收中断(接收FIFO溢出)和接收超时中断
  UARTIntEnable(UART2_BASE, UART_INT_RX | UART_INT_RT); 

## 4.中断

 如下介绍，建议结合相关的寄存器看，包括`UARTCTL寄存器`和`UARTIFLS寄存器` 

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190712163004163.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d4Yzk3MTIzMQ==,size_16,color_FFFFFF,t_70) 

- 接收超时中断：当FIFO不是空的，并且在32位期间没有接收到进一步的数据时触发
- 接收中断：如果开启FIFO，当收数据>=FIFO深度时触发；否则每次接收到数据都触发
- 发送中断：如果开启FIFO，当FIFO中数据<=FIFO深度时触发；否则在FIFO中没有数据时触发

## 5.示例代码

```c
/*官方例程1*/
#include <stdint.h>
#include <stdbool.h>
#include "inc/hw_memmap.h"
#include "inc/hw_types.h"
#include "driverlib/gpio.h"
#include "driverlib/pin_map.h"
#include "driverlib/sysctl.h"
#include "driverlib/uart.h"
int main(void)
{
    //时钟分频  400/2/4=50
	SysCtlClockSet(SYSCTL_SYSDIV_4 | SYSCTL_USE_PLL | SYSCTL_OSC_MAIN | SYSCTL_XTAL_16MHZ);
    
    //使能GPIOA,UART0外设
	SysCtlPeripheralEnable(SYSCTL_PERIPH_UART0);
	SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOA);
    
    //GPIO模式配置 PA0--RX  PA1--TX
	GPIOPinConfigure(GPIO_PA0_U0RX);
	GPIOPinConfigure(GPIO_PA1_U0TX);
    
    //GPIO的UART模式
	GPIOPinTypeUART(GPIO_PORTA_BASE, GPIO_PIN_0 | GPIO_PIN_1);
    
    //UART协议配置 波特率115200 8位 1停止位 无校验位
	UARTConfigSetExpClk(UART0_BASE, SysCtlClockGet(), 115200,(UART_CONFIG_WLEN_8 | UART_CONFIG_STOP_ONE | UART_CONFIG_PAR_NONE));
	
	UARTCharPut(UART0_BASE, 'E');
	UARTCharPut(UART0_BASE, 'n');
	UARTCharPut(UART0_BASE, 't');
	UARTCharPut(UART0_BASE, 'e');
	UARTCharPut(UART0_BASE, 'r');
	UARTCharPut(UART0_BASE, ' ');
	UARTCharPut(UART0_BASE, 'T');
	UARTCharPut(UART0_BASE, 'e');
	UARTCharPut(UART0_BASE, 'x');
	UARTCharPut(UART0_BASE, 't');
	UARTCharPut(UART0_BASE, ':');
	UARTCharPut(UART0_BASE, ' ');
	while (1)
	{
		if (UARTCharsAvail(UART0_BASE)) //判断FIFO内是否有数据
			UARTCharPut(UART0_BASE, UARTCharGet(UART0_BASE));
	} 
    //UARTCharPut：等待从指定端口发送字符
    //UARTCharGet：等待来自端口的字符（FIFO）
}

```



```c
static void Uart1_Init(void){
//使能GPIOB,UART1外设
  SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOB);	
  SysCtlPeripheralEnable(SYSCTL_PERIPH_UART1);
  
     //GPIO模式配置 PA0--RX  PA1--TX
    GPIOPinConfigure(GPIO_PB0_U1RX);
    GPIOPinConfigure(GPIO_PB1_U1TX);
    
    //GPIO的UART模式
  GPIOPinTypeUART(GPIO_PORTB_BASE, GPIO_PIN_0 | GPIO_PIN_1);
	
     //UART协议配置 波特率115200 8位 1停止位 无校验位
	UARTConfigSetExpClk(UART1_BASE, SysCtlClockGet(), 115200,(UART_CONFIG_WLEN_8|UART_CONFIG_STOP_ONE|UART_CONFIG_PAR_NONE));
	
    //禁用FIFO，默认FIFO Level为4/8 寄存器满8字节后产生中断，禁用后接收1位就产生中断
	UARTFIFODisable(UART1_BASE);
    
    //使能UART接收中断
  UARTIntEnable(UART1_BASE,UART_INT_RX);
    
    //UART中断地址注册
  UARTIntRegister(UART1_BASE,UART1_IRQHandler);
    
    //优先级分组
  IntPrioritySet(INT_UART1, USER_INT5);
}
void UART1_IRQHandler(void)//UART中断函数
{				
  //获取中断标志 原始中断标志 不屏蔽中断标志		
  uint32_t flag = UARTIntStatus(UART1_BASE,1);
    
  //清除中断标志
  UARTIntClear(UART1_BASE,flag);
    
  //判断FIFO是否还有数据		
  while(UARTCharsAvail(UART1_BASE))		
  {			
    char ch = UARTCharGet(UART1_BASE);
//		UARTCharPut(UART1_BASE, 0x05);
  }
}


```



# PWM

 TM4C123GH6PM控制器包含两个pwm模块，每个模块由4个pwm发生器和一个控制模块组成，每个发生器可以产生2个pwm信号，**一共可以输出16个**pwm信号（**同一发生器产生的两个信号的周期是一致的，但占空比可以设为不同的**） 

## PWM发生器特点

 ![人生自古谁无死，留取丹心照汗青](https://img-blog.csdnimg.cn/20190710181953400.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d4Yzk3MTIzMQ==,size_16,color_FFFFFF,t_70) 

## PWM结构图

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190710182204486.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d4Yzk3MTIzMQ==,size_16,color_FFFFFF,t_70) 

 其中一个发生器的细节 

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190710182233264.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d4Yzk3MTIzMQ==,size_16,color_FFFFFF,t_70) 

## PWM引脚映射情况

 **它记录了在硬件层面上，哪些pwm信号输出连接到哪些引脚**，编程时需要对照查看 

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190710182803366.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d4Yzk3MTIzMQ==,size_16,color_FFFFFF,t_70) 

## PWM模块时钟来源

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190708100659281.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d4Yzk3MTIzMQ==,size_16,color_FFFFFF,t_70) 

 查看原理图，可见pwm模块时钟来源于经过`USEPWMDIV`分频的系统时钟，所有pwm信号的时钟频率都是这个。 

## PWM信号产生过程

（1）类似stm32，TM4C的pwm利用定时器实现(不过TM4C的pwm模块中有自带的定时器，不需要想stm32那样使用timer外设)，可以选择三种计数模式

向上计数（pwm信号右对齐）
向下计数（pwm信号左对齐）
上下计数（pwm信号中间对齐）
（2）每个pwm信号发生器，可以配置两个pwm比较器（类似stm32中的ccrx），比较器根据设定的比较值和当前计数值输出高电平脉冲
（3）所有计数器、比较器的信息会被pwm信号发生器检测，并生成对应的pwm波

## PWM配置过程及示例代码

配置一个PWM发生器，频率25KHz，信号0（MnPWM0）占空比25%，信号1（MnPWM1）占空比75%，假定系统时钟频率为20M
使能PWM时钟 SysCtlPeripheralEnable()

使能被复用引脚的时钟 SysCtlPeripheralEnable()

使能引脚复用PWM功能 GPIOPinTypePWM()

将PWM信号分配到合适的引脚上 GPIOPinConfigure()

使能PWM时钟，设置PWM分频器为2分频（PWM时钟源为10M） SysCtlPWMClockSet();

配置为向下计数，参数立即更新模式 PWMGenConfigure()

设置周期时间（定时器计数范围），目标频率25K，PWM频率10M，则每一个信号周期有400个PWM周期，故装载值设为400-1（0到399共400个值） PWMGenPeriodSet()

设置信号0占空比25%，信号1占空比75% PWMPulseWidthSet()

启动PWM发生器的定时器 PWMGenEnable()

使能PWM输出 PWMOutputState()

```c
#include <stdint.h>
#include <stdbool.h>
#include "inc/tm4c123gh6pm.h"				//Register Definitions
#include "inc/hw_memmap.h"
#include "inc/hw_types.h"
#include "driverlib/sysctl.h"
#include "driverlib/interrupt.h"
#include "driverlib/gpio.h"
#include "driverlib/uart.h"
#include "uartstdio.h"
#include "driverlib/systick.h"
#include "driverlib/pin_map.h"
#include "driverlib/pwm.h"


#define delay_ms(n); SysCtlDelay(n*(SysCtlClockGet()/3000));

/******************************************************************************************************************
*函数名: PWMInit()
*描	 述：PWM初始化函数
*输	 入：无
*线  路：PF2<->M1PWM6
				 PF4<->M1PWM7
******************************************************************************************************************/
void PWMInit(void)
{
	//配置PWM时钟（设置USEPWMDIV分频器）
	SysCtlPWMClockSet(SYSCTL_PWMDIV_1);																									//PWM时钟 16M

	//使能时钟
	SysCtlPeripheralEnable(SYSCTL_PERIPH_PWM1);			//使能PWM模块1时钟																		
	SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOF);		//使能GPIOF时钟																		
	
	//使能引脚复用PWM功能
	GPIOPinTypePWM(GPIO_PORTF_BASE,GPIO_PIN_2);
	GPIOPinTypePWM(GPIO_PORTF_BASE,GPIO_PIN_3);
	
	//PWM信号分配
	GPIOPinConfigure(GPIO_PF2_M1PWM6);					//PF2->PWM模块1信号6																							
	GPIOPinConfigure(GPIO_PF3_M1PWM7);					//PF3->PWM模块1信号7																								

	//配置PWM发生器
	//模块1->发生器3->上下计数，不同步
	PWMGenConfigure(PWM1_BASE,PWM_GEN_3,PWM_GEN_MODE_UP_DOWN|PWM_GEN_MODE_NO_SYNC);	
	
	//配置PWM周期
	PWMGenPeriodSet(PWM1_BASE,PWM_GEN_3,64000);			//64*10^3/16/10^6=4ms																			
	
	//配置PWM占空比
	PWMPulseWidthSet(PWM1_BASE,PWM_OUT_6,PWMGenPeriodGet(PWM1_BASE, PWM_GEN_3)*0.01);		//比较值为四分之一总计数值，25%
	PWMPulseWidthSet(PWM1_BASE,PWM_OUT_7,PWMGenPeriodGet(PWM1_BASE, PWM_GEN_3)*0.01);		//比较值为四分之三总计数值，75%

	//使能PWM模块1输出
	PWMOutputState(PWM1_BASE,PWM_OUT_6_BIT,true);
	PWMOutputState(PWM1_BASE,PWM_OUT_7_BIT,true);
	
	//使能PWM发生器
	PWMGenEnable(PWM1_BASE,PWM_GEN_3);
}



/******************************************************************************************************************
*函数名: SetDuty(uint32_t ui32Base,uint32_t ui32PWMOut,float duty)
*描	 述：PWM初始化函数
*输	 入：PWM模块编号、信号编号、占空比
******************************************************************************************************************/
void SetDuty(uint32_t ui32Base,uint32_t ui32PWMOut,float duty)
{
		uint32_t ui32Gen;
		uint32_t ui32OutBits;
	
		switch(ui32PWMOut)
		{
			case PWM_OUT_0:	ui32Gen=PWM_GEN_0,ui32OutBits=PWM_OUT_0_BIT;	break;
			case PWM_OUT_1:	ui32Gen=PWM_GEN_0,ui32OutBits=PWM_OUT_1_BIT;	break;
			case PWM_OUT_2:	ui32Gen=PWM_GEN_1,ui32OutBits=PWM_OUT_2_BIT;	break;
			case PWM_OUT_3:	ui32Gen=PWM_GEN_1,ui32OutBits=PWM_OUT_3_BIT;	break;
			case PWM_OUT_4:	ui32Gen=PWM_GEN_2,ui32OutBits=PWM_OUT_4_BIT;	break;
			case PWM_OUT_5:	ui32Gen=PWM_GEN_2,ui32OutBits=PWM_OUT_5_BIT;	break;
			case PWM_OUT_6:	ui32Gen=PWM_GEN_3,ui32OutBits=PWM_OUT_6_BIT;	break;
			case PWM_OUT_7:	ui32Gen=PWM_GEN_3,ui32OutBits=PWM_OUT_7_BIT;	break;
		}
	
    //配置占空比
    PWMPulseWidthSet(ui32Base, ui32PWMOut, PWMGenPeriodGet(ui32Base, ui32Gen)*duty);
    PWMOutputState(ui32Base, ui32OutBits, true);
    //使能发生器模块
    PWMGenEnable(ui32Base, ui32Gen);
}

int main()
{
	SysCtlClockSet(SYSCTL_SYSDIV_1|SYSCTL_USE_OSC|SYSCTL_XTAL_16MHZ|SYSCTL_OSC_MAIN);		//系统时钟16M
	PWMInit();
	
	float duty1=0.1,duty2=0.9;
	float d=0.01;
	
	while(1)
	{
		SetDuty(PWM1_BASE,PWM_OUT_6,duty1);
		SetDuty(PWM1_BASE,PWM_OUT_7,duty2);
		delay_ms(10);
		duty1+=d;
		duty2-=d;
		
		if(duty1>=0.95 && duty2<=0.05)
			d=-0.01;
		else if(duty2>=0.95 && duty1<=0.05)
			d=0.01;
	}
}
```





## PWM_Init

```c
void PWM_Init(void)
{
	
  SysCtlPWMClockSet(SYSCTL_PWMDIV_8); // Set divider to 80M/8=10M  1/10M=0.1us  设置PWM时钟配置
  SysCtlPeripheralEnable(SYSCTL_PERIPH_PWM1); //启用外设（在上电时，所有外设被设置为禁用），此处启用外设PWM1
  SysCtlDelay(2); //启用外设后插入几个周期，使得时钟完全激活  
  SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOF); // 此处使能外设GPIOF

  SysCtlDelay(2);//启用外设后插入几个周期，使得时钟完全激活 
  // 使用备用功能
  GPIOPinConfigure(GPIO_PF2_M1PWM6);//此处将M1PWM6复用到PF2引脚
  GPIOPinConfigure(GPIO_PF3_M1PWM7);//此处将M1PWM7复用到PF3引脚

	

  // 使用引脚与PWM外设
  GPIOPinTypePWM(GPIO_PORTF_BASE, GPIO_PIN_2);//M1PWM6 PF2
  GPIOPinTypePWM(GPIO_PORTF_BASE, GPIO_PIN_3);//M1PWM7 PF3


  // 配置PWM发生器倒计时模式与即时更新的参数
  PWMGenConfigure(PWM1_BASE, PWM_GEN_2, PWM_GEN_MODE_DOWN | PWM_GEN_MODE_NO_SYNC);
  PWMGenConfigure(PWM1_BASE, PWM_GEN_3, PWM_GEN_MODE_DOWN | PWM_GEN_MODE_NO_SYNC);
  

  // 配置PWM发生器的周期
  PWMGenPeriodSet(PWM1_BASE, PWM_GEN_2, 1000); // Set the period
  PWMGenPeriodSet(PWM1_BASE, PWM_GEN_3, 1000);
  

  //使能PWM发生器的定时/计数器  
  PWMGenEnable(PWM1_BASE, PWM_GEN_2);
  PWMGenEnable(PWM1_BASE, PWM_GEN_3);
  

  // 使能PWM输出
  PWMOutputState(PWM1_BASE, PWM_OUT_6_BIT | PWM_OUT_7_BIT , true);

}
```

```c
SysCtlPWMClockSet(SYSCTL_PWMDIV_8);
// 设置PWM时钟配置
 SysCtlPeripheralEnable(SYSCTL_PERIPH_PWM1); 
//启用外设（在上电时，所有外设被设置为禁用），此处启用外设PWM1
 SysCtlDelay(2); 
//启用外设后插入几个周期，使得时钟完全激活 （该延时没有精确的时间划分）
GPIOPinConfigure(GPIO_PF2_M1PWM6);
//函数配置引脚复用器  此处将M1PWM复用到PF2引脚
 GPIOPinTypePWM(GPIO_PORTF_BASE, GPIO_PIN_2);
//此功能不能将任何引脚转换为PWM引脚，它只配置PWM引脚以正常工作。还需要GPIOPINConfigure()函数调用来正确配置PWM函数的引脚
PWMGenConfigure(PWM1_BASE, PWM_GEN_2, PWM_GEN_MODE_DOWN | PWM_GEN_MODE_NO_SYNC);
//参数：PWM模块基址，PWM发生器（PWM_GEN_0，PWM_GEN_1，PWM_GEN_2，PWM_GEN_3），PWM发生器配置
//将PWM发生器配置为倒计时模式，并立即更新参数
PWMGenPeriodSet(PWM1_BASE, PWM_GEN_2, 1000); //配置PWM发生器的周期  参数：PWM模块基址，要修改的PWM发生器，指定PWM发生器输出的周期
PWMGenEnable(PWM1_BASE, PWM_GEN_2);//使能PWM发生器的定时/计数器
  PWMOutputState(PWM1_BASE, PWM_OUT_6_BIT | PWM_OUT_7_BIT , true);//使能或者禁用PWM输出
```

## PWM_Set

```c
void PWM_Set(uint16_t motor1,uint16_t motor2)
{
  PWMPulseWidthSet(PWM1_BASE,PWM_OUT_6,motor1);     //PF2
  PWMPulseWidthSet(PWM1_BASE,PWM_OUT_7,motor2);     //PF3
}
```

```c
 PWMPulseWidthSet(PWM1_BASE,PWM_OUT_6,motor1);
//设置指定PPWM输出的脉冲宽度
```

# 输入捕获

## 输入捕获引脚映射关系

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190712185612222.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d4Yzk3MTIzMQ==,size_16,color_FFFFFF,t_70) 

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190712185639520.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d4Yzk3MTIzMQ==,size_16,color_FFFFFF,t_70) 

## 边沿计数模式

 在该模式中，TimerA或TimerB被配置为能够捕获外部输入脉冲边沿事件的递增/减计数器。共有3种边沿事件类型：正边沿、负边沿、双边沿 

工作过程是：
（1）减计数：设置装载值Preload，并预设一个匹配值Match（应当小于装载值）；计数使能后，在特定的CCP管脚每输入1个脉冲（正边沿、负边沿或双边沿有效），计数值就减1；当计数值与匹配值Match相等时停止运行（若使能中断，这时会被触发）。定时器自动重装载Preload值，但如果需要再次捕获外部脉冲，则要重新进行配置。
（2）加计数：设置匹配值Match；计数使能后，在特定的CCP管脚每输入1个脉冲（正边沿、负边沿或双边沿有效），计数值就加1；当计数值与匹配值Match相等时停止运行（若使能中断，这时会被触发）。定时器自动清0并自动开始捕获外部脉冲计数，不需要重新进行配置

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190712192258596.png) 


注意：

配置为边沿计数模式时，定时器必须配置为拆分模式，64-bit未拆分模式下不可以用Capture
此模式下，8bit的定时器预分频寄存器(Prescaler)不再作为分频器使用，定时器频率和系统时钟频率相同， 预分频寄存器的8bit空间作为计数范围的扩展，增加到定时器计数器的高位。也就是说32/64位定时器的输入捕获计数范围为24/48bit
配置过程：
GPIO设置：

1.GPIOPinConfigure 进行引脚到TnCCPm的信号映射
2.GPIOPinTypeTimer 配置引脚到定时器模式
3.GPIOPadConfigSet 配置其他引脚参数

配置定时器模块为捕捉-边沿计数模式：

4. TimerConfigure 配置定时器模式。注意第二个参数一定是TIMER_CFG_SPLIT_PAIR和一下之一相或
   （1）TIMER_CFG_A_CAP_COUNT 模块A捕捉-边沿减计数模式
   （2）TIMER_CFG_A_CAP_COUNT_UP 模块A捕捉-边沿加计数模式
   （3）TIMER_CFG_B_CAP_COUNT 模块B捕捉-边沿减计数模式
   （4）TIMER_CFG_B_CAP_COUNT_UP 模块B捕捉-边沿加计数模式

设置要捕捉的边沿：

5. TimerControlEvent

设置计数范围：

6. TimerMatchSet设置加/减计数结束值
7. TimerLoadSet设置减计数起始值

中断设置：

8. TimerIntRegister 注册中断服务函数
9. TimerIntEnable 源级中断使能，这里注意配置中断类型
   （1）TIMER_CAPA_MATCH – 模块A计数到达预设值
   （2）TIMER_CAPB_MATCH – 模块B计数到达预设值
10. IntEnable 中断控制器级中断使能
11. IntMasterEnable处理器级中断使能

启动定时器模块：

12. TimerEnable

## 边沿计时模式

简介：在该模式中，TimerA/B被配置为自由运行的16位递减计数器，允许在输入信号的上升沿或下降沿捕获事件。

工作过程是：设置装载值（默认为0xFFFF）、捕获边沿类型；计数器被使能后开始自由运行，从装载值开始递减计数（或从0开始递增计数），计数到0（或装载值）时重装（或清零），继续计数；如果从CCP管脚上出现有效的输入脉冲边沿事件，则当前计数值被自动复制到一个特定的寄存器里，该值会一直保存不变，直至遇到下一个有效输入边沿时被刷新。为了能够及时读取捕获到的计数值，应当使能边沿事件捕获中断，并在中断服务函数里读取。

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190712193416887.png) 


注意：

配置为边沿计时模式时，定时器必须配置为拆分模式，64-bit未拆分模式下不可以用Capture
此模式下，8bit的定时器预分频寄存器(Prescaler)不再作为分频器使用，定时器频率和系统时钟频率相同， 预分频寄存器的8bit空间作为计数范围的扩展，增加到定时器计时器的高位。也就是说32/64位定时器的输入捕获计时范围为24/48bit
配置过程：和计数模式基本一样，仅有以下区别
需要配置定时器模块为捕捉-边沿计时模式

TimerConfigure第二个参数是TIMER_CFG_SPLIT_PAIR和一下之一相或
（1）TIMER_CFG_A_CAP_TIME 模块A捕捉-边沿减计时模式
（2）TIMER_CFG_A_CAP_TIME _UP 模块A捕捉-边沿加计时模式
（3）TIMER_CFG_B_CAP_ TIME 模块B捕捉-边沿减计时模式
（4）TIMER_CFG_B_CAP_ TIME _UP 模块B捕捉-边沿加计时模式
设置计时范围

2. TimerLoadSet无论加计时还是减计时，都用这个函数设置Preload值

中断设置

3. TimerIntEnable的中断类型参数变为以下二者之一
   （1） TIMER_CAPA_EVENT 模块A发生捕获事件
   （2） TIMER_CAPB_EVENT 模块B发生捕获事件


## 示例代码

```c
//timer0A输入捕获，脉冲从PB6输入（PF0作T0CCP0）
int timer0A_cnt=0;
void Timer0Init()
{	
		// 启用Timer0模块   
    SysCtlPeripheralEnable(SYSCTL_PERIPH_TIMER0);
   
    // 启用GPIO_F作为脉冲捕捉脚   
    SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOB);
   
    // 配置GPIO脚为使用Timer4捕捉模式   
    GPIOPinConfigure(GPIO_PB6_T0CCP0);   
    GPIOPinTypeTimer(GPIO_PORTB_BASE, GPIO_PIN_6); 
   
    // 为管脚配置弱上拉模式（捕获下降沿，配置为上拉）
    GPIOPadConfigSet(GPIO_PORTB_BASE, GPIO_PIN_6, GPIO_STRENGTH_2MA, GPIO_PIN_TYPE_STD_WPU);  
   
    // 配置使用Timer4的TimerA模块为边沿触发加计数模式
    TimerConfigure(TIMER0_BASE, TIMER_CFG_SPLIT_PAIR | TIMER_CFG_A_CAP_COUNT_UP);
   
    // 使用下降沿触发   
    TimerControlEvent(TIMER0_BASE, TIMER_A, TIMER_EVENT_NEG_EDGE); 
   
    // 设置计数范围为0~9
    TimerMatchSet(TIMER0_BASE, TIMER_A, 10-1);		//理论匹配周期10^-4*10=0.001s
   
    // 注册中断处理函数以响应触发事件   
    TimerIntRegister(TIMER0_BASE, TIMER_A, Timer0AIntHandler);
   
    // 系统总中断开   
    IntMasterEnable();
   
    // 时钟中断允许，中断事件为Capture模式中边沿触发，计数到达预设值   
    TimerIntEnable(TIMER0_BASE, TIMER_CAPA_MATCH);
   
    // NVIC中允许定时器A模块中断   
    IntEnable(INT_TIMER0A);
   
    // 启动捕捉模块  
    TimerEnable(TIMER0_BASE, TIMER_A);
}


//中断服务函数 理论周期：0.001s
void Timer0AIntHandler(void)
{
	TimerIntClear(TIMER0_BASE, TIMER_CAPA_MATCH);
	timer0A_cnt++;
}
```

# Timer

## 单次运行模式和连续运行模式

 定时器的基本功能为计数(包括加计数和减计数两种)，Stellaris LM4F中以系统时钟为计数节拍(当计数器被拆分使用时可以使用预分频功能，为了简单起见这里不作讨论)。当加计数时，计数器由零开始，逐步加一，直到到达用户预设值；减计数则由某一用户预设值开始，逐步减一，直到计数为零。每当计数完成，则会置相应的状态位(包括中断)，提示计时完成。

单次运行与连续运行工作时没有区别，不同的是单次运行在完成一次计时后会自动停止，连续模式下定时器会自动从计数起点开始(根据计数方向为零或用户设定值)继续计时。 

## 定时器程序设置

1.启用时钟模块

 使用SysCtlPeripheralEnable函数启用相应的定时器模块。

程序示例：
     SysCtlPeripheralEnable(SYSCTL_PERIPH_WTIMER0);

在StellarisWare中，32/16-bit定时器模块为TIMER，64/32-bit定时器模块为WTIMER (Wide Timer)。除了名字不同、计数范围不同外没有其它区别。 

2.设置时钟模块工作模式

 使用TimerConfigure函数对定时器模块的工作模式进行设置，将其设置为定时器功能。

程序示例：
     TimerConfigure(TIMER0_BASE,TIMER_CFG_ONE_SHOT);
     TimerConfigure(WTIMER2_BASE,TIMER_CFG_SPLIT_PAIR | TIMER_CFG_A_ONE_SHOT |TIMER_CFG_B_PERIODIC);

在不拆分的情况下，可以用下面参数中的一个将模块设置成所需的定时器模式：
     TIMER_CFG_ONE_SHOT – 单次减计数模式
     TIMER_CFG_ONE_SHOT_UP – 单次加计数模式
     TIMER_CFG_PERIODIC – 连续减计数模式
     TIMER_CFG_PERIODIC_UP – 连续加计数模式
     TIMER_CFG_RTC – 实时时钟模式

如果需要将计时器拆分，则需使用参数TIMER_CFG_SPLIT_PAIR然后用“|”号连接被拆分定时器A、B的设置。如果只使用了一个可以只设置用到的那个。拆分出来的定时器A和B的设置方法是一样的，只是函数名中各自用A和B：
     TIMER_CFG_A_ONE_SHOT – 定时器A单次减计数
     TIMER_CFG_A_ONE_SHOT_UP –定时器A单次加计数
     TIMER_CFG_A_PERIODIC – 定时器A连续减计数
     TIMER_CFG_A_PERIODIC_UP – 定时器A连续加计数

​     TIMER_CFG_B_ONE_SHOT – 定时器B单次减计数
​     TIMER_CFG_B_ONE_SHOT_UP –定时器B单次加计数
​     TIMER_CFG_B_PERIODIC – 定时器B连续减计数
​     TIMER_CFG_B_PERIODIC_UP – 定时器B连续加计数 

3.设置时钟的计数范围

 使用TimerLoadSet、TimerLoadSet64函数可以为计数设置范围。设置未拆分使用的64/32-bit定时器模块，需要使用TimerLoadSet64函数，对其它模块、其它状况的设置使用TimerLoadSet函数。计数范围为设置值到零(加计数: 0~预设值，减计数: 预设值~0)。

程序示例：
     TimerLoadSet64(TIMER3_BASE, 80000);
     TimerLoadSet(WTIMER0_BASE, TIMER_B, 10000); 

4.启动时钟

 使用TimerEnable函数启动定时器。可以用的参数有TIMER_A、TIMER_B和TIMER_BOTH。可以分别或同时启动A、B定时器。如果定时器没有拆分，直接使用TIMER_A即可。

程序示例：
     TimerEnable(WTIMER0_BASE, TIMER_B); 

## 定时器读取及中断设置

1.计数值读取

 可以使用TimerValueGet函数和TimerValueGet64函数获得定时器当前的计数值。需要注意的是TimerValueGet64返回的是64位结果。

程序示例：
     long val = TimerValueGet(TIMER1_BASE, TIMER_A);
     long long timer_val = TimerValueGet64(WTIMER3_BASE); 

2.中断设置

 一般定时器多用中断响应以满足时间要求。可以用TimerIntRegister向系统注册中断处理函数，用TimerIntEnable来允许某个定时器的中断请求。需要注意的是，在M4中还应该用IntEnable在系统层使能定时器的中断。当然，系统总中断开关也必须用IntMasterEnable使能。

TimerIntEnable在该模式下可以支持：
     TIMER_TIMA_TIMEOUT
     TIMER_TIMB_TIMEOUT

程序示例：
     TimerIntRegister(WTIMER0_BASE, TIMER_B, WTimer0BIntHandler);
     IntMasterEnable();
     TimerIntEnable(WTIMER0_BASE, TIMER_TIMB_TIMEOUT);
     IntEnable(INT_WTIMER0B);

在Timer中断中，需要手工清除中断标志位，可以使用如下代码：
     unsignedlong ulstatus = TimerIntStatus(TIMER4_BASE, TIMER_TIMA_TIMEOUT |TIMER_TIMB_TIMEOUT);
     TimerIntClear(TIMER4_BASE,ulstatus); 

## 示例代码

```c
// Stellaris硬件定义及StellarisWare驱动定义头文件
#include "inc/hw_memmap.h"
#include "inc/hw_types.h"
#include "inc/hw_timer.h"
#include "inc/hw_ints.h"
#include "driverlib/timer.h"
#include "driverlib/interrupt.h"
#include "driverlib/sysctl.h"
#include "driverlib/gpio.h"
#include "utils/uartstdio.h"

// 用于记录进入定时器中断的次数
unsigned long g_ulCounter = 0;

// 初始化UART的函数
extern void InitConsole(void);

// 定时器的中断处理函数
Void WTimer0BIntHandler(void)
{  
    // 清除当前中断标志  
    TimerIntClear(WTIMER0_BASE, TIMER_TIMB_TIMEOUT);

    // 更新进入中断次数的计数  
    g_ulCounter++;
}

// 主程序
int main(void)
{   
    // 用来记录上次计数以判断是否计数改变
    unsigned long ulPrevCount = 0;
   
    // 设置LM4F时钟为50MHz
    SysCtlClockSet(SYSCTL_SYSDIV_4 | SYSCTL_USE_PLL | SYSCTL_OSC_MAIN | SYSCTL_XTAL_16MHZ);
  
    // 使能64/32-bit的时钟模块WTIMER0   
    SysCtlPeripheralEnable(SYSCTL_PERIPH_WTIMER0);

    // 初始化UART   
    InitConsole();

    // 打印程序信息   
    UARTprintf("32-Bit Timer Interrupt ->");   
    UARTprintf("
   Timer = Wide Timer0B");  
    UARTprintf("
   Mode = Periodic");   
    UARTprintf("
   Rate = 1s

");

    // 设置WTimer0-B模块为连续减计数   
    TimerConfigure(WTIMER0_BASE, TIMER_CFG_SPLIT_PAIR | TIMER_CFG_B_PERIODIC);

    // 设置定时器的计数值，这里用系统频率值，即每秒一个中断   
    TimerLoadSet(WTIMER0_BASE, TIMER_B, SysCtlClockGet());

    // 设置WTimer0-B的中断处理函数   
    TimerIntRegister(WTIMER0_BASE, TIMER_B, WTimer0BIntHandler);

    // 启用系统总中断开关  
    IntMasterEnable();

    // 启用WTimer0-B超时中断   
    TimerIntEnable(WTIMER0_BASE, TIMER_TIMB_TIMEOUT);

    // 在系统层面(NVIC)使能WTimer0-B中断   
    IntEnable(INT_WTIMER0B);

    // 启动定时器 
    TimerEnable(WTIMER0_BASE, TIMER_B);

    while(1)  
    {
        // 循环等待WTimer0-B中断更新g_ulCounter计数
        // 若计数改变则进行UART输出       
        if(ulPrevCount != g_ulCounter)
        {
            // UART输出计数值
            UARTprintf("Number of interrupts: %d
", g_ulCounter);

            ulPrevCount = g_ulCounter;       
        }   
    }
}

```

```c
static void Timer_Init(void){
	//定时器0使能
  SysCtlPeripheralEnable(SYSCTL_PERIPH_TIMER0);					
    //32位周期定时器
  TimerConfigure(TIMER0_BASE,TIMER_CFG_PERIODIC);				
    //设定装载值，（80M/1000）*1/80M=1ms
  TimerLoadSet(TIMER0_BASE,TIMER_A,SysCtlClockGet()/1000);		
    //总中断使能
  IntEnable(INT_TIMER0A);										
    //中断输出，设置模式
  TimerIntEnable(TIMER0_BASE,TIMER_TIMA_TIMEOUT); 				
    //中断函数注册
  TimerIntRegister(TIMER0_BASE,TIMER_A,TIMER0A_Handler);		
    //定时器使能开始计数
  TimerEnable(TIMER0_BASE,TIMER_A); 							
    //中断优先级划分
  IntPrioritySet(INT_TIMER0A,USER_INT7);
}



void TIMER0A_Handler(void){

	Scheduler_Run();
    
    //清除当前中断标志
	TimerIntClear(TIMER0_BASE,TIMER_TIMA_TIMEOUT);
}

```

# I2C

## 一、I2C接口的介绍：

   内部集成电路（I2C）总线通过一个两线设计（串行数据线 SDA 和串行时钟线 SCL）来提供双向数据传输，并且与外部 I2C 器件诸如串行存储器（RAM 和 ROM），网络设备，LCD，音频发生器等联系。I2C 总线也可用于产品开发和制造的系统测试和诊断的目的。TM4C123GH6PM 微控制器提供与其他 I2C 总线上的设备交互（发送和接收）的能力。TM4C123GH6PM 控制器的 I2C 模块具有以下特点：

 I2C 总线上的设备可被配置为主机或从机
— 支持一个主机或从机发送和接收数据
— 同时支持主机和从机操作
 四个 I2C 模式：
— 主机发送模式
— 主机接收模式
— 从机发送模式
— 从机接收模式
 四个发送速率：
— 标准模式（100 Kbps）
— 快速模式（400 Kbps）
— 超快速模式（1Mbps）
— 高速模式（3.33Mbps）
 时钟低超时中断的
 双从地址能力
 抗干扰
 主机和从机中断的产生
— 当主机发送或接收操作完成时（或因错误终止时），产生中断
— 当从机发送数据或主机需要数据或检测到起始或停止条件时，产生中断
 主机由仲裁和时钟同步，支持多主机，以及 7 位寻址模式

![1690613571472](C:\Users\86177\AppData\Roaming\Typora	ypora-user-images\1690613571472.png)

## 二、结构图：

 ![img](https://img-blog.csdnimg.cn/2019063015483772.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NDI3Njc4,size_16,color_FFFFFF,t_70) 

## 三、 初始化与配置

以下例子给出如何配置 I2C 模块用于主机传输一个字节。这里假定系统时钟为 20 MHz 。

1. 在系统控制模块使用 RCGCI2C 寄存器使能 I2C 时钟。
2. 通过在系统控制模块的 RCGCGPIO 寄存器为相应的 GPIO 模块使能时钟。要了解使能哪些GPIO 端口。
3. 在 GPIO 模块，通过 GPIOAFSEL 寄存器位它们的复用功能使能相应的引脚。请参看表 23-4，以确定配置那个 GPOI。
4. 使能 I2CSDA 引脚来配置开漏操作。
5. 在 GPIOPCTL 寄存器配置 PMCn 位组为相应的引脚配置 I2C 信号。
6. 向 I2CMCR 寄存器写入 0x00000010 值来初始化 I2C 主机。
7. 通过写入 I2CMTPR 寄存器正确的值来设置所需的 100 Kbps 的 SCL 时钟速度。写入I2CMTPR 寄存器的值代表在一个 SCL 时钟周期中系统时钟周期数。TPR 值由以下等式确定：
   TPR = (System Clock/(2*(SCL_LP + SCL_HP)*SCL_CLK))-1;
   TPR = (20MHz/(2*(6+4)*100000))-1;
   TPR = 9向 I2CMTPR 寄存器写入 0x00000009.
8. 规定主机的从机地址，下一个操作是一个发送，该发送通过向 I2CMSA 寄存器值写入0x00000076 实现。这设置了从机地址为 0x3B。
9. 通过向 I2CMDR 寄存器写入所需数据将数据（位）传输到数据寄存器。
10. 启动从主机到从机的数据单字节的传输是通过向 I2CMCS 寄存器写入 0x00000007 (STOP,START, RUN)实现。
11. 等待直到传输完成，通过轮询 I2CMCS 寄存器的 BUSBSTY 位，直到该位被清除。
12. 检测 I2CMCS 寄存器的 ERROR 位以确保传输被应答。

## 四、环路操作实验

​      可以通过设置 I2C 主机配置（I2CMCR）寄存器的 LPBK 位将 I2C 模块放入一个内部环路模式用于诊断或调试工作。在环回模式下，来自主机的 SDA 和 SCL 信号和从机模块的 SDA 和SCL 信号绑定在一起，并允许内部的设备测试，而不必去通过 I / O实验的验证就是通过主机发送数据-从机接收，从机发数据-主机接收

```c
/********************************************************
 *实验功能：通过环路模式实现单字节的主机/从机数据的发送和接收
 *时间：2019-6-30
 *作者：MountXing
 *******************************************************/
 
//包含一系列头文件
#include <stdbool.h>
#include <stdint.h>
#include "inc/hw_i2c.h"
#include "inc/hw_memmap.h"
#include "inc/hw_types.h"
#include "driverlib/gpio.h"
#include "driverlib/i2c.h"
#include "driverlib/pin_map.h"
#include "driverlib/sysctl.h"
#include "driverlib/uart.h"
#include "utils/uartstdio.h"
 
 
//要发送的I2C数据包的数量
#define NUM_I2C_DATA  5
 
//地址
#define SLAVE_ADDRESS 0x3C
 
/*****************************************************************************
 *初始化配置使用 UART0 外设
 *-UART0RX - PA0
 *-UART0TX - PA1
*****************************************************************************/
void Init_Console_Uart0(void)
{
   
    SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOA);
    GPIOPinConfigure(GPIO_PA0_U0RX);
    GPIOPinConfigure(GPIO_PA1_U0TX);
    SysCtlPeripheralEnable(SYSCTL_PERIPH_UART0);
    UARTClockSourceSet(UART0_BASE, UART_CLOCK_PIOSC);     //内部16M时钟
    GPIOPinTypeUART(GPIO_PORTA_BASE, GPIO_PIN_0 | GPIO_PIN_1);
    UARTStdioConfig(0, 115200, 16000000);
}
 
/*****************************************************************************
 *配置I2C0主从机，并使用环回模式连接它们。
 *初始化I2C0外设
 *I2C0SCL - PB2
 *I2C0SDA - PB3
 *****************************************************************************/
int main(void)
{
    uint32_t pui32DataTx[NUM_I2C_DATA];   //发送数据缓冲区
    uint32_t pui32DataRx[NUM_I2C_DATA];   //接受数据缓冲区
    uint32_t ui32Index;                   //要发送数据到缓冲区的位置
 
    //配置外部时钟为20M
    SysCtlClockSet(SYSCTL_SYSDIV_1|SYSCTL_USE_OSC|SYSCTL_OSC_MAIN|SYSCTL_XTAL_16MHZ);
    SysCtlPeripheralEnable(SYSCTL_PERIPH_I2C0);   //使能I2C0外设
    SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOB);  //使能PB端口
    //配置引脚的复用功能 PB2 PB3
    GPIOPinConfigure(GPIO_PB2_I2C0SCL);           
    GPIOPinConfigure(GPIO_PB3_I2C0SDA);
    //配置I2C引脚
    GPIOPinTypeI2CSCL(GPIO_PORTB_BASE, GPIO_PIN_2);
    GPIOPinTypeI2C(GPIO_PORTB_BASE, GPIO_PIN_3);
 
     //I2CLoopbackEnable(I2C0_BASE);     //调用固件库的环回模式。(I2C.c固件库里没有这个函数)
     HWREG(I2C0_BASE + I2C_O_MCR) |= 0x01; //调用寄存器的环回模式 (改用寄存器实现)
 
    //初始化并使能主机模式，使用系统时钟为 I2C0 模块提供时钟频率，
    //主机模块传输速//率为 100Kbps
    I2CMasterInitExpClk(I2C0_BASE, SysCtlClockGet(), false);
 
    // 使能从机模式
    I2CSlaveEnable(I2C0_BASE);
 
    //设置从机地址
    I2CSlaveInit(I2C0_BASE, SLAVE_ADDRESS);
 
    //设置主机放在总线上的地址，写入从机
    I2CMasterSlaveAddrSet(I2C0_BASE, SLAVE_ADDRESS, false);
 
    //调用uart初始化函数串口显示
    Init_Console_Uart0();
    UARTprintf("I2C 回送例子--->");
    UARTprintf("
   模式 = I2C0");
    UARTprintf("
   模式 = 单独的发送/接收");
    UARTprintf("
   比率 = 100kbps

");
 
    // 数据初始化发送.
    pui32DataTx[0] = 'I';
    pui32DataTx[1] = '2';
    pui32DataTx[2] = 'C';
    pui32DataTx[3] = 'O';
    pui32DataTx[4] = 'K';
   
 
    // 初始化接收缓冲区.
    for(ui32Index = 0; ui32Index < NUM_I2C_DATA; ui32Index++)
    {
        pui32DataRx[ui32Index] = 0;
    }
    UARTprintf("转换方式: 主机 -> 从机
");
 
    // 将I2C数据从主机发送到从机
    for(ui32Index = 0; ui32Index < NUM_I2C_DATA; ui32Index++)
    {
      UARTprintf("  发送: '%c'  . . .  ", pui32DataTx[ui32Index]);
 
        // 将要发送的数据放在数据寄存器中
        I2CMasterDataPut(I2C0_BASE, pui32DataTx[ui32Index]);
 
        //从机回显发送数据到主机
        I2CMasterControl(I2C0_BASE, I2C_MASTER_CMD_SINGLE_SEND);
 
        // 等待从机接收应答
        while(!(I2CSlaveStatus(I2C0_BASE) & I2C_SLAVE_ACT_RREQ))
        {
          
        }
 
        // 从从机寄存器读取数据
        pui32DataRx[ui32Index] = I2CSlaveDataGet(I2C0_BASE);
 
        // 等待主机接收应答
        while(I2CMasterBusy(I2C0_BASE))
        {
          
        }
 
        UARTprintf("接收: '%c'
", pui32DataRx[ui32Index]);
    }
 
    // 重置接收缓冲区
    for(ui32Index = 0; ui32Index < NUM_I2C_DATA; ui32Index++)
    {
        pui32DataRx[ui32Index] = 0;
    }
    
    UARTprintf("

转换方式: 从机 -> 主机
");
 
    //主机从该地址读取数据
    I2CMasterSlaveAddrSet(I2C0_BASE, SLAVE_ADDRESS, true);
 
    // 初始化I2C主机模块状态为单端接收
    I2CMasterControl(I2C0_BASE, I2C_MASTER_CMD_SINGLE_RECEIVE);
 
    // 等待主机请求从机发送数据
    while(!(I2CSlaveStatus(I2C0_BASE) & I2C_SLAVE_ACT_TREQ))
    {
      
    }
 
    for(ui32Index = 0; ui32Index < NUM_I2C_DATA; ui32Index++)
    {
 
      UARTprintf("  发送: '%c'  . . .  ", pui32DataTx[ui32Index]);
 
        //将要发送的数据放到从机数据寄存器中
        I2CSlaveDataPut(I2C0_BASE, pui32DataTx[ui32Index]);
 
        // 初始化I2C主机模块状态为单端接收
        I2CMasterControl(I2C0_BASE, I2C_MASTER_CMD_SINGLE_RECEIVE);
 
        // 等待从机发送完毕
        while(!(I2CSlaveStatus(I2C0_BASE) & I2C_SLAVE_ACT_TREQ))
        {
          
        }
 
        // 从主机数据寄存器读取数据
        pui32DataRx[ui32Index] = I2CMasterDataGet(I2C0_BASE);
        UARTprintf("接收: '%c'
", pui32DataRx[ui32Index]);
    }
 
    //显示实验结束
    UARTprintf("
传输实验结束.

");
    return 0;
}
```
