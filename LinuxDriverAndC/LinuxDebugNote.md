
# linux 開發日常
## 索引
- [keyword](#keyword)
- [dailyProblem](#dailyproblem)
- [git vscode clang](#git-vscode-clang)
- 
## keyword
* 搜尋指令差異([link](https://blog.faq-book.com/?p=1013))
    * whereis (whereis ls)找到特定的檔案
    * which (which vi)找到執行檔
    * find /path
        * find ./Documents/ -name "*.v"
        * find /path/to/directory -regex ".*\.txt$"
            > 搜尋所有以 ".txt" 結尾的文件
        * find /path/to/directory -regex ".*/[0-9].*"
            > 搜尋所有以數字開頭的文件
        * find /path/to/directory -regex ".*file.*\.log$"
            > 搜尋所有名稱包含 "file" 和以 ".log" 結尾的文件
        * find /path/to/directory -regex ".*/(abc|def).*"
            > 搜尋所有以 "abc" 或 "def" 開頭的文件

* dmesg | tail
* dh -f
* mount /dev/hda1 /mnt
    > 将 /dev/hda1 挂在 /mnt 之下。

## dailyProblem

### 20230915 debug read only problem
遇到read only problem
可能是在做更新替換時不小心移除sd card導致權限錯亂
https://blog.csdn.net/jolly10/article/details/87855010
https://serverfault.com/questions/304416/i-cant-delete-files-rm-cannot-remove-x-read-only-file-system
參考這兩篇
1. 使用`cat /etc/mtab` 或`cat /proc/mounts` 找出有問題的磁區
2. `mount -o remount,rw /media/usbdisk` ，remount你剛剛的磁區ex: `sudo mount -o remount,rw  /media/tim/boot/`


## git vscode clang
* 如果github在vscode本地端已經有專案，網路上的版本又已經更新，vscode怎麼更新本地端版本？
    >  點選左下角的分支圖案，旁邊的小圖可以看到有改變的檔案數標示，按下去會說要[pull](https://ithelp.ithome.com.tw/articles/10280729)，選Yes後就會幫你更新本地端的版本了
* 
### .clang-format
.clang-format還不太會用
https://clang.llvm.org/docs/ClangFormat.html
https://cloud.tencent.com/developer/article/2124979
```shell
sudo apt-get install clang-format
clang-format main.cpp -style=LLVM
```
https://gcc.gnu.org/projects/strees/index.html

# Debugging(tracing)
https://bootlin.com/doc/training/debugging/debugging-slides.pdf
SystemTap
    


# [讀書會] Linux環境編程：從應用到內核

連結:  [Linux環境編程：從應用到內核](https://hackmd.io/KYRgLADAZgJgzFAtAQwBxwKyLHYrEBGAbBmIsBARGAQJxgDGUlQA?view)
    
https://hackmd.io/5QaOx2wAS1G8yZuIFV9F_Q?view