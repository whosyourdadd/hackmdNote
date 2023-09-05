# VHDLandVLSI_Note
如標題

# VHDL_Verilog上課筆記
上課用的開發板
https://store.digilentinc.com/basys-3-artix-7-fpga-trainer-board-recommended-for-introductory-users/
自學網站
http://www.asic-world.com/


IP 交易平台
https://www.design-reuse.com/

```shell=
##$ source /opt/Xilinx/Vivado/2017.4/settings64.sh
$ source /media/kkman00234/LENOVO/Xilinx/Vivado/2016.4/settings64.sh

$ vivado
```


* Verilog語法要知道那些可合成，那些不行
* 如果是用verilog指的應該就會是digital designer
* 那麼準備方向就會是，logic circuit熟練，design flow了解，verilog熟練


https://hdlbits.01xz.net/wiki/Step_one



## 章節重點整理
### ch1
* Typical Design Flow
* Hardware synthesis
![](https://i.imgur.com/bn9yxe7.png =60%x)

assign block
assign non-block

### ch2 Hierarchical Modeling and simulate
* Hierarchical Modeling
    * four levels : Behavioral , data flow gate level, switch
    * top-down bottom-up
* module
    * 裡面不能在放module
* system function(輔助命令)
    * $monitor
    * $time
    * $display 
    * $stop 暫停模擬
    * $finish 中止模擬

### ch3_verilog_basic_concept
* comment // /* ... */
* operator +-&|
* number
    * 4'b1101
    * 12'ha0ef 
    * 16'd1234 
    * 沒特別給位元數就以機器決定(32 /64 bit)
* x/z
    * x unknow
    * z high impedance
* negative number
    * -8'd3   (三 儲存在八位元二補數中)
    *  4'd-2  (不合法的規格)
* underscore characters and question marks
    * 12'b1111_0000_1010 (底線可以增加可讀性)
    * 4'b10??  (等同於4'b10zz)
    * 問號在casex/casez會有更深的討論 
* string
* Identifiers and keywords
    * reg/input/... 
* Data value
    * strong 1 + weak 0 = strong 1
    * strong 1 + strong 0 = unknow x
    * net(wire)： 沒有驅動就是高阻抗z 
    * reg :不需要驅動 
    * 注意net vs reg的關係與差異 感覺很重要！！！！！！
    * vector : wire/reg都可以宣告 
    * integer real time (不確定是否可以被合成)
    * real number assign to integer => 自動捨去小數點
    * time Tstamp = $time
    * array : reg arrayEx[3:0]
    * vector: reg [3:0] vectorEx
    * array != vector
    * parameter : just like constant in C
    * string storage: 每個字元都需要1byte才能存起來
    * define include 跟c的一樣
    * wire  需搭配assign使用
    * reg 需搭配always block使用
      
### ch4 module ,ch5 Gate-level Modeling (2 basic class)
* and/or gate
  and or xor nand nor xnor
* buf/not gate

    <閘名稱> <閘編號> ( 輸出埠, 輸入埠1, 輸入埠2… );

    閘名稱：使用的邏輯閘名稱( ex. and, or, nor... )
    閘編號：給予該閘名稱或編號，可不寫，交由軟體處理
    輸出埠：輸出埠只能有一個，並且寫在最前面
    輸入埠：輸入邏輯閘的資料口，需使用逗號分別 
* notif truth table
![](https://i.imgur.com/tKnSvW1.png =60%x)

* delay -- rise/fall/turn-off delay
    turn off delay 其他值到高阻抗的延遲時間
### ch6 Dataflow Modeling
 
*    驅動某值至net ( 等號左式只能是net，右式可以是 net 或 reg )
*    資料流層次的描述方式，只能敘述組合邏輯電路( 不含有記憶性電路 )
*    但輸出不可以包含輸入( EX : a = a + b; → 隱含有記憶性 → 錯誤 )


## 雜記
### DIP

數位ic的基礎,


Switch Level
不能實做

verilog language review
hardware desigh review
verilog ide review

# VSLI_Design_大型積體電路設計

* Introduction
* Logic Design with MOSFETs
* Physical structure of CMOS ICs
* Fabrication of CMOS ICs
* Elements of Physical Design
* Electrical characteristics of MOSFETs
* Electronic analysis of CMOS logic gates
* Designing high speed CMOS logic networks
* Advanced techniques in CMOS logic circuits
* Arithmetic circuits in CMOS VLSI
* Memory design



https://hom-wang.gitbooks.io/verilog-hdl/content/Chapter_06.html

[中原電機讀書會](https://sites.google.com/site/cycueehdlsg/)

http://www.mwftr.com/SoCs14/verilog_intro_2002.pdf

http://www.csd.nutn.edu.tw/materials.htm
--

## Ch2 Logic Design with MOSFETs

### DIGITAL LOGIC

數位邏輯電路用來運行邏輯值，通常被電壓範圍代表，這些1/0 high/low true false透過MOSFET計算建構出來，而這些元件是類比的。電壓界於VDD~臨界區間是1，臨界電壓至0 GND是0，如果電壓界於臨界區間之間，在未定義區域邏輯會出問題。



![](https://i.imgur.com/lT3V7yJ.png =20%x)



Figure 1	Fig. 1: Mapping from voltages to logical values. Voltages between ground and a certain threshold represent the logical value 0. Voltages between a higher threshold and VDD represent the logical value 1. The threshold levels are design choices. If a voltage falls in the gap between the defined logical ranges, the result is undefined and there must be an error in the logic circuit that produced it.

### MOSFET APPROXIMATIONS

MOSFETs can be approximated as either open or short circuits between drain and source.
MOSFET透過集極D，源極S來區近開/短路迴路
將整個範圍的電壓對應到單一的邏輯有利於積極逼近，來做電路分析。舉例來說，我們想要得到一個1，我們不需要去檢察目前的電壓，只需要接近VDD即可。這樣的想法對於NMOS/PMOS的邏輯電路設計很有用。

NMOS/PMOS可參考下圖

![](https://i.imgur.com/8Ej1HdB.png =60%x)

NMOS的Drain通常在上，SOURCE通常在下，PMOS剛好相反並且Gate有個圈圈。Gate =1表是開關導通。NMOS A= 1導通，PMOS = 0導通。



### CMOS LOGIC CIRCUITS

CMOS = NMOS+PMOS的電晶體組成。如下圖所示:

![](https://i.imgur.com/9YdynM2.png =15%x)
Figure 2


CMOS可以確保一定是VDD或是短路，非一個模糊的狀態。並且確保不會從gnd穿過z，這個設計確保cmos的效率。圖二是一個input的邏輯電路，電路叫做inverter或not。因為你在控制A=0的時候剛好Z=1(VDD)，A=1則Z=0(GND)，原理如下圖所示。
![](https://i.imgur.com/zOb4t3o.png =70%x)

Figure 3
![](https://i.imgur.com/Jy5YNfj.png =20%x)
圖三是一個兩個輸入的邏輯電路。這個電路(NAND)兩個輸入A/B。注意PMOS是並聯，NMOS是串連。這樣的做法確保電壓不是0就是1。


REF: [四、 場效電晶體原理](http://140.120.11.1/semicond/handout/chap4.pdf)
