
# SPI Driver

## 連結

Petalinux设计流程:  https://blog.csdn.net/weixin_55796564/article/details/128477894  
Vitis下Linux应用程序开发流程: https://blog.csdn.net/clj609/article/details/115442338  
petalinux 修改设备树: https://www.cnblogs.com/YYFaGe/p/14453608.html  

* Driver 
    * spi-zynqmp-gqspi.c 
https://github.com/Xilinx/linux-xlnx/blob/master/drivers/spi/spi-zynqmp-gqspi.c#L282 
    * gpio-xilinx.c
https://github.com/Xilinx/linux-xlnx/blob/master/drivers/gpio/gpio-xilinx.c
    * Creating Custom IP and Device Drivers for Linux¶
https://xilinx.github.io/Embedded-Design-Tutorials/docs/2021.1/build/html/docs/Introduction/Zynq7000-EDT/8-custom-ip-driver-linux.html
    * Johannes4Linu GPIO
    https://github.com/Johannes4Linux/Linux_Driver_Tutorial/tree/main/04_gpio_driver
    * Johannes4Linu IOCTL
    https://github.com/Johannes4Linux/Linux_Driver_Tutorial/tree/main/13_ioctl

https://hackmd.io/@amberchung/linux-spi-subsystem

[Yocto Linux #1 - Initial setup and ZedBoard bring-up](https://www.youtube.com/watch?v=XPnmB-THjiY&t=670s&ab_channel=BOPV)
https://github.com/NonerKao/syscall30

[linux驱动之spi框架](https://www.jianshu.com/p/5b2586b642a9)


## 想看看其他人都怎麼寫SPI DRVIER的  

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
