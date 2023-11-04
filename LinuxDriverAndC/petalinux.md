# 使用petalinux建立image
![image alt](https://img-blog.csdnimg.cn/9d224d64d2964170b8abefe168458c53.png)

## 目錄
- [使用petalinux建立image](#使用petalinux建立image)
  - [目錄](#目錄)
  - [連結](#連結)
  - [指令](#指令)
  - [Module或APP設定](#module或app設定)
  - [Devicetree](#DeviceTree)
  - [build_image](#build_image)


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



## 指令
1. 從vivado導出xxx.xsa
2. 建立petalinux模板   
   ```petalinux-create --type project --template zynqMP -n <projectname>```
3. 導入硬體描述  
    ```petalinux-config --get-hw-description <xxx.dsa目錄>```
    > 
    > 用來導入XSA，會把你指定路徑的XSA放到這個專案內，他會進去抓裡面第一個XSA檔應該是按照檔案名稱順序，若要比免抓錯不要放其他XSA
    
4. 設定kernel  
    ```petalinux-config```
    * DTG Settings->MACHINE_NAME为 zcu104-revc 
    * 進入後比較重要的是ethernet setting可以使用自動IP或固定IP
    * 在image packaging config -> Root file system如果要從INITRD改成EXT4，ethernet會被改成auto
    * 第一次build建議每個選點進去看一下，避免奇怪的錯誤
    * SD card節點設定(need check)   

    ```petalinux-config -c kernel```

5. 建立系統鏡像   
   ```petalinux-build```
    > 如果編譯過程有失敗 可以使用 ```petalinux-build -c kernel -x distclean``` 清除做到一半的image
6. 使用qemu開機模擬  
   ```petalinux-boot --qemu --kernel```


若出現```Failed to boot. Image file '/<project-path>/pre-built/linux/images/pmu_rom_qemu_sha3.elf' doesn't exist.```
的時後，直接```mkdir -p pre-built/linux/images``` 再把pmu_rom_qemu_sha3.elf複製進去即可

---
## Module或APP設定
1. create module sample       
```$ petalinux-create -t modules -n modulename --enable```  
```$ petalinux-create -t apps -n myapp --enable```
    >寫module的範例: [module sample](https://github.com/Xilinx/Embedded-Design-Tutorials/tree/master/docs/Introduction/Zynq7000-EDT/ref_files/example12/LKM)  
    >(名稱不要大寫編譯會過不了)  
    > 可以在以下路徑找到  
    > <project name>/project-spec/meta-user/recipes-modules/  
    > <project name>/project-spec/meta-user/recipes-apps/  

3. 改成自己所需的moudle以後編譯  
    ```petalinux-build```   
    ```petalinux-build -c myapp```      
    ```petalinux-config -c rootfs```   
4. 檢查是否可以在選單中找到module  
    在專案中可以在以下路徑找到.ko檔  
    project-name/build/tmp/sysroots-components/zynqmp_generic/blink/lib/modules/5.10.0-xilinx-v2021.2/extra/blink.ko

5. 開機後可以在以下路徑找到module driver
    > /lib/modules/5.10.0-xilinx-v2021.2/extra/

## DeviceTree  
* ```petalinux-build -c device-tree```可以編譯出```image.ub```，此為系統所使用的devicetree,也可透過petalinux-build直接對全編譯  
* 如果你的功能都沒改只有動device tree可以只更新image.ub加快開發速度
* 以下為DeviceTree兩個node的寫法  
  ```ls /dev/spidev*``` 可得```/dev/spidev1.0  /dev/spidev1.1```
```
/include/ "system-conf.dtsi"
/ {

};
&spi0 {
	status = "okay";
	num-cs = <2>;
	spidev@0 {
		compatible = "rohm,dh2228fv";
		spi-max-frequency = <200000000>;
		reg = <0>;
	};
	spidev@1 {
		compatible = "rohm,dh2228fv";
		spi-max-frequency = <200000000>;
		reg = <1>;
	};
};
```  
* 以下為兩個PS SPI都使用的寫法，記得要改好XSA，否則開機modprobe就會有錯誤訊息，無法產生node  
  ```ls /dev/spidev*``` 可得```/dev/spidev1.0  /dev/spidev1.1  /dev/spidev2.0```

```
/include/ "system-conf.dtsi"
/ {

};
&spi0 {				/* &表示繼承default device tree */
	status = "okay";	/* default都是"disable，要用到就要改成ok" */
	num-cs = <2>;		/* 因為有兩個node會有兩個cs所以num-cs為2 */
	spidev@0 {
		compatible = "rohm,dh2228fv";
		spi-max-frequency = <200000000>;
		reg = <0>;
	};
	spidev_test@1 {		/* 命名為spidev_test或spidev不影響實際在/dev/的命名 */
		compatible = "rohm,dh2228fv";
		spi-max-frequency = <200000000>;
		reg = <1>;
	};
};	

&spi1 {				/*& 表示繼承default device tree */
	status = "okay";
	num-cs = <1>;		/* 因為有一個node會有兩個cs所以num-cs為1 */
	123456@0 {		/* 命名為123456不影響實際在/dev/的命名 */
		compatible = "rohm,dh2228fv";
		spi-max-frequency = <200000000>;
		reg = <0>;
	};
};	
```
* 也可以這樣寫(一個節點的寫法
```
/include/ "system-conf.dtsi"
/ {
	amba: axi {
		spi0: spi@ff040000 {
			compatible = "xlnx,zynq-spi-r1p6";
			status = "okay";
			interrupt-parent = <&gic>;
			interrupts = <0 19 4>;
			reg = <0x0 0xff040000 0x0 0x1000>;
			clock-names = "ref_clk", "pclk";
			#address-cells = <1>;
			#size-cells = <0>;
			power-domains = <&zynqmp_firmware PD_SPI_0>;
			num-cs = <1>;
		    spidev@0{
		        	compatible = "rohm,dh2228fv";
		        	reg = <0x0>;
		        	spi-max-frequency = <200000000>;
		    };
		};	
		
	};

};
```

## build_image    
https://coldnew.github.io/b394a9ce/   
當完成petalinux-build，準備燒板子，執行以下命令，並且把BOOT.bin , image.ub丟到boot區即可
```sh!
petalinux-package --boot --u-boot --fpga ./images/linux/system.bit --force
```
