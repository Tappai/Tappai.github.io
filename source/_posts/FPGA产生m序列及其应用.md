---
title: FPGA产生m序列及其应用
date: 2019-10-09 16:55:32
tags: [原创,FPGA,密码学]
categories: [工学]
---

伪随机序列又称为伪随机码，是一组人工生成的周期序列。它不仅具有随机序列的一些统计特性和高斯白噪声所有的良好的自相关特征，而且具有某种确定的编码规则，同时又便于重复产生和处理，因而在通信领域应用广泛。

<!--more-->

通常产生的伪随机序列电路可以用线性反馈移位寄存器，其产生的周期最长的二进制数字序列称为最大长度线性反馈移位寄存器序列，简称m序列。

### 伪随机序列发生器

![](https://tappat-1300227703.cos.ap-guangzhou.myqcloud.com/picture/FPGA/m_1.png)

m序列在保密通信中的应用,如下表所示：

![](https://tappat-1300227703.cos.ap-guangzhou.myqcloud.com/picture/FPGA/m_2.png)

> 简单地说，就是将信号源发出的信息经过m序列的**异或**操作处理加密，达到接收方之前，再用**加密时使用的那串m序列**再次异或处理即可得到原本的信息。之所以要强调是加密时的那串序列，是因为m序列发生器产生的m序列是在时刻变化着的。

### FPGA实现

我们要在FPGA中进行的设计与上述原理思路一致。

---

设计一个用于加密的器件，这里命名为EncodeMachine，内部产生m序列，并将其与输入的数据异或后再输出。

代码如下：

~~~verilog
module EncodeMachine(
                         clk, 
                         rst_n,
                         encode_data_in,
					               encode_data_out
                       );
             
                         input       clk;
                         input       rst_n;
                         input[7:0]  encode_data_in;
                         output      encode_data_out;
				                 				                 
				                 reg[7:0]  shift_reg;
				                 wire[7:0] encode_data_out;
       ///////////////////////////////////////////////////////////////////
       
       parameter seed = 8'b10101010;
       
       ///////////////////////////////////////////////////////////////////      
             
                         always@(posedge clk or negedge rst_n)
                         begin
                             if(~rst_n)
                                 shift_reg <= seed;
                             else 
                                 begin
                                   shift_reg[0]<=shift_reg[1];
                                   shift_reg[1]<=shift_reg[2];
                                   shift_reg[2]<=shift_reg[3];
                                   shift_reg[3]<=shift_reg[4];
                                   shift_reg[4]<=shift_reg[5];
                                   shift_reg[5]<=shift_reg[6];
                                   shift_reg[6]<=shift_reg[7];
                                   shift_reg[7]<=shift_reg[0]^shift_reg[4]^shift_reg[5]^shift_reg[6];
                                end
                         end 
				                 
				                 assign encode_data_out = encode_data_in ^ shift_reg;
				                
endmodule
~~~

---

再设计一个用于解密的器件，这里命名为DecodeMachine，**产生与EncodeMachine同步的m序列**，将加密的数据与同步时刻的m序列异或输出，得到原数据。

> 所谓同步，也就是设定同一个seed初值。

代码如下：

~~~verilog
module DecodeMachine(
                         clk, 
                         rst_n,
                         decode_data_in,
					               decode_data_out
                       );
             
                         input       clk;
                         input       rst_n;
                         input[7:0]  decode_data_in;
                         output      decode_data_out;
				                 				                 
				                 reg[7:0] shift_reg;
				                 wire[7:0] decode_data_out;
    					parameter seed = 8'b10101010;

				         always@(posedge clk or negedge rst_n)
                         begin
                             if(~rst_n)
                                 shift_reg <= seed;
                             else 
                                 begin
                                   shift_reg[0]<=shift_reg[1];
                                   shift_reg[1]<=shift_reg[2];
                                   shift_reg[2]<=shift_reg[3];
                                   shift_reg[3]<=shift_reg[4];
                                   shift_reg[4]<=shift_reg[5];
                                   shift_reg[5]<=shift_reg[6];
                                   shift_reg[6]<=shift_reg[7];
                                   shift_reg[7]<=shift_reg[0]^shift_reg[4]^shift_reg[5]^shift_reg[6];
                                end
                         end 

				           
				                 assign  decode_data_out = decode_data_in ^ shift_reg;
	
	endmodule
~~~

---

最后在textbench中设定好测试数据，仿真一下：

![m_4](https://tappat-1300227703.cos.ap-guangzhou.myqcloud.com/picture/FPGA/m_4.png)

这里一共显示了五行二进制数据。

- 第一行数据是输入的原始数据

- 第二行是经m序列加密后的实时数据
- 第三行是经过解密后还原的原始数据
- 第四行是加密器件的内部m序列
- 第五行是解密器件的内部m序列



textbench参考代码：

~~~verilog
`timescale 1ns/1ps             
module testbench;
    
       reg              clk;  
       reg              rst_n;
       reg[7:0]         encode_data_in;
       wire[7:0]        encode_data_out;
       wire[7:0]        decode_data_out;
		 
	     EncodeMachine  EncodeMachine_inst 
              (
                .clk(clk),
                .rst_n(rst_n),
                .encode_data_in(encode_data_in),
                .encode_data_out(encode_data_out)
              );
      DecodeMachine  DecodeMachine_inst 
              (
                .clk(clk),
                .rst_n(rst_n),
                .decode_data_in(encode_data_out),
                .decode_data_out(decode_data_out)
              );      
        

                           
       always #10
           clk   =  ~clk;
           
       initial
	         begin
	                #0    clk     = 1;
	                      rst_n   = 0;
	                      encode_data_in = 8'h00;
								  #40   rst_n   = 1;
								        encode_data_in = 8'h41;
								  #20   encode_data_in = 8'h41;
								  #20   encode_data_in = 8'h41;
								  #20   encode_data_in = 8'h41;
								  #20   encode_data_in = 8'h41;
								  #20   encode_data_in = 8'h41;
								  #20   encode_data_in = 8'h41;
								  #20   encode_data_in = 8'h41;
								  #20   encode_data_in = 8'h41;
								  #20   encode_data_in = 8'h41;
                  #1000 $stop ;      
		       end

endmodule 
~~~

