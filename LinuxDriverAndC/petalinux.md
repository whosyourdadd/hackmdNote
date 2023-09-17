# 使用petalinux建立image
![image alt](https://img-blog.csdnimg.cn/9d224d64d2964170b8abefe168458c53.png)

## 目錄
- [使用petalinux建立image](#使用petalinux建立image)
  - [目錄](#目錄)
  - [連結](#連結)
  - [指令](#指令)
  - [Module或APP設定](#module或app設定)
  - [build\_image](#build_image)


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
    > petalinux-create --type project --template zynqMP -n <projectname>
3. 導入硬體描述
    > petalinux-config --get-hw-description <xxx.dsa目錄>
    > 
    > 用來導入XSA，會把你指定路徑的XSA放到這個專案內，他會進去抓裡面第一個XSA檔應該是按照檔案名稱順序，若要比免抓錯不要放其他XSA
    
4. 設定kernel
    > petalinux-config
    * DTG Settings->MACHINE_NAME为 zcu104-revc 
    * 進入後比較重要的是ethernet setting可以使用自動IP或固定IP
    * 在image packaging config -> Root file system如果要從INITRD改成EXT4，ethernet會被改成auto
    * 第一次build建議每個選點進去看一下，避免奇怪的錯誤
    * SD card節點設定(need check)

     > petalinux-config -c kernel

5. 建立系統鏡像
    > petalinux-build
    > 如果編譯過程有失敗 可以使用 petalinux-build -c kernel -x distclean 清除做到一半的image
6. 使用qemu開機模擬
    > petalinux-boot --qemu --kernel


若出現
```shell=
Failed to boot. Image file '/<project-path>/pre-built/linux/images/pmu_rom_qemu_sha3.elf' doesn't exist.
```
的時後，直接mkdir -p pre-built/linux/images 再把pmu_rom_qemu_sha3.elf複製進去即可

---
## Module或APP設定
1. create module sample
$ petalinux-create -t modules -n modulename --enable
$ petalinux-create -t apps -n myapp --enable
    >(名稱不要大寫編譯會過不了)
    > 可以在以下路徑找到
    > <project name>/project-spec/meta-user/recipes-modules/
    > <project name>/project-spec/meta-user/recipes-apps/

2. 改成自己所需的moudle以後編譯
    petalinux-build
    petalinux-config -c rootfs
3. 檢查是否可以在選單中找到module
    在專案中可以在以下路徑找到.ko檔
    > project-name/build/tmp/sysroots-components/zynqmp_generic/blink/lib/modules/5.10.0-xilinx-v2021.2/extra/blink.ko

4. 開機後可以在以下路徑找到module driver
    > /lib/modules/5.10.0-xilinx-v2021.2/extra/


## build_image    
https://coldnew.github.io/b394a9ce/
當完成petalinux-build，準備燒板子，執行以下命令，並且把BOOT.bin , image.ub丟到boot區即可
```sh!
petalinux-package --boot --u-boot --fpga ./images/linux/system.bit --force
```
