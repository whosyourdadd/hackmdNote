# SystemVerilog
很久之前的讀書筆記，來源有點不可考了@@

## ch1 introduce
驗證複雜的soc是一件花時間的工作，現在有更多的方法論改善收斂發法，從block level,top level,chip level
### asic design flow
1. market survey
2. spec extraction and design planing
3. architecure and microarchieure for design
4. rtl design use systemverilog
5. rtl verification using system verilog
6. logic synthesis
7. design for test
8. prelayout sta
## ch2 SystemVerilog Literal Values and Data Types
學習任何新的語言最重要的是了解結構與資料型態，system verilog支援verilog 與c的型態結合組合與序向數位電路的驗證
### 2.1 predefine gates
* and,nand,or,nor,xor,xnor
### 2.2 structural modeling
利用predefine gates建立結構，本小節用一個2:1 multiplexer舉例
```verilog=
module structural_design(output y_out,int set_in,a_in,b_in);

logic tmp_1,tmp_2,tmp_3;
and u1(tmp_2,sel_in,a_in);
and u2(tmp_3,tmp_1,b_in);
not u3(tmp_1,sel_in);
or u4(y_out,tmp_2,tmp_3);
endmodule
```
### 2.3 system verilog format specifier
https://zhuanlan.zhihu.com/p/38563777

总结Verilog wire和reg的区别：

* wire表示导线结构，reg表示存储结构。
* wire使用assign赋值，reg赋值定义在always、initial、task或function代码块中。
* wire赋值综合成组合逻辑，reg可能综合成时序逻辑，也可能综合成组合逻辑。

net类型数据需要使用assign关键字连续赋值（continuous assignment）。虽然assign语句一般被综合成组合逻辑，但net本质还是导线，真正被综合成组合逻辑的是assign右边的逻辑运算表达式。常见的net类型数据包括wire、tri、wand和supply0等。

variable类型设计用于表示存储结构，它内部存储状态，并在时钟沿到来或异步信号改变等条件触发时改变内部状态。variable类型数据需要使用过程赋值（procedural assignment），即赋值定义在always、initial、task或function语法块中。reg是最典型的variable类型数据，但需要说明的是，综合工具可能将reg优化综合成组合逻辑，并不一定是寄存器。常见的variable类型数据包括reg、integer、time、real、realtime等。


总结SystemVerilog logic的使用方法：

* 单驱动时logic可完全替代reg和wire，除了Evan提到的赋初值问题。
* 多驱动时，如inout类型端口，使用wire。

SystemVerilog在Verilog基础上新增支持logic数据类型，logic是reg类型的改进，它既可被过程赋值也能被连续赋值，编译器可自动推断logic是reg还是wire。唯一的限制是logic只允许一个输入，不能被多重驱动，所以inout类型端口不能定义为logic。不过这个限制也带来了一个好处，由于大部分电路结构本就是单驱动，如果误接了多个驱动，使用logic在编译时会报错，帮助发现bug。所以单驱动时用logic，多驱动时用wire。

在Jason的博客评论中，Evan还提到一点logic和wire的区别。wire定义时赋值是连续赋值，而logic定义时赋值只是赋初值，并且赋初值是不能被综合的。

```verilog=
wire mysignal0 = A & B;     // continuous assignment, AND gate
logic mysignal1 = A & B;    // not synthesizable, initializes mysignal1 to the value of A & B at time 0 and then makes no further changes to it.
logic mysignal2;
assign mysignal2 = A & B;   // Continuous assignment, AND gate
```
参考资料
[1] http://www.verilogpro.com/verilog-reg-verilog-wire-systemverilog-logic

