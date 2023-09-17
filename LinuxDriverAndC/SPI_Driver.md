
# SPI Driver

## Driver比較
想看看其他人都怎麼寫SPI DRVIER的

| [spi-zynqmp-gqspi.c](https://github.com/Xilinx/linux-xlnx/blob/master/drivers/spi/spi-zynqmp-gqspi.c)  | [spi-bcm2835.c](https://github.com/Xilinx/linux-xlnx/blob/master/drivers/spi/spi-bcm2835.c) |備註 |
| ---- | --- |--- |
| [xilinx gqspi IP spec](https://docs.xilinx.com/r/en-US/pg153-axi-quad-spi/AXI-Quad-SPI-v3.2-LogiCORE-IP-Product-Guide) |[BMC2835 IP spec](https://pdf1.alldatasheet.com/datasheet-pdf/view/502533/BOARDCOM/BCM2835.html) <br>BOARDCOM (BMC2835本身是一個periphals)  | |
| qspi_probe          | bmc spi probe
| spi_alloc_master    | devm_spi_alloc_master  ||
| spi_controller_get_devdata    | platform_set_drvdata  ||
| platform_set_drvdata    | spi_controller_get_devdata  ||
| of_device_get_match_data    | devm_platform_ioremap_resource  ||
| devm_platform_ioremap_resource    | devm_clk_get <br>clk_get_rate <br>platform_get_irq  ||
| devm_clk_get <br>clk_prepare_enable <br>init_completion|bcm2835_dma_init <br>bcm2835_wr(clear txrx)|| 



```clike
list_for_each_entry(xfer, &msg->transfers, transfer_list) {
		if (xfer->len > maxsize) {
			ret = __spi_split_transfer_maxsize(ctlr, msg, &xfer,
							   maxsize, gfp);
			if (ret)
				return ret;
		}
	}

```


## SPI debug討論
目前TIM遇到使用user space用ioc寫入spi時遇到cs 週期過長問題，並且他的示波器結果、printk、使用vivado的工具產出的deubg信號三者沒辦法對起來．目前確認結果：
    * 使用ps mio spi跟外掛pl qspi(內部會設定成一般spi)都會有cs delay問題，因此判斷可能真的是driver問題
    * 我認為有可能是在等接收信號的問題所以cs才會開這麼久
    * 看能不能使用別人寫好的code，針對cs試試看
    * [spidev - 辣個 userspace 的驅動程式](https://ithelp.ithome.com.tw/articles/10248619)
* [如何使用linux內核提供SPI設備驅動Spidev](https://hackmd.io/@amberchung/linux-spidev)
    可以參考linux內核裡tools/spi/spidev_test.c提供的範例，調用open/ioctrl/read/write/close等API函數進行SPI測試驗證
    * 這裡簡單描述一下各個API的主要功能
        * open：開啟設備
        * ioctrl：根據底下傳入的參數做相對應的操作
            1. 可讀取或寫入spi相關設定
                * SPI_IOC_WR_MODE：寫模式
                * SPI_IOC_RD_MODE：讀模式
                * SPI_IOC_WR_BITS_PER_WORD：設置每個byte的有效位
                * SPI_IOC_RD_BITS_PER_WORD：讀取設置的每個byte的有效位
                * SPI_IOC_WR_MAX_SPEED_HZ：設置spi最大速度
                * SPI_IOC_RD_MAX_SPEED_HZ：讀取設置的spi最大速度
                * SPI_IOC_WR_LSB_FIRST：設置為最低位元傳送或接收
                * SPI_IOC_RD_LSB_FIRST：讀取設置的spi最低位元
            2. spi資料傳送接收
                * SPI_IOC_MESSAGE(n)：n為傳送的數量
        * write：只傳送不接收spi資料
        * read：只接收不傳送spi資料
        * close：關閉設備
         https://github.com/torvalds/linux/tree/master/tools/spi     https://www.kernel.org/doc/html/latest/spi/spidev.html