## EDA实验1实验报告

### 一、实验目的

- 实践基于 FPGA 设计和实现组合逻辑电路的流程和方法； 
- 学习一种硬件描述语言； 
- 熟悉利用 FPGA 平台进行设计验证的方法。 

### 二、实验内容

- 基于实验套件中的 FPGA 开发板，实现简易计算器： 
- 修改设计，以十进制形式显示运算数和运算结果。

### 三、实验设计思路

- 首先依次建立加法器、减法器、乘法器，对A、B进行计算；
- 利用数据选择器，通过K的输入，选择输出数据；
- 对输出数据进行显示。包括16进制与十进制。

### 四、各模块实现方法

- 加法器：（bit8_add.v）

```verilog
module add8(i0,i1,sum,s);
input [3:0] i0,i1;
output [7:0] sum;
output s;

assign sum = i0 + i1;
assign s = 1'b0;
endmodule
```

- 减法器：（bit8_sub.v）

```verilog
module sub8(i0,i1,diff,s);
input [3:0] i0,i1;
output [7:0] diff;
output s;

assign s = i0 > i1;
assign diff =s ? i0-i1 : i1-i0;

endmodule
```

- 乘法器：（bit8_mul.v）

```verilog
module mul8(i0,i1,prod,s);
input [3:0] i0,i1;
output [7:0] prod;
output s;

assign prod = i0 * i1;
assign s = 1'b0;
endmodule
```

- 数据选择器：（mux8four.v 与 muxfour.v）

```verilog
// 四位的数据选择器
module mux8four(i0,i1,i2,i3,sel,out);
input [7:0] i0,i1,i2,i3;
input [1:0] sel;
output [7:0] out;
reg [7:0] out;
always @ (i0 or i1 or i2 or i3 or sel)
begin
	case (sel)
		2'b00: out = i0;
		2'b01: out = i1;
		2'b10: out = i2;
		2'b11: out = i3;
	endcase
end

endmodule

// 一位的数据选择器
module muxfour(i0,i1,i2,i3,sel,out);
input i0,i1,i2,i3;
input [1:0] sel;
output out;
reg out;
always @ (i0 or i1 or i2 or i3 or sel)
begin
	case (sel)
		2'b00: out = i0;
		2'b01: out = i1;
		2'b10: out = i2;
		2'b11: out = i3;
	endcase
end

endmodule
```

- 数据显示：十六进制（show_num.v）

```verilog
module show_num(clk,num,seg,dig);
input clk;
input [7:0] num;
output [6:0] seg;
output [1:0] dig;

wire [3:0] num1, num2;
assign num1 = num [7:4];
assign num2 = num [3:0];

reg [3:0] num_r;
reg [10:0] cur;
reg [1:0] dig_r;
reg [6:0] seg_r;
assign dig = dig_r;
assign seg = ~seg_r;

always @ (cur[10:10])
begin
	case (cur[10:10])
		1'b0: num_r <= num1;
		1'b1: num_r <= num2;
	endcase
end

always @ (cur[10:10])
begin
	case (cur[10:10])
		1'b0: dig_r <= 2'b01;
		1'b1: dig_r <= 2'b10;
	endcase
end

always @ (num_r)
begin
	case (num_r)
		4'b0000: seg_r <= 7'b0111111; 	//0
		4'b0001: seg_r <= 7'b0000110; 	//1
		4'b0010: seg_r <= 7'b1011011; 	//2
		4'b0011: seg_r <= 7'b1001111; 	//3
		4'b0100: seg_r <= 7'b1100110; 	//4
		4'b0101: seg_r <= 7'b1101101; 	//5
		4'b0110: seg_r <= 7'b1111101; 	//6
		4'b0111: seg_r <= 7'b0100111; 	//7
		4'b1000: seg_r <= 7'b1111111; 	//8
		4'b1001: seg_r <= 7'b1101111; 	//9
		4'b1010: seg_r <= 7'b1110111; 	//A
		4'b1011: seg_r <= 7'b1111100; 	//b
		4'b1100: seg_r <= 7'b0111001; 	//C
		4'b1101: seg_r <= 7'b1011110; 	//d
		4'b1110: seg_r <= 7'b1111001; 	//E
		4'b1111: seg_r <= 7'b1110001; 	//F		
	endcase
end

always @ (posedge clk)
begin
	cur <= cur+1'b1;
end

endmodule
```

- 数据显示：十进制（show_num10.v）

