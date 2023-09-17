# 第一次寫 Linux kernel driver就上手
[連結](http://rswiki.csie.org/dokuwiki/_media/courses:100_2:lab10_doc.pdf)

[動手寫 Linux Driver](http://blog.logan.tw/2013/01/linux-driver.html)

### driver基本架構
在 driver 的基本架構中，
1. 向系統註冊一個 driver
2. 項系統註冊我們提供的open、close、read 和 write的服務

重要的Event
* Initial module
  當 driver 被載入之後第一個被呼叫的函式，類似一般 C 語言中的 main function，在此 function 中向系統註冊為字元 device 和所提供的服務
* Open device
  當我們的 device 被 fopen 之類的函式開啟時所執行的對應處理函式
* Close device
  使用者程式關閉我們的 device 時執行的對應處理函式
* I/O control
  使用者可透過 ioctl 命令設定 device 的一些參數
* Read device
  當程式從我們的 device 讀取資料時對應的處理函式
* Write device
  當程式對我們的 device 寫入資料時對應的處理函式
* Remove module
  當 driver 被移除時所執行的處理函式，必須對系統取消註冊 device

### 基本程式架構

```clike=
//Includes essential headers
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>
MODULE_LICENSE("Dual BSD/GPL");

static int demo_init(void) {
 printk("<1>I am the initial function!\n");
 return 0;
}

static void demo_exit(void) {
 printk("<1>I am the exit function!\n");
}

module_init(demo_init);
module_exit(demo_exit);
```
上面程式所看到的 printk 就是在 kernel space
中功能和 printf 功能相近的函式。另外 init function 和 exit function 是可以任意
命名的，只要使用 module_init 和 module_exit 巨集宣告即可。

當我們執行 insmod 指令載入 driver 時，driver 中的 initial function 就會被呼
叫，同樣的 exit function 會再執行 rmmod 指令時被呼叫。請大家再重新 compile
這個範例並且使用 insmod 和 rmmod 測試一次。printk 的輸出一般來說會直接輸
出在 console，如果沒有辦法看到任何輸出，請使用 dmsg 命令或者是到
/var/log/syslog 中察看。

### 加入 open、close、I/O control、read 和 write

```clike=
#include <linux/init.h>
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/fs.h>
static ssize_t drv_read(struct file *filp, char *buf, size_t count, loff_t *ppos)
{
    printk("device read\n");
    return count;
}

static ssize_t drv_write(struct file *filp, const char *buf, size_t count, loff_t *ppos)
{
    printk("device write\n");
    return count;
}

static int drv_open(struct inode *inode, struct file *filp)
{
    printk("device open\n");
    return 0;
}

int drv_ioctl(struct inode *inode, struct file *filp, unsigned int cmd, unsigned long arg)
{
    printk("device ioctl\n");
    return 0;
}

static int drv_release(struct inode *inode, struct file *filp)
{
    printk("device close\n");
    return 0;
}

struct file_operations     drv_fops =
{
    read: drv_read,
    write: drv_write,
    ioctl: drv_ioctl,
    open: drv_open,
    release: drv_release,
};

#define MAJOR_NUM 60
#define MODULE_NAME "DEMO"
static int demo_init(void) {
    if (register_chrdev(MAJOR_NUM, "demo", &drv_fops) < 0) {
        printk("<1>%s: can't get major %d\n", MODULE_NAME, MAJOR_NUM);
        return (-EBUSY);
    }
    printk("<1>%s: started\n", MODULE_NAME);
    return 0;
}
static void demo_exit(void) {
    unregister_chrdev(MAJOR_NUM, "demo");
    printk("<1>%s: removed\n", MODULE_NAME);
}
module_init(demo_init);
module_exit(demo_exit);

```
上面的程式提供了一個字元裝置最基本的架構。使用一個struct file_operations 來設定所有操作對應的 function，這個 structure 的定義可以在linux/fs.h 中找到。在 initial module 的 function 中，透過 register_chrdrv 來註冊一個字元裝置，並將剛剛所設定的 structure 傳給系統。使用者便可透過一般的檔案操作函式來存取該 device，只要在 open、close、I/O control、read 和 write等函式中加上對應的硬體操作，就可以完成一個簡單的 driver 了。另外在 removemodule的時候必須呼叫unregister_chrdev取消裝置註冊，以免系統產生異常喔。

值得一提的是第一個參數 EXAMPLE_MAJOR 可以是 60, 61, 62。如果是正式要釋出的 Driver，就必須要從 Documentation/devices.txt 選取適當的 Major ID。

#### 建立裝置檔案

當driver已經完成，要如何使用程式來開啟該裝置呢？在Linux中，大部分的 device 都以檔案的型態存在/dev/目錄之下。現在我們必須建立一個裝置節點的檔案，透過以下指令來建立對應的裝置節點：mknod /dev/demo c 60 0 其中/dev/demo 是裝置名稱，c 代表字元裝置，60 代表主要版本，0 代表次要版本。當 driver 在向系統註冊的時候也必須用同樣的型態、名稱和主要版本註冊，以免失敗。

### 使用你的driver

當 driver 完成了之後，我們就可以寫一個簡單的測試程式來檢驗 driver 是否
正常運作，其實方式也相當簡單，只要將我們剛剛建立的裝置檔案/dev/demo 當
作一般檔案開啟並測試我們所寫的功能如 read、write 即可。

<b>test.cc</b>
```clike=
#include <stdio.h>
int main()
{
    char buf[512];
    FILE *fp = fopen("/dev/demo", "w+");
    if(fp == NULL) {
        printf("can't open device!\n");
        return 0;
    }
    fread(buf, sizeof(buf), 1, fp);
    fwrite(buf, sizeof(buf), 1, fp);
    fclose(fp);
    return 0;
}
```
~~編譯(arm-linux-gcc --static -gdwarf-2 test.c)此測試程式，並把它放到目標~~

* 編譯(gcc -Wall -o demo demo.c)
* sudo insmod ./demo.ko
板上執行後，可以檢視 driver 的訊息輸出結果觀察程式和 driver 通訊的流程~~

* sudo mknod /dev/demo c 60 0
/dev/example 是我們要存放檔案的路徑，c 代表 Character Device，60 是這個驅動程式的 Major ID，0 是驅動程式的 Minor ID。 
* sudo chmod 666 /dev/example
為了方便測試，我們把這個 Device 改成所有人都可以讀寫。
* sudo dmesg | tail
(driver 的輸出可能由螢幕輸出，或者使用 dmesg、檢視/var/log/syslog 等方式得
到)。

### 讀取 User Space 的資料

在前一節當中我們提供了一個 API 讓 User Space 可以操作 Driver。但是其實我們是不能直接存取 buf 的內容。因為 Kernel Space 與 User Space 有不同的位址空間，所以不能直接存取他們。我們必須借助 copy_from_user 這個 API。

在使用這個 API 之前，我們必需引入 <asm/uaccess.h>：
```clike=
#include <asm/uaccess.h>
```


然後我們就可以使用 copy_from_user 來存取 User Space 的位址空間，舉例來說：
```clike=
ssize_t drv_write(struct file *filp, const char *buf, size_t size, loff_t *f_pos) {
    size_t pos;
    uint8_t byte;
    printk("<1>EXAMPLE: write  (size=%zu)\n", size);
    for (pos = 0; pos < size; ++pos) {
        if (copy_from_user(&byte, buf + pos, 1) != 0) {
            break;
        }
        printk("<1>EXAMPLE: write  (buf[%zu] = %02x)\n", pos, (unsigned)byte);
    }
    return pos;
}
```

值得注意的是 copy_from_user() 會回傳剩下未完成的 byte 數。所以一般來說這個回傳值必須是 0 才是成功地讀入資料。要把資料從 Kernel Space 複製到 User Space 則是使用 copy_to_user() 函式，至於使用方法就不再贅述。

$ echo -n 'abcd' > /dev/demo
$ sudo dmesg | tail 


### 和 I/O 溝通
Driver 主要的目的在於提供一個和硬體溝通的介面給應用程式。目前為止我
們只描述了一個 driver 的模型要如何建構，但是並沒有真正的和硬體有任何通
訊。因為與硬體實際的通訊會因為不同的硬體規格和接腳位置而有所不同，這邊
將以講解概念為主，實作上請同學自行參考硬體的 specs。

I/O 主要分為 memory mapped I/O 和 port I/O，使用哪一種會因系統而異
(80x86 系列大部分是使用 port I/O，而許多 embedded system 則是採用 memory
mapped I/O)，以下將對於兩種不同的使用模式作個簡單的介紹：

#### Memory mapped I/O 
使用和存取 memory 相同的指令進行 I/O，將 I/O port 當作 memory address
的一部份稱為 memory mapped I/O。假設我們的 device 有一個 8 bit 的 I/O port
接到我們的系統，記憶體位址在 0x10000，我們的程式可以透過下列範例所介紹
的方式進行讀寫。

```clike=
#define DATA_PORT (*(volatile char*)(0x10000))
char read_b() {
    return DATA_PORT;
}
void write_b(char data) {
    DATA_PORT = data;
}

```
#### Port I/O

I/O 有獨立的 I/O address 並且使用不同於存取記憶體的命令操作，稱為 port
I/O。當系統所使用的 I/O 為 Port I/O，我們可以使用 inb 和 outb 等系統函式對
I/O port 進行操作，不過在那之前必須先保留 I/O 位址給 driver，下列程式為確
認並保留 PC 系統上的 parallel port 給我們的程式：

```clike=
/* Registering port */
int port = check_region(0x378, 1);
if (port) { 
    printk("<1>parlelport: cannot reserve 0x378\n");
    return -1;
}
request_region(0x378, 1, "demo");

```
當 driver 解除安裝時需要釋放 I/O 位址
```clike=
release_region(0x378,1);
```
接下來，就可以在程式中利用 inb 和 outb 對裝置進行 I/O 囉
```clike=
char read_b() {
     return inb(0x378);
}
void write_b(char data) {
     outb(data, 0x378);
}
```



## Develope the Environment
1. [Building and Install the preeempt kernel on Ubuntu](https://ubuntuforums.org/showthread.php?t=2273355)

2. User space process communcation to kernel space device driver

* device file :
  用來把usr process與driver連起來的角色
* mkmod :
  設定裝置檔的屬性
* major number
  
* minor number

3. kernel device driver communcation to kernel device driver
