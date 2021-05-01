---
title: ucosⅡ移植mini2440及其应用
date: 2019-12-01 23:07:00
tags: [原创,嵌入式]
categories: '工学'
---

嵌入式实践的一些记录

<!--more-->

## 1 准备阶段

实验环境：Windows Xp

---

###  1.1 先了解一下mini2440的接口布局

![](https://tappat-1300227703.cos.ap-guangzhou.myqcloud.com/picture/ucos/2240port.png)

> 实践中至少需要接好`电源接口`、`串口1`、`USB Slave`

> 一般把选择开关设置为Nor Flash，进入BIOS模式
>
> > Nor Flash模式下，程序直接烧录到内存中，断电后即失效。若是选择NAND Flash，则能将程序断电后也保留。

---

### 1.2 步骤说明

由于环境等原因，我自己测试的时候按如下步骤才能顺利进行：

1. 插好必要接口，**此时不要打开电源**
2. 在电脑上安装好驱动
3. 打开超级终端
4. 打开DNW
5. **到这一步，再打开电源**

---

### 1.3 安装驱动

安装附盘中的FriendlyARM USB Download Driver（先不用接上板子）

> 注：原本想在win10上操作，然而win10系统下安装总是失败，更换设备也一样

---

### 1.4 检测驱动

选择开关Nor Flash，连接到PC，打开电源，按照硬件向导提示完成配置

> 注：这一步是在最后才出现的

---

### 1.5 设置超级终端

在WindowsXP环境下，在`开始-->程序-->附件-->通讯`中，打开`超级终端`

> 默认Telnet程序？ 选择“否”
>
> 位置信息   选择“取消”
>
> 确定取消吗？ 选择“是”
>
> 再点击 “是”
>
> 连接描述  随便输入一个名称
>
> 连接到...   这里直接选默认端口 COM:X(如果发现不是1~4的端口需要先去硬件里修改)

接下来是重点步骤

![](https://tappat-1300227703.cos.ap-guangzhou.myqcloud.com/picture/ucos/2440_zd.png)

这一步，**每秒位数(波特率)要设置为115200**，**数据流控制要选择无**，否则无法正常连接

接着再处理好弹出窗口就到超级终端传输页面了。

### 1.6 DNW设置

一开始要先设置好端口，点Configuration中的options进行设置

![](https://tappat-1300227703.cos.ap-guangzhou.myqcloud.com/picture/ucos/DNW_1.png)

> 根据mini2440用户手册说明，需要把下载地址改为0x30000000

可以看到DNW里只能检测4个端口，然而实际上板子经常会默认接到COM5或者COM6端口上

这种情况可以到设备管理器中，找到端口栏，在对应的端口上右键属性进行修改

![](https://tappat-1300227703.cos.ap-guangzhou.myqcloud.com/picture/ucos/2440port2.png)

看下COM1~COM4里面哪个未被占用就改成那个

> 一般情况下不需要更改COM口



### 1.7 下载测试

![](https://tappat-1300227703.cos.ap-guangzhou.myqcloud.com/picture/2440_2.png)

在超级终端输入`d`，进入待下载状态

---

然后打开DNW，在`USB Port`菜单中选择`Transmit`，选择附盘中的测试文件，下载完成后，如下

![](https://tappat-1300227703.cos.ap-guangzhou.myqcloud.com/picture/2440_3.jpg)





## 2 UCOS移植准备

系统环境：Windows Xp

编译软件：ADS1.2

### 2.1 工程文件

附带光盘文件中已经给我们准备好了ucos的工程文件，找到`uCos2`文件夹，备份一份，然后把文件夹属性中的`只读`去掉

---

### 2.2 编译

点`File-->Open`，找到`uCOS_2440.mcp`并选择。

接着点`Project-->Make`编译。

> 自己刚开始编译的时候，位于`\uCos2\S3C2440\source\`下的`ucos2_vga_1024x768_demo.c`这个文件各种报错，之后发现这文件对我们的实验并没有什么影响，于是删除就好了。

编译完成后，在`\uCos2\uCos_2440_Data\DebugRel`目录下会生成`2440ucos2.bin`，这个bin文件就可以直接像上面使用的测试文件那样下载到mini2440内存中运行了。

---

### 2.3 调整

我们实验所用的LCD屏幕型号是`W35`，是新出的一款产品，原始代码并没有这个型号屏幕的对应设置

> W35数据：Height 240 x Width 320

先前烧录的结果会发现画面显示不完整。

> 因为它使用的型号是T35

打开`\uCos2\S3C2440\includes\Option.h`，这个文件中包含了屏幕显示的相关参数设置

稍微浏览一下可以发现默认用的是LCD_T35，那我们就干脆修改T35的数据

网上找了一篇相关文章：[linux2.6.32.2 mini2440平台移植-- LCD 显示驱动 ( W35屏 )](https://blog.csdn.net/dengwenq/article/details/7904149)

参考里面的数据把相应的那段代码改成这样：

~~~c
#elif defined(LCD_T35)
#define LCD_WIDTH 320
#define LCD_HEIGHT 240
#define LCD_PIXCLOCK 4 //这个按原本的，不修改

#define LCD_RIGHT_MARGIN 68
#define LCD_LEFT_MARGIN 66
#define LCD_HSYNC_LEN 4

#define LCD_UPPER_MARGIN 4
#define LCD_LOWER_MARGIN 4
#define LCD_VSYNC_LEN 9
#define LCD_CON5 ( (1 << 11)| (1<<0) | (1 << 8) | (1 << 6) | (1 << 9) | ( 1<< 10)) //这个也不修改
~~~

改完重新编译，下载。

![](https://tappat-1300227703.cos.ap-guangzhou.myqcloud.com/picture/ucos/2440_desktop.jpg)

---

### 2.4 小灯测试

打开`\uCos2\S3C2440\source\main.c`，这里面有两个任务段

看下Task1

~~~c
void Task1	(void *pdata) //task for test
{
	
	
	U16 TestCnt=0;
	U16 Version;


	Version=OSVersion();	
	
			
	while (1)
	{
	
	 TestCnt++;
     OSPrintf("********************************\n");
     OSPrintf("Enter Task1 Cnt=%d\n",TestCnt);	
     OSPrintf("Enter Task1\n");
     OSPrintf("uC/OS Version:V%4.2f\n",(float)Version*0.01);//ucos version 
     OSPrintf("********************************\n");
     
     //led 
     if(TestCnt%2)
     	rGPBDAT = 0x0000;
     else
    	rGPBDAT = 0x07fe;
    	
     OSTimeDly(OS_TICKS_PER_SEC);

	}
}
~~~

这个就是控制小灯的任务，修改`OSTimeDly`中的值即可改变小灯亮暗间隔。

![](https://tappat-1300227703.cos.ap-guangzhou.myqcloud.com/picture/ucos/2440_led.jpg)

---

### 2.5 时间显示

`main.c`里的Task2就是时间显示功能，不过默认代码里把这个任务关了，找到

~~~c
#if defined(LCD_N35) || defined(LCD_T35) || defined(LCD_X35)    
~~~

这段代码，下面有句`#if 0`改为`#if 1`就可以开启。

> [2019.12.04]你把下面这段复制到报告里

~~~c
void Task2(void *pdata)
{
    unsigned int i, x, m, n, k, y;
    int tmp,key;         

    int width = 10;
    int height = 100;
        
    OSPrintf("Task LCD Running...\r\n");


    //===========================
    // RTC初始化
    //===========================
    Rtc_Init();
    
	//LCD 初始化
	Lcd_N35_Init();

    while(1)
    {
    
    	i++;
    	if(i>99)i=0;

		if(rBCDYEAR==0x99)
			rYear = 1999;
		else
			rYear    = (2000 + rBCDYEAR);
			rMonth   = FROM_BCD(rBCDMON & 0x1f);
			rDay		= FROM_BCD(rBCDDAY & 0x03f);
			rDayOfWeek = rBCDDATE-1;
			rHour    = FROM_BCD(rBCDHOUR & 0x3f);
			rMinute     = FROM_BCD(rBCDMIN & 0x7f);
			rSecond     = FROM_BCD(rBCDSEC & 0x7f);
        
#if defined(LCD_N35) || defined(LCD_T35) || defined(LCD_X35)        
       OSTimeDly( 5 );
       OSPrintf("Task LCD.\n");
       #if 1
       //在LCD上打印时间
       Lcd_printf(100,0,RGB( 0xFF,0xFF,0xFF),RGB( 0x00,0x00,0x00),0,"%02d:%02d:%02d\n", rHour, rMinute, rSecond);
       //在LCD上打印日期，星期
       Lcd_printf(0,15,RGB( 0xFF,0xFF,0xFF),RGB( 0x00,0x00,0x00),0,"%4d-%02d-%02d 星期%d      中国移动\n",
        	      rYear, rMonth, rDay,rDayOfWeek);
       
       Lcd_printf(0,230,RGB( 0xFF,0xFF,0xFF),RGB( 0x00,0x00,0x00),0," 友善之臂uC/OS2任务演示中:%02d" , i);
       #endif
 	   OSTimeDly(OS_TICKS_PER_SEC/5);
	 //  mydelay(100);
    }
}
~~~



## 3 内存管理

以下内容属于理论猜测，板子还给老师了，没实际测试过



### 3.1 代码分析

打开`\uCos2\uCos_Ⅱ\SOURCE\os_mem.c`文件，找到`OSMemCreate`函数代码

~~~c
OS_MEM  *OSMemCreate (void *addr, INT32U nblks, INT32U blksize, INT8U *err)
~~~

可以看到，函数包含4个参数，其中`INT32U blksize`是内存块大小。

### 3.2 修改内存块大小

整个工程中仅有一处涉及到内存块创建，打开`\uCos2\Printf\Printf.c`，其中有如下代码段

```c
void OSPrintfInit(void)
{   
	INT8U err;
	pUartMem=OSMemCreate(UartBuf,UartMsgNum,UartMsgBufLengh,&err);//Create a mem for Uart post Msg
	
	pUart_Q=OSQCreate(&MsgUart[0],UartMsgNum);//Create a Quen for Uart
	
	OSTaskCreate(TaskUart,(void*)0,&TaskUartStk[TaskUartStkLengh-1],TaskUartPrio);//Creat a Task to sent Msg to Uart
}
```

其中对`UartMsgBufLengh`的定义为

~~~c++
#define UartMsgBufLengh 100
~~~

即内存块大小为4*100=400字节，若要修改内存块大小，修改该值即可。

## 4 按键控制

### 4.1 资料分析

在用户手册中，我们可以找到如下说明：

![](https://tappat-1300227703.cos.ap-guangzhou.myqcloud.com/picture/ucos/mini2440_key.png)

打开`\uCos2\S3C2440\includes\2440addr.h`，里面有如下定义：

~~~c
#define rGPGCON    (*(volatile unsigned *)0x56000060)	//Port G control
#define rGPGDAT    (*(volatile unsigned *)0x56000064)	//Port G data
#define rGPGUP     (*(volatile unsigned *)0x56000068)	//Pull-up control G
~~~

---

### 4.2 函数构造

于是根据以上信息，我们可以设计出一个用于检测按键反馈的函数：

~~~c
U8 Key_Scan( void )
{
	Delay( 80 ) ;

	if(      (rGPGDAT&(1<< 0)) == 0 )	
		return 1 ;
	else if( (rGPGDAT&(1<< 3)) == 0 )
		return 2;
	else if( (rGPGDAT&(1<< 5)) == 0 )
		return 3 ;
	else if( (rGPGDAT&(1<< 6)) == 0 )
		return 4 ;
	else if( (rGPGDAT&(1<< 7)) == 0 )
		return 5 ;
	else if( (rGPGDAT&(1<<11)) == 0 )
		return 6 ;
	else
		return 0xff;

}
~~~

> 1X6 矩阵键盘
> 六个输入引脚：	
>
> 				EINT8 -----( GPG0  )
> 				EINT11 -----( GPG3  )
> 				EINT13-----( GPG5  )
> 				EINT14-----( GPG6 )
> 				EINT15-----( GPG7 )
> 				EINT19-----( GPG11 )

---

### 4.3 应用

回到main文件，将我们构造的函数添加进去，接着回到之前的小灯任务（Task1），其中：

~~~c
 if(TestCnt%2)
     	rGPBDAT = 0x0000;
     else
    	rGPBDAT = 0x07fe;
    	
     OSTimeDly(OS_TICKS_PER_SEC);
~~~

把这段代码if语句的条件改为key_scan为参数（可以是检测任意按键或某个特定按键）。

实际运行时，检测到按键按下，小灯发生变化。