```verilog
module show_num10(clk,num,seg,dig);
input clk;
input [7:0] num;
output [6:0] seg;
output [2:0] dig;

reg [3:0] num_r1,num_r2,num_r3;

reg [3:0] num_r;
reg [11:0] cur;
reg [2:0] dig_r;
reg [6:0] seg_r;

assign dig = dig_r;
assign seg = ~seg_r;

always @ (num)
begin
	num_r1 = num % 10;
	num_r2 = (num / 10)% 10;
	num_r3 = num / 100;

end

always @ (num_r1 or num_r2 or num_r3 or cur[11:10])
begin
	case (cur[11:10])
		2'b00: num_r <= num_r3;
		2'b01: num_r <= num_r2;
		2'b10: num_r <= num_r1;
		2'b11: num_r <= 4'b1111;			// 输出15，此时数码管不显示
	endcase
end

always @ (cur[11:10])
begin
	case (cur[11:10])
		2'b00: dig_r <= 3'b011;
		2'b01: dig_r <= 3'b101;
		2'b10: dig_r <= 3'b110;
		2'b11: dig_r <= 3'b111;				// 令数码管不显示
	endcase
end

always @ (num_r)
begin
	case (num_r)
		4'b0000: seg_r <= 7'b0111111; 	//0
		4'b0001: seg_r <= 7'b0000110; 	//1
		4'b0010: seg_r <= 7'b1011011; 	//2
		4'b0011: seg_r <= 7'b1001111; 	//3
		4'b0100: seg_r <= 7'b1100110; 	//4
		4'b0101: seg_r <= 7'b1101101; 	//5
		4'b0110: seg_r <= 7'b1111101; 	//6
		4'b0111: seg_r <= 7'b0100111; 	//7
		4'b1000: seg_r <= 7'b1111111; 	//8
		4'b1001: seg_r <= 7'b1101111; 	//9
		default:	seg_r <= 7'b0000000;		//数码管不显示，减小干扰
	endcase
end

always @ (posedge clk)
begin
	cur <= cur+1'b1;
end

endmodule
```



- 主函数：（final.v）

```verilog
// 十六进制下的主函数
module eda1(clk,a,b,f,seg,dig,s);
input [3:0] a,b;
input [1:0] f;
input clk;
output [6:0] seg;		//控制输出的数
output [1:0] dig;		//两个输出的数码管
output s;

wire [7:0] num;
wire [7:0] add_out, sub_out, mul_out;
wire add_s, sub_s, mul_s;
add8	our_adder(a,b,add_out,add_s);
sub8	our_subtracter(a,b,sub_out,sub_s);
mul8	our_multiplier(a,b,mul_out,mul_s);

mux8four output_num(8'b0,add_out,sub_out,mul_out,f,num); //对数值进行选择 
muxfour output_s(1'b0,add_s,sub_s,mul_s,f,s);				  //对符号进行选择 
show_num our_shownum(clk,num,seg,dig);
endmodule

// 十进制下的主函数
module eda1(clk,a,b,f,seg,dig,s);
input [3:0] a,b;
input [1:0] f;
input clk;
output [6:0] seg;		//控制输出的数
output [2:0] dig;		//3个输出的数码管
output s;

wire [7:0] num;
wire [7:0] add_out, sub_out, mul_out;
wire add_s, sub_s, mul_s;
add8	our_adder(a,b,add_out,add_s);
sub8	our_subtracter(a,b,sub_out,sub_s);
mul8	our_multiplier(a,b,mul_out,mul_s);

mux8four output_num(8'b0,add_out,sub_out,mul_out,f,num); //对数值进行选择 
muxfour output_s(1'b0,add_s,sub_s,mul_s,f,s);				  //对符号进行选择 
show_num10 our_shownum(clk,num,seg,dig);
endmodule
```



### 五、实验测试

说明：SW1-4 代表数字 B，SW5-8 代表数字 A，LED表示正负号（暗为正，亮为负），KEY1、KEY2控制K

（这里理论上有图，但是我当时没存，找不到了）

十六进制：

$0101_{(2)}*1110_{(2)}=5_{(10)}*14_{(10)}=70_{(10)}=46_{(16)}$



$0101_{(2)}+1101_{(2)}=5_{(10)}+13_{(10)}=18_{(10)}=12_{(16)}$



$1001_{(2)}-1011_{(2)}=9_{(10)}-11_{(10)}=-2_{(10)}=-2_{(16)}$



KEY均按下，显示为0





十进制：

$1101_{(2)}*1100_{(2)}=13_{(10)}*12_{(10)}=156_{(10)}$



### 六、实验中遇到的问题与解决方法

​	在十进制输出时，不同数码管之间干扰较大，因此在数码管显示间隙时间段内，控制相应字段的均为暗，且控制所有数码管均不选通。