[2] Spear, Chris. SystemVerilog for Verification, Second Edition: A Guide to Learning the Testbench Language Features. Springer, 2008.
### 2.4 multi-bit constants and concatenation
```verilog=
//used to specify the a_in,b_in,c_in as logic net type and 8 bit wide
logic [7:0] a_in,b_in,c_in;//8 bit signed representation with logic net type
logic signed [7:0] d_in;
assign e_in = 4'b1100;//assigning binary value 1100 to net
assign f_in = 4'hE// assigning hex value 1110 to net
assign g_in = 3;//assign decimal value 3 to net 
assign h_in = -2;//assign demial value -2 to net
assign x_in = e_in[2:1];//Part select from the 4-bit binary number that is x_in = 10
assign y_in = {e_in,f_in};
```
### 2.5 literals 文字
systemVerilog supports integer,logic,real,time,string,array literals
#### real literals
```verilog=
real 2.4;
real 2.0e10;
Shortreal `(2.4)//?
```
#### time literal
* fs,ps,ns,us,ms,s
* step //the step definition for the time unit
這裡的時間是掃瞄當前的時間精度
#### string 
#### array 
array 類似c的初始化
```verilog=
int n[1:2][1:3] = {{0,1,2},3,{4}}
typedef int triple [1:3];
$ mydisplay(triple'{0,1,2});
```
#### structure literal
```verilog=
typedef struct {int a;shortreal b;} ab;
ab c;
```
#### data types
##### integer
##### two value or four value data type
* shortint (two state 16-bit signed int)
* int (two state 32-bit signed int)
* long int. (two state 64-bit signed int)
* byte (two state 8 bit signed int or ascii)
* logic (four state user define vector size)
* reg (four state user define vector size)
* integer (four state 32 bit signed int)
* time (four state 64-bit unsigned int)
four state: '0','1','X','Z'
##### chandle data
for DPI(direct programming interface )
```verilog=
chandle var;
```
看不懂要幹嘛的
###### string
```verilog=
string myClass = "class of the bytes";
```
###### event
```verilog=
event ready;//used to declare the event
>event ready_sig = ready;//declares the ready sig alias event
```
###### user define type
```verilog=
typedef int intP;
intP a,b;
typedef fun;
fun f1 = 1;
typedef int fun;
```

## ch3 Hardware Description Using SystemVerilog
### 3.1 how can we start?
就講說HDL有可以合成跟不能合成的並且systemVerilog可以像Ｃ一樣物件導向
#### number and const
#### operator
大部分跟Ｃ一樣
##### reduction operators
```verilog=
&a_in ;// four-bit equ a_in[3]&a_in[2]&a_in[1]&a_in[0]
|a_in ;// four-bit equ a_in[3]|a_in[2]|a_in[1]|a_in[0]
^a_in ;// four-bit equ a_in[3]^a_in[2]^a_in[1]^a_in[0]

```

### 3.2 net type
net type 有wire與reg，組合邏輯會用assign wire,如果用序向邏輯則會用reg
```verilog=
wire [3:0] data bus;
reg [3:0] data bus;
reg [3:0] memory[0:15];//16 element memory each have 4 bits;
logic [3:0] data bus;// in systemVerilog logic instead of reg and wire use inside always_ff,always_comb determines it is reg or wire
```
新增關鍵字always_ff 用於序向控制(ff mean flip-flop)
```verilog=
module seq_design(input wire a_in,b_in,clk,output logic y1_out,y2_out);
assign y1_out = a_in ^ b_in;
always_Ff @(posedge clk,negedge reset_n)
begin
    if(-reset_n)
        y2_out <= '0';
    else
        y2_out <= a_in & b_in;
    end
endmodule
```
### 3.3 let us think about combinational elements
組合邏輯不論是nonblocking or blocking執行都必須是相同的結果(concurrency)
```verilog=
module Comb_design(input wire a_in,b_in,output wire y1_out,y2_out);
    assign #2ns y1_out =   a_in ^ b_in;
    assign #3ns y2_out = ~(a_in ^ b_in);
```
以下是組合邏輯的test bench
```verilog=
module test_comb_design();

reg a_in;
reg b_in;
wire y1_out;
wire y2_out;
 
 comb_design DUT (.a_in(a_in), .b_in(b_in), .y1_out(y1_out), .y2_out(y2_out));
 //DUT = device under test
 always 
 #100 a_in = ~a_in;
 always
 #200 b_in = ~b_in;
 
 initial
 begin
 a_in = '0';
 b_in = '0';
 end
 endmodule
```
下面是另一個使用reduction XOR來確認奇偶位元的功能 
```verilog=
module parity_checker(input wire[15:0] data_in,output logic parity_out);
assign parity_out = ^data_in;
endmodule: parity_checker
```
#### Procedural block: always_comb
原本在verilog裡面使用的是always@(* )，但是這個用法很敏感，對於所有的輸入與暫存變數，所以systemVerilog建立了新的關鍵字always_comb讓模擬跟所有暫存變數都會被調用
```verilog=
module comb_design(input a_in,b_in,output reg[7:0] y_out);

always_comb
begin
y_out[0] = ~(a_in);
y_out[1] =  (a_in | b_in);
y_out[2] = ~(a_in | b_in);
y_out[3] =  (a_in & b_in);
y_out[4] = ~(a_in & b_in);
y_out[5] =  (a_in ^ b_in);
y_out[6] = ~(a_in ^ b_in);
y_out[7] =   a_in;
end
endmodule
```
### 3.4 use always_comb to implment code convert
*格雷碼（Gray code）是由貝爾實驗室的Frank Gray在1940年提出，用於在PCM（脈衝編碼調變）方法傳送訊號時防止出錯，並於1953年三月十七日取得美國專利。格雷碼是一個數列集合，相鄰兩數間只有一個位元改變，為無權數碼，且格雷碼的順序不是唯一的*

本節使用4bit binary to 4 bit gray
```verilog=
module binary_to_gray(input [3:0] binary_data,output reg[3:0] gray_data);
always_comb
begin 
gray_data[3] = binary_data[3];
gray_data[2] = binary_data[3] ^ binary_data[2];
gray_data[1] = binary_data[2] ^ binary_data[1];
gray_data[0] = binary_data[1] ^ binary_data[0];
end
endmodule
```

### 3.5 Basic idea of concurrency
```verilog=
module comparator_16_bit(input logic[15:0] a_in,b_in,output bit g_t_out,e_t_out,l_t_out);
//each out is single bit
//g_t_out is high when a_in is greater than b_in
//e_t_out is high when a_in is equal to b_in
//l_t_out is high when a_in is less b_in
always_comb
begin: a_ingreater_b_in
if(a_in > b_in)
```

### 3.6 Procedural Block always_latch
always_latch,always_ff 用在序向設計，這裡舉了一個栓鎖作為範例
```verilog=
module latch_8bit(input latch_en,input [7:0] data_in,output reg [7:0],output reg [7:0] data_out);

always_latch
begin
    if(latch_en)
        data_out <= data_in
end
endmodule
```
### 3.7 Procedural Block always_ff
```verilog=
module register_8bit(input clk,input reset_n,input [7:0] data_in,output reg[7:0] data_out);
always_ff @(posedge clk or negedge reset_n)
begin
    if(~reset_n)
        data_out <= 8`d0;
    else
        data_out <= data_in;
    end
endmodule
```

### 3.8 Use always_ff implement seq design
這裡介紹了一個四個指令的alu,要用always_ff建立指令選擇的mux
```verilog=
module alu(input clk,input reset_n,input a_in,b_in,input [1:0] op_code,output logic y_out)

always_ff @(posedge clk or negedge reset_n)
if(~reset_n)
    y_out <= 1`d0;
else
    case(opc_code)
        2`b00: y_out <= a_in | b_in;
        2`b01: y_out <= a_in ^ b_in;
        2`b10: y_out <= a_in & b_in;
    default:
        y_out <= ~a_in;
    endcase
endmodule
```

### 3.9 instantiation using named port conections(verilog style)
name port指的是用宣告一個模組的方式做連結避免上百個模組要用會花很多時間 top module 就像宣告一個function一樣(但是verilog module又跟c不一樣)
```verilog=
module hierarchical_design(input wire a_in,b_in,c_in,output logic sum_out,carry_out);

wire s0_out;
wire c0_out;
wire c1_out;

half_addr U1(.a_in(a_in),.b_in(b_in),.sum_out(s0_out),carry_out(c0_out));
half_addr U2(.a_in(a_in),.b_in(b_in),.sum_out(s0_out),carry_out(c0_out));

or_gate U3(.a_in(c0_out),.b_in(c1_out),.y_out(carry_out));

endmodule;

module half_addr(input wire a_in,b_in,output logic sum_out,carry_out)

    assign sum_out = a_in^b_in;
    assign carry_out = a_in & b_in;

endmodule:half_addr
module or_gate(input wire a_in,b_in,output logic y_out);
    assign y_out = a_in | b_in;
endmodule: or_gate
```

### 3.10 instantition using mixed port connectivity
systemverilog支援mix port it mean using(.*),ch11會深入討論
#### 3.11 結論
1.  assign 在組合邏輯當中用於連續輸出
2.  always_comb 用於組合邏輯中使用
3.  always_latch 用於latch中使用
4.  always_ff 用於暫存器或是序向邏輯
5.  多個連續的assign statment將會平行的執行(?)
6.  systemVerilog支援 .named .name implicit 與mix port連結方式



## ch4 systemVerilog and oops support
systemVerilog支援class,structues,unions 

### 4.1 enumerated data type

```verilog=
enum {red,yellow,green} state;
enum {idle, xx= `x,s1 = 2`b01 , s2=2`b02}
```

```verilog=
typedef enum{red,green,blue,yellow} Colors;
Colors c = c.first;//first will
forever
begin
$display("%s: %d",c.name,c)
if(c==c.last)
    break;
