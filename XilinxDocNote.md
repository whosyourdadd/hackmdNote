# Xilinx Doc Note
一些看文件之後的整理

## Zynq UltraScale+ MPSoC Embedded Design Methodology Guide (UG1228
https://docs.xilinx.com/v/u/en-US/ug1228-ultrafast-embedded-design-methodology-guide
### ch1 introduce
* 四種power domain 
    * low power domain
    * full power
    * PL power domain
    * Battery power domain
    四個電源是分開且隔離，使用PMU管理(Platform mangment unit)
* Vector mathodology diagram
    ![](https://hackmd.io/_uploads/SkOzdM7nn.png =70%x)
    * 向量圖從zynqmp的特性展開並可針對使用情境的特性去評估，以上圖adas的特性來說，需要特性原則就是及時運算，其他電源、儲存媒介、安全性就沒這麼重要，根據需求針對板子去分配資源，也是這份guide想要做的
* block
![](https://hackmd.io/_uploads/B1IwWNQ3n.png)
![](https://hackmd.io/_uploads/Sk1YbEQhh.png)

* RPU
    * 32bit operation
    * 600mhz
使用 SoC 將處理卸載到 PL 的過程可概括為：
1. 基於硬體功能
2. 最佳化矩陣計算
3. pipeline 效能
4. optimize struture
5. 減少latency
6. improve area

--
評估軟體開發需求
1. Would you prefer running bare metal or using an OS?
2. If you'd prefer bare metal, what are the specific reasons behind this choice?
3. Do you need real-time capabilities?
4. Do you have any boot time constraints? If so, how firm are those?
5. What is your preferred build or development environm
# Vitis High-Level Synthesis User Guide(UG1399)
https://github.com/Xilinx/Vitis-HLS-Introductory-Examples

https://docs.xilinx.com/r/en-US/ug1399-vitis-hls/Mixing-Data-Driven-and-Control-Driven-Models
https://docs.xilinx.com/r/en-US/ug1399-vitis-hls/Enabling-the-Vivado-IP-Flow

## Design Principles
這個文件主要就是提供一個好的設計原則去合成電路，作為C\C++ 演算法加速的原則
### Throughput(吞吐量) and Performance
Throughput 定義為每單位時間執行的特定操作的數量或每單位時間產生的結果的數量。 這是以每單位時間生產的任何東西（汽車、摩托車、I/O 樣本、內存字、迭代），例如，“記憶體頻寬”有時用於代表記憶體系統的吞吐量， 同樣，性能不僅被定義為更高的吞吐量，而且還被定義為更高的吞吐量和低功耗。 在當今世界，降低功耗與提高吞吐量同樣重要。
### Architecture Matters(架構很重要)
要了解客製硬體如何加速，必須要先了解你的程式是怎麼在傳統硬體上跑的，范紐曼架構可程式化流程主宰了電腦系統七十年，具有multi process,multi thread，達到高吞吐量跟效能。

這樣的系統大量的應用在手機，電腦，遊戲串流...，但現在的挑戰是，怎麼設計一種新的可編成架構，讓你保有足夠的可編程性，同時實現高吞吐量與低電源消耗。

FPGA提供可編程並提供足夠的記憶體頻寬與低功耗的解決方案，不同於CPU跑程式，FPGA提供客製電路來處理需求信號行為，將特出運算映射在裝置上做併行運算

### 三種模式
* 生產者消費者
* 串流資料
* pipeline
### Abstract Parallel Programming Model for HLS
在做平行化之前要先寫出非平行化的程式，去比較改善前改善後
Both blocking and non-blocking read and write semantics are supported for channels, as described in [HLS Stream Library](https://docs.xilinx.com/r/o4VR_92ERnsC86VNmOmjrw/Bj1AYtJD7OOZJM22wSB67g)
![image alt](https://docs.xilinx.com/api/khub/maps/o4VR_92ERnsC86VNmOmjrw/resources/Ls7mKl2HTCmRJNLSgPwVgw/content?Ft-Calling-App=ft%2Fturnkey-portal&Ft-Calling-App-Version=4.2.6&filename=tmx1663362907208.image)
### Control and Data Driven Tasks
