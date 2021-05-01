---
title: FPGA测量两路同频信号的相位差
categories: '工学'
date: 2019-09-18 16:35:56
tags: [原创,FPGA]
---
## 概述
**测量两路信号相位差有两种思路:**

+ 第一种是在`verilog`程序中直接让两路信号经过异或门输出，测量该输出信号的占空比，其占空比乘以360度就得到相位差。
<!--more-->
  该方法的详细内容参考该篇文章：[FPGA测两路信号相位差](https://blog.csdn.net/lt66ds/article/details/9749183)

+ 第二种是在拥有测量时间差和测量信号频率的两个功能模块的基础上，利用基本公式得到相位差。

  下面简单介绍第二种方法。

---

#### 频率计

频率计模块的原理如图所示：

![img](https://tappat-1300227703.cos.ap-guangzhou.myqcloud.com/picture/FPGA/0.png)



由此可以得到信号的频率。

参考代码：

~~~verilog
module get_frequence(
                        clk,
                        rst_n,
                        fx,
                        NA,
                        NB
                     );
                     input         clk;
                     input         rst_n;
                     input         fx;
                     output[31:0]  NA;
                     output[31:0]  NB;
                     
                     reg[31:0]     NA;
                     reg[31:0]     NB;
                     
                     reg[31:0]     counter;
                     reg[31:0]     counter_a;
                     reg[31:0]     counter_b;
                     
                     wire          Tp;
                     reg           T;
                                        
                     always @(posedge clk or negedge rst_n)
                     begin
                           if(~rst_n)
                                counter <= 0;
                           else if(counter < 200000000)
                                counter <= counter+1;
                           else if(counter == 200000000)
                                counter <= 0;
                     end
                     
                     assign Tp = ( counter < 100000000)?1:0 ;
                      
                     
                     always @(posedge fx or negedge rst_n)
                          if(~rst_n)
                              T <= 0;
                          else
                              T <= Tp;
                          
                     always @(posedge fx or negedge rst_n)
                     begin
                            if(~rst_n)
                                 counter_a <= 0;
                            else if(~T)
                                  counter_a <= 0;
                            else if(T)
                                  counter_a <= counter_a + 1;
                     end
                     
                     always @(negedge T or negedge rst_n)
                          if(~rst_n)
                               NA <= 0;
                          else
                               NA <= counter_a;
                      
                     always @(posedge clk or negedge rst_n)
                     begin
                            if(~rst_n)
                                  counter_b <= 0;
                            else if(~T)
                                  counter_b <= 0;
                            else if(T)
                                  counter_b <= counter_b + 1;
                     end
                     
                     always @(negedge T or negedge rst_n)
                             if(~rst_n)
                                 NB <= 0;
                             else
                                 NB <= counter_b;

endmodule
~~~



---

#### 时间间隔电路

测量时间差的模块原理如图所示：

![img](https://tappat-1300227703.cos.ap-guangzhou.myqcloud.com/picture/FPGA/1.png)

由此可得到两路信号的时间差。

参考代码：

~~~verilog
module measure_time(
                        clk,
                        rst_n,
                        fA,
                        fB,
                        NA,
                        NB
                     );
                     input         clk;
                     input         rst_n;
                     input         fA;
                     input         fB;
                     output[31:0]  NA;
                     output[31:0]  NB;
                     
                     reg[31:0]     NA;
                     reg[31:0]     NB;
                     
                     reg[31:0]     counter;
                     reg[31:0]     counter_a;
                     reg[31:0]     counter_b;
                     
                     wire          Tp;
                     reg           T;
                     wire          fAA;
                     wire          fBB;
                     reg           clk_NA;
                     
                     //预置闸门时间Tp = 1s                   
                     always @(posedge clk or negedge rst_n)
                     begin
                           if(~rst_n)
                                counter <= 0;
                           else if(counter < 200000000)
                                counter <= counter+1;
                           else if(counter == 200000000)
                                counter <= 0;
                     end
                     
                     assign Tp = ( counter < 100000000)?1:0 ;
                      
                     
                     always @(posedge fA or negedge rst_n)
                          if(~rst_n)
                              T <= 0;
                          else
                              T <= Tp;
                      
                     assign fAA = ~(~fA);
                     assign fBB = ~fB;
                     
                     always @( posedge fAA or negedge fBB )
                        if(~fBB)
                              clk_NA <= 0;
                        else if(fBB)
                              clk_NA <= T;
                        
                           
                     always @(posedge clk_NA or negedge rst_n)
                     begin
                            if(~rst_n)
                                 counter_a <= 0;
                            else if(~T)
                                  counter_a <= 0;
                            else if(T)
                                  counter_a <= counter_a + 1;
                     end
                     
                     always @(negedge T or negedge rst_n)
                          if(~rst_n)
                               NA <= 0;
                          else
                               NA <= counter_a;
                      
                     always @(posedge clk or negedge rst_n)
                     begin
                            if(~rst_n)
                                  counter_b <= 0;
                            else if(~T)
                                  counter_b <= 0;
                            else if(clk_NA)
                                  counter_b <= counter_b + 1;
                     end
                     
                     always @(negedge T or negedge rst_n)
                             if(~rst_n)
                                 NB <= 0;
                             else
                                 NB <= counter_b;

endmodule
~~~

---

#### 在仿真测试中调用

仿真文件中，需要同时调用两个模块，仿真结果如图所示：

![image](https://tappat-1300227703.cos.ap-guangzhou.myqcloud.com/picture/FPGA/3.png)



根据前面的公式可以得到：信号频率为`1MHz`，时间差为`2x10^-7s`

于是，相位差为2Pi/5



`textbench`参考代码：

```verilog
`timescale 1ns/1ns
module testbench;

        reg clk;                          	
        reg fA;
        reg fB;
        reg rst_n;
        wire[31:0] NA;
        wire[31:0] NB;  
        wire[31:0] NC; 
        wire[31:0] ND;                       
        reg       clk_d;        
        reg       f_test;      
        measure_time  mt_inst(
                                .clk(clk),
                                .rst_n(rst_n),
                                .fA(fA),
                                .fB(fB),
                                .NA(NA),
                                .NB(NB)
                            );
        get_frequence  gf_inst(
                                .clk(clk),
                                .rst_n(rst_n),
                                .fx(fA),
                                .NA(NC),
                                .NB(ND)
                            );
        
        always #5           clk = ~clk;   // f0 = 100MHz
      
        always #100         clk_d = ~clk_d;
        
              
        always  
            #500 f_test  =  ~f_test;         // fA = 1MHz
        
        always @(posedge clk_d)
           begin      
                 fA <= f_test;
                 fB <= fA;
           end    
       
       initial
	           begin
	                    
	  	               #0      clk = 0;
	  	                       rst_n = 0;
		                        fA  = 0;
		                        f_test = 0;
		                        clk_d =0;
		                      		                       
		                 #10     rst_n = 1;
		                 	               	 	                 	
	                   #2500000000    $stop;                 
  
	           end

endmodule 
```