c = c.next;//next enum
end

```

#### 4.1.3 Enumerate types 
```verilog=
typedef enum {red,green,blue,yellow,white,black} Colors;
Colors col;
inter a,b;
a = blue * 3;
col = yellow;
b = col + gree

```

#### 4.1.4 Automatic Cast to Base Type

```verilog=
typedef enum{Red,Green,Blue} Color;
typedef enum{Mon,Tue,Wed,Thu,Fri,Sat,Sun}Week;
Colors C;
int i;
C = Colors'(C+1);
C = C + 1;C++;C += 2;C = 1;
C = Colors'(Su);
l = C + W;
```

#### 4.2 Structures;

```verilog 
typedef struct{
    int a_in,b_in;
    bit [15:0] address_in;
    byte op_code;
}processor_data;

process_data instance_s;
always_ff @(posedge clk,negedge reset_n)
if(~reset_n)
begin
    instance_s.a_in = 64
    instance_s_b_in = 32;
    instance_s.address_in = 0;
    instance_s.op_code = 8'h00;
    
    instance_s = {64,32,0,8'h00};//by
    instance_s = {op_code:8'h00,a_in:64,b_in:32,address_n:0};//By using the name
    
end
else
begin
...
end
end

```

##### 4.2.1 Unpacked and Packed structure
```verilog=
struct packed signed{
    int a;
    shortint b;
    byte c;
    byte[7:0] d;
}pack1;
struct packed unsigned{
    time a;
    integer b;
    logic [31:0] c;
}pack2;
```

#### 4.3 unions
跟ｃ一樣相同的結構可以用union
```verilog=
union{
    int a_in;
    int b_in;
    int unsigned c_in;
    }data_u
    
typedef union{
    int a_in;
    int b_in;
    int unsigned c_in;
}
```
#### 4.4 arrays

##### 4.4.1 static array
```verilog=
module test_bench;
bit[7:0]mem_data;
initial 
begin
mem_data = 8'hFF;//1. let us assign a value to the vector
for(int i = 0 ; i < $size(mem_data); i++) begin
    $display("mem_data[%0d]= %b",i,mem_data[i]);
end
end
endmodule
bit[3:0][7:0]mem_data;//Packed
bit[7:0]mem_mem[10:0];//Unpack



always_ff @(posedge clk,negedge reset_n)
if(~reset_n)
    begin
    array_a = {default:0};
    array_a[0] = {default:5};
    end
else
begin
...
end
    
```

##### 4.4.2 single dim packed array
```verilog=
module test_bench;
bit [7:0]mem_data;//one-dimensional packed array
initial begin
mem_data = 8'hff;
```




## ch5 import systemVerilog enhancements
本章節會從一些情境去把下列關鍵字用法帶入
```verilog=
loops functuins task labels and enhances
```
### 5.1 Verilog procedural block
```verilog=
//combinational design modeling
always @(a_in,b_in)//it will sensitivity
//意思就是每個信號都會跑進來
begin
end
//combinational design
//用 always @* 意思就是所有的信號都會動作
always @*
begin

end
//squential
always @(posedge clk or negedge reset_n)
begin

end
```

### 5.3 block label
改善可讀性 sv有label加強
```verilog=
always_comb: 
begin: a_in_greater_b_in
    if(a_in > b_in)
        g_t_out = 'b1;
    else
        g_t_out = 'b0;
end:a_in_greater_b_in
```
### 5.4 statement label
```verilog=
always_comb
begin   
    if(a_in > b_in)
        greater: g_t_out = b'1;
    else
        not_greater: g_t_out = b'0;
    end: a_in_greater_b_in
end
```
### 5.5 module label
每個module都有獨立的名字 endmodule後面可以接一個label來增加可讀性
```verilog=
module comparator_16_bit(input logic[15:0]a_in,b_in,output bit g_t_out,e_t_out,l_t_out);
always_comb
begin: a_in_greater_b_in
    if(a_in>b_in)
        g_t_out = 'b1;
    else
        g_t_out = 'b0;
    end: a_in_greater_b_in
assign e_t_out = (a_in == b_in);
assign l_t_out = (a_in < b_in);

endmodule : comparator_16_bit

```
### 5.6 task and function enhancements
* 相同點
    1. 均放在Module中，將重複的code寫成函數供引用,提升程式設計的效率。
    2. 均不能使用wire變數。
    3. 均用於Behavior Model，本身內容不能有always model。(一般會寫在always model中)

* 相異處
    * Function
        1. 可以引用其他的Function，但是不能引用task。
        2. 至少要有一個以上的input宣告，以及只能有一個output。
        3. 不一定要在程式區塊(Procedural block)中。
        4. 一定要在等號右邊。
    * Task
        1. 可以引用其他的function或task。
        2. 不一定要有input、output or inout宣告。
        3. 一定要在程式區塊(Procedural block)中。
```verilog=
function and_logic(input logic a_in,b_in);
    and_logic = a_in & b_in;
endfunction
```

### 5.7 void function 
主要目的是克服function不能call task的限制
```verilog=
function void half_subtractor(input logic a_in,b_in,output diff_out,borrow_out);
    diff_out = a_in ^ b_in;
    borrow_out = a_in & b_in;
endfunction: half_substractor

task alu(input logic [7:0] a_in,b_in,input logic [1:0] op_code,output logic [7:0] alu_out);
    case (op_code)
        2'b00 : alu_out = a_in + b_in;
        2'b01 : alu_out = a_in - b_in;
        2'b10 : alu_out = a_in ^ b_in;
        2'b11 : alu_out = a_in & b_in;
    endcase
endtask: alu
```
### 5.8 Loops
#### 5.8.1 verilog for loop
```verilog=
module name_of_module(input,output) 
integer i;
always @(posedge clk) begin
    for(i = 0 ; i>= 1023 ; i++)begin
    //loop body
    end
end
endmodule: name_of_module
```
#### 5.8.2~5.8.3 systemVerilog for loop
```verilog=
module name_of_module(input,output)
always_ff @(posedge clk)
begin
for(int i = 0,j = 1000;i*j < 100; i++,j--) begin
    //systemVerilog 有自動儲存產生變數的功能
    //loop可以被合成
    //for loop body
    end
end
endmodule
```
#### 5.8.4 while_loop
```verilog=
module verilog_whileLoop_example ()
always_comb begin
    if(cond)begin
        //assignments
        end
        else while(control_cond_for_while_loop)
            begin
            //assignment
            end
        end
end
endmodule
/*
systemVerilog do while
*/
module SystemVerilog_whileloop()
always_comb
    do begin
    if(condition)begin
        end
    else if(condition)begin
    end
    while(control_cond_for_while_loop)
    //
    end  
endmodule
```
### 5.9~5.10
小節
1. i++可以被合成 但如果是temp =i++則無法
2. always 相較於systemverilog的缺點是不能看關鍵字性質就大概知道這個區塊性質，並且會造成一些simulation或合成 overhead

## ch6 combination design 
* key word: mux,demux,decode,encode,priority encoder,unique,always @*,always_comb_,procedure block,testbench,synthesis,delays
```verilog=
module processor_Design #(parameter data_bus = 8,parameter address_bus = 16) (input[data_width-1:0] data_in,input [select_width-1:0] sel_in,output logic y_out)
always_comb begin
    case (sel_in)
        2'd0: y_out = data_in[0];
        2'd1: y_out = data_in[1];
        2'd2: y_out = data_in[2];
        default: y_out = data_in[3];


module parameter_design_mux4to1 #(parameter data_width = 4,parameter selsct_width =2)(input[data_width-1:0] data_in,input [select_width-1:0] sel_in,output logic y_out);
logic [1:0] tmp_wire;
always_comb begin
    tmp_wire[0] = (sel_in[0] )? data_in[1] : data_in[0];
    tmp_wire[1] = (sel_in[0] )? data_in[3] : data_in[2];
    y_out = (sel_in[1]) ? tmp_wire[1] : tmp_wire[0];
end

module decoder_2to4(input [1:0] sel_in,input enable_in,output logic [3:0] y_out);
always_comb
begin: deboder_functionality
case ({enable_in_sel,sel_in})
    4:y_out = 1;
    5:y_out = 2;
    6:y_out = 4;
    7:y_out = 8;
    default: y_out = 0;
end
endmodule

module test_decoder();
logic [1:0] sel;
wire [3:0] y_out;
decoder_2to4 DUT(.sel_in(sel_in),.enable_in(enable_in),.y_out(y_out));
always #25 sel_in[0] = ~sel_in[0];
always #50 sel_in[1] = ~sel_in[1];
always #100 enable_in = ~enable_in;
initial begin
    sel_in = '0;
    enable_in = 1'b0;
    #100 $monitor("time = %3d,enable_in = %d,sel_in = %d,y_out = %d",$time,enable_in,sel_in,y_out);
end
endmodule

```
## ch7 sequential design
* summary
    1. 序向邏輯輸出為目前的輸入或是先前的最後輸出
    2. latch都是level sensitive,and they are transparent during active level.
    3. flip-flop are edge triggered, and thay are used to sample the data on active edge of the clock.
    4. using always_latch latch will be inferred.
    5. using always_ff register will be inferred.
    6. async reset ,ouput initial irrespective of active edge of clock, and it uses lesser area.
    7. in the sync reset,ouput initial on the active edge of the clock, and it uses more area due to the additional logic in the data and reset path.
```verilog=
module latch_8bit(input latch_en,input[7:0] data_in,output reg [7:0]data_out);
always_latch begin
if(latch_en)
    data_out <= data_in;
end
endmodule
//asynchronous reset
module register_8bit(input clk,input reset_n,input[7:0] data_in,output reg[7:0] data_out);
always_ff @(posedge clk or negedge reset_n)begin
if(~reset_n)
    data_out <= 8'd0;
else 
    data_out <= data_in;
end
endmodule
//synchronous reset
// reset  clk                     data_out
//     0  rising edge             '0'
//     1  level or inactive edge  previous output
//.    1  rising edge             data_in
/*
(1) 同步复位会综合成更小的FF，特别当reset生成逻辑电路作为FF的输入，这样组合逻辑电路的数量变多，所以总的门电路节省不是那么有意义。
(2) 同步复位更加容易工作当使用基于周期的仿真器。
(3) 同步复位确保电路100%是同步的。
(4) 同步复位确保复位只发生在时钟有效边沿，对小的复位毛刺来说，时钟就像滤波器。
(5) 在一些设计中，复位必须由内部条件产生，同步复位能滤除毛刺，所以被采用。
(6) 使用同步复位，FF能够在复位buffer tree中帮助buffer tree的时序保持在一个周期内。
*/
module register_8bit(input clk,input reset_n,input[7:0] data_in,output reg[7:0] data_out);
always_ff @(posedge clk)begin
if(~reset_n)
    data_out <= 8'd0;
else    
    data_out <= data_in;
end
endmodule

```
https://www.cnblogs.com/dangxia/archive/2012/05/21/2512171.html

```verilog=
module up_down_counter(input logic clk,reset_n,up_down,output logic [3:0] q_out);
always_ff @(posedge clk or negedge reset_n) begin
    if(~reset_n)
        q_out <= '0;
    else if(up_down)
        q_out <= q_out+1;
    else
        q_out <= q_out-1;
    end
end
endmodule

module shift_register(input logic clk,reset,load_shift,right_left,input logic [3:0] data_in,output logic [3:0] q_out);
always_ff @(posedge clk or negedge reset_n)begin
if(~reset_n)
    q_out <= '0;
else if(load_shift)
    q_out <= data_in;
else if(right_left)
    q_out <= (data_in>>1);
else
    q_out <= (data_in<<1);
end
endmodule

module ring_counter(input logic clk,reset_n,load_in,input logic [3:0] data_in,ouput logic [3:0] q_out)
always_ff @(posedge clk or negedge reset_n)begin
if(~reset_n)
    q_out <= '0;
else if(load_in)
    q_out <= data_in;
else 
q_out <= {q_out[0],out[3:1]};
end
endmodule


module Johnson_counter(input logic clk,reset_n,load_in,input logic[3:0] data_in,output logic [3:0] q_out);
always_ff @(posedge clk or negedge reset_n)begin
if(~reset_n)
    q_out <= '0;
else
    q_out <= data_in;
else
q_out <= {~q_out[0],q_out[3:1]};
end
endmodule


module arithmethic_unit(
    input clk,
    input [1:0] op_code,
    input [3:0] a_in,b_in,
    output reg[3:0] result_out,
    output reg  carry_out);
    always_ff @(posedge clk) begin
    case(op_code)
        2'd0:{carry_out,result_out} <= a_in+b_in;
        2'd0:{carry_out,result_out} <= a_in-b_in;
        2'd2:{carry_out,result_out} <= a_in+1'b1;
        2'd3:{carry_out,result_out} <= a_in-1'b1;
    endcase
endmodule

module tb()
reg clk;
reg [1:0] op_code;
reg [3:0] a_in,b_in;
wire[3:0] result_out;
wire carry_out;
arithmetic_unit DUT(
    .clk(clk),
    .op_code(op_code),
    .a_in(a_in),
    .b_in(b_in),
    .result_out(result_out),
    .carry_out(carry_out)
    );
    always #10 clk<= !clk;
    initial #0
        begin
            clk = 0;
            op_code = 2'd0;
            a_in = 4'd0;
            b_in = 4'd0;
            #10
            op_code = 2'd1;
            a_in = 4'd5;
            b_in = 4'd5;
            #40
            op_code = 2'd0;
            a_in = 4'd5;
            b_in = 4'd5;
            #20
            op_code = 2'd2;
            a_in = 4'd5;
            b_in = 4'd5;
            #40
            op_code = 2'd3;
            a_in = 4'd5;
            b_in = 4'd5;
        #200 $finish;
        end
        always @(negedge clk)begin
            $monitor(" time= %3d, op_code = %d,a_in=%d,b_in=%d,result_out=%d,carry_out=%d",$time,op_code,a_in,b_in,result_out,carry_out);
            end
endmodule
```

## ch8 RTL Design and synthesis guidelines
rtl設計原則需要定義變數名稱
|Net or interface     |Naming style               |
|---------------------|---------------------------|
|system clock         |sys_clk                    |
|master clock         |master_clk                 |
|slave clock          |slave_clk                  |
|module clock         |clk                        |
|data inputs          |data_in,enable_in,load_in  |
|asynchronous LowReset|reset_n                    |
|synchronous LowReset |reset_n_sync               |
|master reset         |reset_n_master             |
|outputs              |data_out                   |
|state definition     |preset_state,next_state.   |
### 8.2 non-full case statement
如果沒有把所有的case寫出來會產生系統不知道某個沒寫到的case會使用上次的最中結果
```verilog=
module non_full_case(input logic [1:0] sel_in,input a_in,b_in,c_in,output logic y_out);
always_comb begin:decode_block
    case(sel_in)
        2'b00:y_out = a_in;
        2'b01:y_out = b_in;
        2'b10:y_out = c_in;
    endcase
end
endmodule

module full_Case(input logic [1:0] sel_in,input a_in,b_in,c_in,d_in,output logic y_out);
always_comb begin:mux_logic
case (sel_in)
    2'b00: y_out = a_in;
    2'b01: y_out = b_in;
    2'b10: y_ouy = c_in;
    2'b11: y_out = d_in;
endcase
end:mux_logic
endmodule
//思源科技的full case會自動把沒有導向的線路取消掉(變1v3 mux)避免沒有寫到的case
module full_case(input logic[1:0] sel_in,input a_in,b_in,c_in,output logic y_out);
always_comb begin:mux_logic
    case (sel_in)
        2'b00:y_out = a_in;
        2'b01:y_out = b_in;
        2'b10:y_out = c_in;
    endcase
end:mux_logic
endmodule
//一個case對到超過一個條件時，用unique 可以產生錯誤回報
module unique_case(input logic [1:0] sel_in,input a_in,b_in,c_in,d_in,output logic y_out);
always_comb begin    
    unique case(sel_in)
        2'b10: y_out = c_in;
        2'b00: y_out = a_in;
        2'b01: y_out = b_in;
        2'b10: y_out = d_in;
    endcase
end
endmodule

```
#### 8.6 casez statement
用casez可以讓沒有定義的case對don't care但可能會出現潛在問題，須小心使用
#### 8.7 priority case
priority case 可以跟casez、casex共同使用，主要可以用在多個輸入case當中
```verilog=
module dec_2to4(input logic [1:0] sel_in,input enable_in,output logic [3:0] y_out);
always_comb begin:priority_logic
    y_out = '0;
    priority case({enable_in,sel_in})
        3'b1_00:y_out[sel_in] = 1'b1;
        3'b1_01:y_out[sel_in] = 1'b1;
        3'b1_10:y_out[sel_in] = 1'b1;
        3'b1_11:y_out[sel_in] = 1'b1;
    endcase
end:priority_logic
endmodule
```
prioirty if-else不能跟 case condition一起用

8.8~8.11感覺沒很常用 先不看

## ch9 RTL Design and strateges for complex
### 9.1 strategies for complex designs
設計結構扮演重要的角色在複雜設計當中，考慮影片解碼標準h.264 encoder的功能block
* predicition
* transform
* encoder
### 9.2 ALU
|InputAndOutput  |Width  |Direction |Description 
|----------------|-------|----------|---------
|clk             |1-bit  |Input     |
|op_code         |3-bit  |Input     |
|a_in            |4-bit  |Input     |
|b_in            |4-bit  |Input     |
|result_out      |4-bit  |Output    |
|carry_out       |1-bit  |Output    |
```verilog=
module arithmetic_logic_unit(
input clk,
input [2:0] op_code,
input [3:0] a_in,b_in,
output reg [3:0] result_out,
output reg carry_out);

always_ff @(posedge clk) begin: clocked_ALU_Logic
decode: case(op_code)
    3'd0:{carry_out,result_out} <= a_in + b_in;
    3'd1:{carry_out,result_out} <= a_in - b_in;
    3'd2:{carry_out,result_out} <= a_in + 1'b1;
    3'd3:{carry_out,result_out} <= a_in - 1'b1;
    3'd4:{carry_out,result_out} <={0,a_in | b_in;}
    3'd5:{carry_out,result_out} <={0,a_in ^ b_in;}
    3'd6:{carry_out,result_out} <={0,a_in & b_in;}
    3'd7:{carry_out,result_out} <={0, ~a_in;}
endcase
end:clocked_ALU_Logic
endmodule:arithmetic_logic_unit
```
### 9.3 Barrel shifter
```verilog=

```
### 9.4 signle and dual port 
```verilog=
module Async_memory_ram #(parameter width = 16,parameter logsize = 8)(input clk,wr_en,
input [logsize-1:0] rd_addr,
input [logsize-1:0] wr_addr,
input [width-1:0] data_in,
ouput [width-1:0] data_out);

localparam size = 2**logsize;
logic[width-1:0] memory[size-1:0];
assign data_out = memory[rd_addr];//used for the read of the data

always_ff @(posedge clk)begin: memory_write
if(wr_en)
    memory[wr_addr] <= data_in;
end:memory_write
endmodule
```
### 9.5 Bus arbiters and design strategy
### 9.6 mulitple clock domains
### 9.7 FIFO design and strategies
### 9.8 summary
* 複雜系統建議切成各個子模組有利於合成
* 每個功能跟介面需要知道他的用途
* 使用Pipeline logic有利於改善速度
* enable tool options 對於區域最佳化與register平衡
## ch10 finite state machine
### 10.14 summary
* fsm 被用在隨機序向電路行為
* fsm 有兩種 moore and meanly
* Moore 輸出目前狀態
* Meanly 輸出目前狀態與輸入
* encodeing 方法有binary grey one hot
* one hot encoding + unique case 產生clean data path
* enumerate data type 在fsm很好用
* state machines 應該把data 以及控制路徑分開
* 為了避免glitch fsm使用序向boundaries
## ch11 systemVerilog port
### 11.15 summary
* systemVerilog有.name port 讓系統
* systemVerilog 允許巢狀module 架構
* systemVerilog 支援extern module寫入另一個module
* interface encapsulate interconnection to etablish communication between block
* the unspectified interface is referred to as a 'generic' (??
* a modport can also have the expression
* a modport can also have their own name
* module can use modport name
## ch12 verification constructs
### 12.9 summary
* systemVerilog 有一些好用的非合成的語法方便驗證
* function verification is without delays,whereas the timing simulation is with delays;
* the initial procedure block execute once and is used to assign the values of intermediate variables and desired ports.
* it is recommended to use the single procedural block for the clock initialization and clock generation logic
* The reset assertion or de-assertion can be accomplished by using the assignments within the initial procedural block.
* the $display system task can used to display the text by adding the new line character automatically.
* $monitor is one of the important system tasks and used to monitor the output continuously.
* $strobe is used to monitor the output for end of the simulation cycle.
* the verification document can have the information about the strategies, test case, corner cases, test vectors.
## ch13 verification tech and automation
* summary
* the active, inactive, pre-nba, nba,post_nba, observed, post-observed and reactive regions are iterative.
* the fork join thread finishes when all the child threads are over.
* the fork join_any thread finishes when any child thread are spawned.
* the fork join_none thread finished when child threads are spawned.
* the clocking block groups the signals that are synchronous to a desired clock to have their timing expilict.
* the skew must be a constant value and can be specified as a parameter.
* specifying a clocking block using a SystemVerilog interface can significantly reduce the amount of code needed to connect the testbench without race condition.
## ch14 advance verification constructs
## ch15 verification case study
