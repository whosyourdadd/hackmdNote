# bring_up

目標:

1. 了解一般產業當中bring up在做甚麼
2. 認識bootloader
3. 找到相關流程，並且可以跟著流程走一遍



* [万字连载（上）：如何 Bringup SoC 芯片](https://blog.csdn.net/melody157398/article/details/124678825)
* [谈论bringup我们到底在谈论什么？](https://blog.csdn.net/codectq/article/details/104484568?spm=1001.2101.3001.6650.6&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-6-104484568-blog-129807113.235%5Ev38%5Epc_relevant_anti_vip&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-6-104484568-blog-129807113.235%5Ev38%5Epc_relevant_anti_vip&utm_relevant_index=7)
* [[RPi bring up] hello world! 树莓派裸机点亮led](https://blog.csdn.net/sdfui32iruiwed/article/details/125401182)

## 工作的時候別人會希望你會的知識內容
1. Board bring-up
2. Linux booting process
3. Interrupts in linux
4. Function pointer, linked list and strings

## 如何進行bring up 樹梅派為例

### 基本步驟

* 驗證硬體: 在開始進行之前，需要先驗證電路板中的處理器，以及其他硬體是否正確，可使用硬體驗證版測試基本電路完成此功能
* 配置bootloader
* 移植與訂製驅動程序:
    根據具體的設備需求和應用需求，可能需要移植、制定或編寫操作系統驅動程序，確保所有硬體資源與外圍設備可正常運行
* 系統集成測試

### 實例
* 樹梅派是使用Broadcom bootloader啟動，採用BCM2835/6/7處理器，這些
* Broadcom bootloader通常放在bootcode.bin當中，處理器會先在SD卡當中載入該文件，樹梅派的bootloader允許使用者在SD卡中配置config.txt以及設備樹(.dtb)，......等等

![image alt](https://img-blog.csdnimg.cn/db2085e75b4646a8b37f8a6454c22cae.png)]
* 啟動流程: 上電 -> bootcode.bin -> stard.elf -> fixup.dat ...
https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#raspberry-pi-boot-modes

![image alt](https://img-blog.csdnimg.cn/img_convert/e91be85ccd6d8e738d3b1cd8183ccc1a.png)
## xilinx zynq boot
https://www.xilinx.com/support/documentation-navigation/design-hubs/dh0072-zynq-mpsoc-boot-and-config-hub.html
* Embedded systems (usually) don’t have BIOS, ACPI and/or UEFI.