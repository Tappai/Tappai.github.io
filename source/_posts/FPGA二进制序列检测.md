---
title: FPGA二进制序列检测
categories: '工学'
date: 2019-10-02 16:39:42
tags: [原创,FPGA]
---

## 目的

设计一个二进制序列检测器，当检测到10110序列时，就输出1（一个时钟周期的脉冲），其它情况下输出0

<!--more-->

## 方法

采用Moore状态机或Mealy状态机进行设计



## Moore状态机

#### 特性：

#### 输出仅与当前状态有关

---

#### 转换原理：

![](https://tappat-1300227703.cos.ap-guangzhou.myqcloud.com/picture/FPGA/prj1_1.png)

简单说明一下，图中的Sx/0，比如说S1/0，意思是切换到S1状态时，输出0；箭头上的数字是指接收到该信号（1或0）时，从箭尾状态变为箭头所指状态。

显然，输入10110序列，状态变化为：S0→S1→S2→S3→S4→S5，并且切换到S5时输出1

---

#### verilog程序实现

~~~verilog
module Moore_State_Machine (
                            clk,
                            reset,
                            data_in,
                            data_out
                          );

              input  clk;
               input  reset;
               input  data_in;
               output data_out;
      
		           //enum STATE_TYPE one-hot
               parameter [5:0]S0  =  6'b000001;
               parameter [5:0]S1  =  6'b000010;
               parameter [5:0]S2  =  6'b000100;
               parameter [5:0]S3  =  6'b001000;
               parameter [5:0]S4  =  6'b010000;
               parameter [5:0]S5  =  6'b100000;
                    
               reg        data_out;     
               reg[5:0]   present_state;
               reg[5:0]   next_state;
               
               // present_state assignment,sequential logic
               always @(posedge clk or posedge reset)
               begin
                     if(reset)
                               present_state  <= S0;
                     else
                               present_state  <= next_state;
               end

               //next_state assignment,combinational logic
               always @(present_state or data_in)
               begin
                     case(present_state)
                          S0: next_state=(data_in==1)?S1:S0;                             
                          S1: next_state=(data_in==0)?S2:S1;
                          S2: next_state=(data_in==1)?S3:S0;                             
                          S3: next_state=(data_in==1)?S4:S2;
                          S4: next_state=(data_in==0)?S5:S1;                             
                          S5: next_state=(data_in==0)?S0:S1;        
                          default: next_state = S0;
                     endcase
               end
               
               //output assignment,combinational logic
               always @(present_state)
               begin
                     case(present_state)              
                            S0: data_out  = 0;
                            S1: data_out  = 0;
                            S2: data_out  = 0;
                            S3: data_out  = 0;
                            S4: data_out  = 0;
                            S5: data_out  = 1;
                       default: data_out = 0;
                    endcase
               end
endmodule
~~~

---

#### 仿真代码

~~~verilog
`timescale 1ns/1ps

module testbench;
    
       reg         data_in;
       reg         reset;
       reg         clk; 
       wire        data_out;
                                  
  Moore_State_Machine  moore_state_machine_inst 
            (
                .clk(clk),
                .reset(reset),
                .data_in(data_in),
                .data_out(data_out)
             );
             
   always #10 clk = ~clk;
           
    initial
	       begin
	                            
	                           #0 clk          = 0;
	                              reset        = 0;
	                        	     data_in       = 0;
	                          #40 reset        = 1;
                            #40 reset        = 0;
		          
		                        #50 data_in  =  0;
		                        #20 data_in  =  1;
		                        #20 data_in  =  1;
		                        #20 data_in  =  0;
		                        #20 data_in  =  1;
		                        #20 data_in  =  1;
		                        #20 data_in  =  0;
		                        #20 data_in  =  1;
		                        #20 data_in  =  1;
		                        #20 data_in  =  0;
		                        #20 data_in  =  1;
		                        #20 data_in  =  1;
		                        #20 data_in  =  0;
		                        #20 data_in  =  0;
		                            
          
		                       #100 $stop ;      
		                    
		               end
		               
endmodule 
~~~



---

#### 仿真测试

![](https://tappat-1300227703.cos.ap-guangzhou.myqcloud.com/picture/FPGA/prj1_2.png)

