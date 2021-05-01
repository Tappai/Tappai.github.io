---
title: FPGA产生三路120度相位差正弦波
categories: '工学'
date: 2019-09-25 12:19:32
tags: [原创,FPGA]
---

## 概述

直接数字频率合成（DDS）的基本原理是利用采样定理，通过查表法产生波形，其基本结构如下图所示，其中fc为数字逻辑电路的时钟频率。

![](https://tappat-1300227703.cos.ap-guangzhou.myqcloud.com/picture/FPGA/prj2_1.png)

<!--more-->

## 量化正弦波

**第一步，我们先需要将正弦波进行量化，才能将数值输送到RAM中保存**

这里给出利用C语言进行量化的程序，修改正弦波表达式即可得到对应正弦波

~~~C
//在正弦波一个周期内取样512点，每点8bit量化，存储在memory[512]中
    #include<math.h>
    #include<stdio.h>
    #define Pi 3.1415926
   
    float SinWave(float x)
	{
	    return ( 0.5 + 0.5*sin(x) ); //正弦波表达式
	}
    
	void main()
    {
       int n,memory[512];
       float w; 
	   FILE *f;
	
	   f =fopen("result.txt","w"); //保存输出内容文档的名称，运行程序后在本程序文件夹生成
       w = 2*Pi/512;
       for(n=0;n<512;n++)
       {
	       memory[n]  =  255 * SinWave(n*w);
	       fprintf(f," memory[%3d] <= 8'h%x;\n",n,memory[n]); //已改为verilog程序中的格式
	       
	   }
       fclose(f);
    }
~~~

**需要提醒的是：**该程序在`devcpp`、`codeblocks`等IDE上都无法正常运行，**建议**使用**VC++6.0**进行编译

> 之后发现可能是由于其需要整个工程文件来运行

**试图用自己实践文件的同学:**特别注意我更改的输出格式，如果用原先的格式输出放到`verilog`程序里是有问题的（具体可对照`verilog`实践的初始程序）。

## RAM

**本次实践中用RAM代替了ROM**

这里给出RAM的框架

~~~verilog
module RAM(clk,rst_n,addr,data_out); //这里的RAM就是器件名
     	   input       clk;
		   input       rst_n;
		   input [8:0] addr;
		   output[7:0] data_out;
		   
		   reg[7:0]    data_out;
		   reg[7:0]  mem[511:0];
	 
	always @(posedge clk or negedge rst_n)
	 if(~rst_n)
		begin
            /*
            把之前输出的一大串mem值放这段
            */
             end
		 
		always @(posedge clk)
		    data_out = mem[addr];	 
endmodule
         
~~~

量化后的正弦波数值就保存在RAM中了。

一个RAM对应一路，如果你需要两路正弦波，就创建另一个`verilog`文件，两个文件分别改第一行的名称为RAM_1和RAM_2，三路同理，以此类推。

## 调用RAM和D/A转换

接下来就是把量化后的正弦波在FPGA中通过D/A转换重新生成为连续波形。

这里给出三路正弦波转换的代码：

~~~verilog
module SinWave(clk,rst_n,FTW,SinWaveData_1,SinWaveData_2,SinWaveData_3);
       
		   input       clk;
		   input       rst_n;
		   input[31:0] FTW;
		   output[7:0] SinWaveData_1;
		   output[7:0] SinWaveData_2;
		   output[7:0] SinWaveData_3;

		   reg[31:0] FACCResult;
		
		   always@(posedge clk or negedge rst_n)
		      if(~rst_n)
		         FACCResult <= 0;
		      else
		         FACCResult <= FACCResult + FTW;
		
		
		   RAM RAM_inst(
		                  .clk(clk),
					  	  .rst_n(rst_n),
						   .addr(FACCResult[31:23]),
						   .data_out(SinWaveData_1)
		                );
			
			RAM_1 RAM_1_inst(
		                  .clk(clk),
					  	  .rst_n(rst_n),
						   .addr(FACCResult[31:23]),
						   .data_out(SinWaveData_2)
		                );		
		    RAM_2 RAM_2_inst(
		                  .clk(clk),
					  	  .rst_n(rst_n),
						   .addr(FACCResult[31:23]),
						   .data_out(SinWaveData_3)
		                );   

endmodule 
~~~

可以看到，我分别使用了三个RAM，因此输出端也要修改为三个，并且连接到对应的器件端口(data_out)。

## 结果测试

修改好`testbench`运行结果如下：

![](https://tappat-1300227703.cos.ap-guangzhou.myqcloud.com/picture/FPGA/prj2_2.png)

三路相差120度的正弦波



这里给出我的`testbench`程序：

~~~verilog
`timescale 1ns/1ps             
module testbench;
    
       reg              clk;  
       reg              rst_n;
       reg[31:0]        FTW;
       wire[7:0]        SinWaveData_1;
       wire[7:0]        SinWaveData_2;
      wire[7:0]        SinWaveData_3;
		 
	     SinWave SinWave_inst 
              (
                .clk(clk),
                .rst_n(rst_n),
                .FTW(FTW),
                .SinWaveData_1(SinWaveData_1),
                .SinWaveData_2(SinWaveData_2),
                .SinWaveData_3(SinWaveData_3)
              );
           

       always #5
           clk   =  ~clk;      //clock = 100MHz
           
       initial
	         begin
	                #0       clk     = 1;
	                         rst_n   = 0;
	                         FTW     = 32'h01000000;
					#40      rst_n   = 1;
					#10000   $stop;
            end

endmodule 
~~~

