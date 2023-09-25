
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

```clike=

static int xilinx_spi_probe(struct platform_device *pdev)
{
	struct xilinx_spi *xspi;
	struct resource *res;
	int ret;
	u32 num_cs = 0, bits_per_word = 8;
	u32 cs_num;
	struct spi_master *master;
	struct device_node *nc;
	u32 tmp, rx_bus_width, fifo_size;
	bool startup_block;

	if (of_property_read_u32(pdev->dev.of_node, "num-cs", &num_cs))
		dev_info(&pdev->dev,
			 "Missing num-cs optional property, assuming default as <1>\n");
	if (!num_cs)
		num_cs = 1;

	if (num_cs > XILINX_SPI_MAX_CS) {
		dev_err(&pdev->dev, "Invalid number of spi slaves\n");
		return -EINVAL;
	}

	startup_block = of_property_read_bool(pdev->dev.of_node,
					      "xlnx,startup-block");
	master = spi_alloc_master(&pdev->dev, sizeof(struct xilinx_spi));
	if (!master)
		return -ENODEV;

	xspi = spi_master_get_devdata(master);
	master->dev.of_node = pdev->dev.of_node;
	platform_set_drvdata(pdev, master);
	res = platform_get_resource(pdev, IORESOURCE_MEM, 0);
	xspi->regs = devm_ioremap_resource(&pdev->dev, res);
	if (IS_ERR(xspi->regs)) {
		ret = PTR_ERR(xspi->regs);
		goto put_master;
	}
	ret = of_property_read_u32(pdev->dev.of_node, "fifo-size",
				   &fifo_size);
	if (ret < 0) {
		dev_err(&pdev->dev,
			"Missing fifo size\n");
		return -EINVAL;
	}
	if (of_property_read_u32(pdev->dev.of_node, "bits-per-word",
				 &bits_per_word))
		dev_info(&pdev->dev,
			 "Missing bits-per-word optional property, assuming default as <8>\n");

	xspi->rx_bus_width = XSPI_ONE_BITS_PER_WORD;
	for_each_available_child_of_node(pdev->dev.of_node, nc) {
		if (startup_block) {
			ret = of_property_read_u32(nc, "reg",
						   &cs_num);
			if (ret < 0)
				return -EINVAL;
		}
		ret = of_property_read_u32(nc, "spi-rx-bus-width",
					   &rx_bus_width);
		if (!ret) {
			xspi->rx_bus_width = rx_bus_width;
			break;
		}
	}

	xspi->axi_clk = devm_clk_get(&pdev->dev, "axi_clk");
	if (IS_ERR(xspi->axi_clk)) {
		if (PTR_ERR(xspi->axi_clk) != -ENOENT) {
			ret = PTR_ERR(xspi->axi_clk);
			goto put_master;
		}

		/*
		 * Clock framework support is optional, continue on,
		 * anyways if we don't find a matching clock
		 */
		xspi->axi_clk = NULL;
	}

	ret = clk_prepare(xspi->axi_clk);
	if (ret) {
		dev_err(&pdev->dev, "Failed to prepare AXI clock\n");
		goto put_master;
	}

	xspi->axi4_clk = devm_clk_get(&pdev->dev, "axi4_clk");
	if (IS_ERR(xspi->axi4_clk)) {
		if (PTR_ERR(xspi->axi4_clk) != -ENOENT) {
			ret = PTR_ERR(xspi->axi4_clk);
			goto clk_unprepare_axi_clk;
		}

		/*
		 * Clock framework support is optional, continue on,
		 * anyways if we don't find a matching clock
		 */
		xspi->axi4_clk = NULL;
	}

	ret = clk_prepare(xspi->axi4_clk);
	if (ret) {
		dev_err(&pdev->dev, "Failed to prepare AXI4 clock\n");
		goto clk_unprepare_axi_clk;
	}

	xspi->spi_clk = devm_clk_get(&pdev->dev, "spi_clk");
	if (IS_ERR(xspi->spi_clk)) {
		if (PTR_ERR(xspi->spi_clk) != -ENOENT) {
			ret = PTR_ERR(xspi->spi_clk);
			goto clk_unprepare_axi4_clk;
		}

		/*
		 * Clock framework support is optional, continue on,
		 * anyways if we don't find a matching clock
		 */
		xspi->spi_clk = NULL;
	}

	ret = clk_prepare(xspi->spi_clk);
	if (ret) {
		dev_err(&pdev->dev, "Failed to prepare SPI clock\n");
		goto clk_unprepare_axi4_clk;
	}

	pm_runtime_set_autosuspend_delay(&pdev->dev, SPI_AUTOSUSPEND_TIMEOUT);
	pm_runtime_use_autosuspend(&pdev->dev);
	pm_runtime_enable(&pdev->dev);
	ret = pm_runtime_get_sync(&pdev->dev);
	if (ret < 0)
		goto clk_unprepare_all;

	xspi->dev = &pdev->dev;

	/*
	 * Detect endianess on the IP via loop bit in CR. Detection
	 * must be done before reset is sent because incorrect reset
	 * value generates error interrupt.
	 * Setup little endian helper functions first and try to use them
	 * and check if bit was correctly setup or not.
	 */
	xspi->read_fn = xspi_read32;
	xspi->write_fn = xspi_write32;

	xspi->write_fn(XSPI_CR_LOOP, xspi->regs + XSPI_CR_OFFSET);
	tmp = xspi->read_fn(xspi->regs + XSPI_CR_OFFSET);
	tmp &= XSPI_CR_LOOP;
	if (tmp != XSPI_CR_LOOP) {
		xspi->read_fn = xspi_read32_be;
		xspi->write_fn = xspi_write32_be;
	}

	xspi->buffer_size = fifo_size;
	xspi->irq = platform_get_irq(pdev, 0);
	if (xspi->irq < 0 && xspi->irq != -ENXIO) {
		ret = xspi->irq;
		goto clk_unprepare_all;
	} else if (xspi->irq >= 0) {
		/* Register for SPI Interrupt */
		ret = devm_request_irq(&pdev->dev, xspi->irq, xilinx_spi_irq, 0,
				       dev_name(&pdev->dev), master);
		if (ret)
			goto clk_unprepare_all;
	}

	/* SPI controller initializations */
	xspi_init_hw(xspi);

	pm_runtime_put(&pdev->dev);

	master->bus_num = pdev->id;
	master->num_chipselect = num_cs;
	master->setup = xspi_setup;
	master->set_cs = xspi_chipselect;
	master->transfer_one = xspi_start_transfer;
	master->prepare_transfer_hardware = xspi_prepare_transfer_hardware;
	master->unprepare_transfer_hardware = xspi_unprepare_transfer_hardware;
	master->bits_per_word_mask = SPI_BPW_MASK(bits_per_word);
	master->mode_bits = SPI_CPOL | SPI_CPHA | SPI_CS_HIGH;

	xspi->bytes_per_word = bits_per_word / 8;
	xspi->tx_fifo = xspi_fill_tx_fifo_8;
	xspi->rx_fifo = xspi_read_rx_fifo_8;
	if (xspi->rx_bus_width == XSPI_RX_ONE_WIRE) {
		if (xspi->bytes_per_word == XSPI_TWO_BITS_PER_WORD) {
			xspi->tx_fifo = xspi_fill_tx_fifo_16;
			xspi->rx_fifo = xspi_read_rx_fifo_16;
		} else if (xspi->bytes_per_word == XSPI_FOUR_BITS_PER_WORD) {
			xspi->tx_fifo = xspi_fill_tx_fifo_32;
			xspi->rx_fifo = xspi_read_rx_fifo_32;
		}
	} else if (xspi->rx_bus_width == XSPI_RX_FOUR_WIRE) {
		master->mode_bits |= SPI_TX_QUAD | SPI_RX_QUAD;
	} else {
		dev_err(&pdev->dev, "Dual Mode not supported\n");
		goto clk_unprepare_all;
	}
	xspi->cs_inactive = 0xffffffff;

	/*
	 * This is the work around for the startup block issue in
	 * the spi controller. SPI clock is passing through STARTUP
	 * block to FLASH. STARTUP block don't provide clock as soon
	 * as QSPI provides command. So first command fails.
	 */
	if (startup_block)
		xilinx_spi_startup_block(xspi, cs_num);

	ret = spi_register_master(master);
	if (ret) {
		dev_err(&pdev->dev, "spi_register_master failed\n");
		goto clk_unprepare_all;
	}

	return ret;

clk_unprepare_all:
	pm_runtime_disable(&pdev->dev);
	pm_runtime_set_suspended(&pdev->dev);
	clk_unprepare(xspi->spi_clk);
clk_unprepare_axi4_clk:
	clk_unprepare(xspi->axi4_clk);
clk_unprepare_axi_clk:
	clk_unprepare(xspi->axi_clk);
put_master:
	spi_master_put(master);

	return ret;
}


```

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
