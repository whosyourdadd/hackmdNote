# VerilogMiscNote

## 參考連結
* [Must-have verilog systemverilog modules](https://github.com/pConst/basic_verilog)
    * [spi master](https://github.com/pConst/basic_verilog/blob/master/spi_master.sv)
    * [spi testbench](https://github.com/pConst/basic_verilog/blob/master/spi_master_tb.sv)
    * [xilinx qspi ip module](https://docs.xilinx.com/r/en-US/pg153-axi-quad-spi/AXI-Quad-SPI-v3.2-LogiCORE-IP-Product-Guide)
    * [Verilog-Design-Examples](https://github.com/snbk001/Verilog-Design-Examples)
* [SPI problem](https://hackmd.io/Q9jhWS5iQdiWSupXXOAfJw?view)
* BRAM相關
    * Block memory https://zipcpu.com/tutorial/lsn-08-memory.pdf
    * yodalee.me Block memory https://yodalee.me/2021/09/openfpga_bram/
    * [zynq bram  ps pl端數據交換](https://blog.csdn.net/wangjie36/article/details/117607389)
    * https://blog.csdn.net/qq_24815615/article/details/89843000
    * https://www.toutiao.com/article/7143256726588965416/?wid=1688859273592
    * [自己寫紅外線IP 放到BRAM再從BRAM寫讀資料出來](https://blog.csdn.net/weixin_45637597/article/details/122211343)
    * https://www.cnblogs.com/suozhang/p/13915640.html
    * Xilinx PPT [axi4 Creating and Adding Custom IP]( 
https://xilinx.eetrend.com/files-eetrend-xilinx/forum/201509/9208-20395-creating_and_adding_custom_ip.pdf)
* https://ys-hayashi.me/2021/09/xilinx-ahb-soc-01/
* https://www.youtube.com/watch?v=pxbmNsWoId8&ab_channel=FastWalkthrough
* 
## 想看的東西
### RSIC-V
* [從零開始的RISC-V SoC架構設計與Linux核心運行 - 硬體篇](
https://hackmd.io/@w4K9apQGS8-NFtsnFXutfg/B1Re5uGa5#SoC%E6%9E%B6%E6%A7%8B)
* https://github.com/yutongshen/RISC-V-Simulator
* https://wiki.csie.ncku.edu.tw/User/Chiwawachiwawa
* https://hackmd.io/@CWWPPB/rJIBMMtpi
* [Linux 核心專題: 將 Linux 執行於 FPGA 為基礎 RISC-V 處理器](
https://hackmd.io/@sysprog/B1Jl_HlBn)

### fpga open source ide
https://github.com/YosysHQ/icestorm
Lattice iCE40-UP5K
nextpnr -- a portable FPGA place and route tool

* 開源 FPGA/Verilog 入門學習研討與工作坊
:::success
OpenSource FPGA/Verilog Learning Environment Introduction & Workshop
- 9:00-10:00 開源 FPGA Toolchain 簡介
    - OpenSource IceStorm 工具鏈介紹
    - 簡單易用的 APIO 與 VSCode Verilog 開發環境
    - 適合教學的圖形開發環境 ：IceStudio IDE
- 10:00-11:00 開源 FPGA 開發板介紹與電路設計
    - Lattice iCE40-UP5K 晶片的應用電路設計
    - WiFiBoy FPGA 玩學機 OK:iCE40Pro 電路設計
- 11:00-12:00 開源 CPU 設計介紹
    - 開源 32bit RISC-V CPU/SOC 實作介紹
    - 開源 8bit NES VGA 遊戲機實作介紹
    - 開源 16bit Forth CPU J1A 實作介紹
- 13:00-1400 簡單易學的 8bit CPU 設計教學
    - OK8 CPU 原始程式解析
    - OK8 如何做 Blink 範例
    - OK8 如何加入有趣的 Sound Generator 範例
- 14:00-16:00 工作坊
    - icestorm 開發環境安裝實務操作
    - APIO 與 VSCode 開發環境安裝與實務操作
    - IceStudio 開發環境安裝與實務操作
    - 三款開源CPU的實務操作
    - 極簡 OK8 CPU 的實務操作
:::
https://hackmd.io/@HsuChiChen/content?utm_source=preview-mode&utm_medium=rec
[這幾個禮拜完成的作品 - RISC-V處理器](https://hackmd.io/@w4K9apQGS8-NFtsnFXutfg/HkLq0BIUI)
[伴伴學RISC-V](https://hackmd.io/@accomdemy/r1pNfeBdq)
# 寫verilog的一些問題經驗整理
目前想把一些看到的code整理下來，
1. reg 只能有一個輸入
* [以前的一些整理](###ch3_verilog_basic_concept)
2. compiler 出現module not found並且add source去跳出在清單上要overwrite ⇒ 開頭`include "yourmodule.v"
3. 如果想要做多個不同的simu可以用create多個sim set來達成
4. module的input宣告方式必須是wire的形式，以上面這樣寫的預設值就是wire
5. latch VS FF
    * Latch由組合邏輯組成，緣觸發時才會動作
    * flip-flop由 
6. custom spi ip 開發遇到哪些問題？如何改善
    * 首先除了撰寫spi ip外，完成testbench初步確認波形是否正確
    * 輸出訊號不如預期(三角波)，首先降低時脈確認時脈正確性，並檢查rising time，調整driving strength與slew rate(雖然效果不大)
    * 與原先xilinx完成的ip core進行波形比較
    * 確保 SPI 控制器產生的時鐘邊緣和外部 IC 預期接收的邊緣對齊。這是確保數據能夠準確讀取和寫入的關鍵。
    *  
https://www.xilinx.com/video/hardware/timing-analysis-controls.html
sample code

# Xilinx
## vivado操作流程
* [Getting Started with Vivado IP Integrator](https://docs.xilinx.com/r/en-US/ug994-vivado-ip-subsystems/Getting-Started-with-Vivado-IP-Integrator)
* project create (選板子/選晶片)
* add source (看狀況加入verilog)
* block diagram
* synthesis
* implement
* io pin
* bit generate

## AXI Block RAM
arm 的spec http://www.gstitt.ece.ufl.edu/courses/fall15/eel4720_5721/labs/refs/AXI4_specification.pdf
xilinx ug1037
https://docs.xilinx.com/v/u/en-US/ug1037-vivado-axi-reference-guide

![](https://hackmd.io/_uploads/HkrnjzTY2.png =110%x)


[AXI_lite代码简解-AXI-Lite 源码分析](https://xilinx.eetrend.com/blog/2020/100054854.html)
>在调用写函数时，如果不做地址偏移的话， axi_awaddr[3:2]的值默认是为0的，举个例子，如果我们自定义的IP的地址被映射被调用时地址映射为为0x43C00000，那么Write(0x43C00000,Value)写的就是slv_reg0的值。如果地址偏移4位，如Write(0x43C00000 + 4,Value) 写的就是slv_reg1的值，依次类推。分析时只关注slv_reg0（其他结构上也是一模一样的）

### 時序紀錄
![](https://hackmd.io/_uploads/r15UO4KY2.png)
http://www.gstitt.ece.ufl.edu/courses/fall15/eel4720_5721/labs/refs/AXI4_specification.pdf



![image alt](https://pic2.zhimg.com/80/v2-7ad434149429ed0e69f34b3e8e9370a1_720w.webp)

![image alt](https://pic2.zhimg.com/80/v2-fe1793d2fdcf07d11266c6c0605f99ad_720w.webp =70%x)
如下图所示为一个 AXI4 的实例，Xilinx 的 ZYNQ 系列 FPGA 通过总线互联AXI Interconnect 连接到 AXI BRAM Controller 控制 BRAM 存储器资源，五种颜色的内分别表示一个通道，从上至下依次为读地址通道（araddr）、写地址通道（awaddr）、写响应通道（bresp）、读数据通道（rdata）和写数据通道（wdata），每个通道中均有 valid 和 ready 握手信号。
![image alt](https://pic1.zhimg.com/80/v2-9d52557aae8c98f2378931805b925388_720w.webp =100%x)
读地址/写地址通道，主机通过这两个通道向从机写入地址和控制信息，通道的方向为主机 Master 向从机 Slave 传输，通道内除了 ready 信号外的其余信号均为输出，valid 为高电平时表示主机认为自己输出的数据有效，ready 信号为输入信号，由从机 Slave 给出，当 ready 为高时表示从机已经准备好接收主机的数据，ready和valid同时为高时代表从机准备好接收主机数据且主机此时发送了有效数据，正确的传输开始。除了传输地址外，arlen[7:0] 代表突发传输的长度，8 位可表示 0~255，代表传输长度 1~256。
[CSDN xilinx AXI4 bus ]
https://blog.csdn.net/DengFengLai123/article/details/113530982

![image alt](https://img-blog.csdnimg.cn/20210201221602921.png =110%x)

![image alt](https://img-blog.csdnimg.cn/20210201221558474.png)

https://blog.csdn.net/QUACK_G/article/details/125951718
![image alt](https://img-blog.csdnimg.cn/93258e6cfc5f4fbea59d501496b22e4b.png)

![image alt](https://img-blog.csdnimg.cn/8de2eb10e4774ef09718b971f95c1ddb.png)