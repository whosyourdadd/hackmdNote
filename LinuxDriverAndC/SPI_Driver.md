
# SPI Driver

## 連結

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

| [spi-zynqmp-gqspi.c](https://github.com/Xilinx/linux-xlnx/blob/master/drivers/spi/spi-zynqmp-gqspi.c)  | [spi-bcm2835.c](https://github.com/Xilinx/linux-xlnx/blob/master/drivers/spi/spi-bcm2835.c) | [spi-cadence.c](https://github.com/Xilinx/linux-xlnx/blob/master/drivers/spi/spi-cadence.c)| 
| ---- | --- | --- |
| [xilinx gqspi IP spec](https://docs.xilinx.com/r/en-US/pg153-axi-quad-spi/AXI-Quad-SPI-v3.2-LogiCORE-IP-Product-Guide) |[BMC2835 IP spec](https://pdf1.alldatasheet.com/datasheet-pdf/view/502533/BOARDCOM/BCM2835.html) <br>BOARDCOM (BMC2835本身是一個periphals)  | xilinx ps spi ctrl |
| qspi_probe          | bmc spi probe | cdns_spi_probe | 
| spi_alloc_master    | devm_spi_alloc_master  | spi_alloc_master |
| spi_controller_get_devdata    | platform_set_drvdata  | spi_master_get_devdata|
| platform_set_drvdata    | spi_controller_get_devdata  | platform_set_drvdata |
| of_device_get_match_data <br> devm_platform_ioremap_resource  | devm_platform_ioremap_resource  | devm_platform_ioremap_resource |
| devm_clk_get("pclk") <br> devm_clk_get("ref_clk") <br> clk_prepare_enableX2 <br>  | devm_clk_get <br>clk_get_rate <br>platform_get_irq  | devm_clk_get("pclk") <br> devm_clk_get("ref_clk") <br> clk_prepare_enableX2 <br>| 
| init_completion| | | 
| pm_runtime_useX4 <br> of_property_read_bool("has-io-mode") <br> pm_runtime_get_sync <br> zynqmp_qspi_init_hw|bcm2835_dma_init <br>bcm2835_wr(clear txrx)|pm_runtime_useX5 <br> of_property_read_u32("num-cs") <br> of_property_read_u32("is-decoded-cs") <br> cdns_spi_detect_fifo_depth <br> cdns_spi_init_hw <br> | 
|platform_get_irq <br> devm_request_irq <br> dma_set_mask("DMA...")|  | platform_get_irq <br> devm_request_irq <br> |
|ctlr->num_chipselect <br> ctlr->bits_per_word_mask <br> zynqmp_qspi_mem_ops <br> zynqmp_qspi_setup_op <br> ctlr->bits_per_word_mask <br> ctlr->dev.of_node <br> ctlr->auto_runtime_pm <br> ctlr->multi_cs_cap| | master->use_gpio_descriptors<br> prepare_transfer_hardware <br> cdns_prepare_message <br> cdns_transfer_one <br> cdns_spi_chipselect <br> master->mode_bits <br> clk_get_rate(ref clk) <br> master->max_speed_hz <br> master->bits_per_word_mask|
|pm_runtime_mark_last_busy <br> pm_runtime_put_autosuspend||pm_runtime_mark_last_busy <br> pm_runtime_put_autosuspend <br> spi_register_master|



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
