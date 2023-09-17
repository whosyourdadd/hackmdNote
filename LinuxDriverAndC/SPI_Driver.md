
# SPI Driver
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