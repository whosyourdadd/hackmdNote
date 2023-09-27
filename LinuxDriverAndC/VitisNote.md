# VitisNote

## standalone操作流程
參考連結
* [官方使用Standalone Software for PS Subsystems](https://xilinx.github.io/Embedded-Design-Tutorials/docs/2021.2/build/html/docs/Introduction/ZynqMPSoC-EDT/4-build-sw-for-ps-subsystems.html)  
* [hackster.io 參考流程](https://www.hackster.io/BryanF/ultra96-v2-vitis-2020-2-hello-world-from-arm-a53-2d952a)  
* [coldnew.github 板子介紹](https://coldnew.github.io/b969f0df/)  
* [Ultra96 v1/v2 Getting started with Xilinx Vitis  yt ](https://www.youtube.com/watch?v=gv4kXKqoWYc&ab_channel=96Boards)  
### 大方向
* vivado project建立
    1. vivado 先去網路上更新bsp，才能找到avnet板子的bsp
    2. 以Ultral96-V2建立block diagram
    3. systhises,implentment,（沒有pl external就不用設定io）
    4. bitstream,建立xsa
* vitis project建立
    1. 根據xsa建立platform project  
    2. 整體流程可參考hackster.io[這篇](https://www.hackster.io/BryanF/ultra96-v2-vitis-2020-2-hello-world-from-arm-a53-2d952a)，比較需要注意的是設定standalone BSP時stdin,stdout要設定uart1  
 ![](https://hackster.imgix.net/uploads/attachments/1289330/image_7j3L4DO7gN.png)  
 ![](https://hackster.imgix.net/uploads/attachments/1289331/image_J8cfhdWAuY.png)     
    3. 設定好之後對platform project,build project  
    4. 確認都沒問題之後，建立appilcation project  
    5. 選擇剛剛建立好的platform project作為基底，並且選你要使用的core  
    6. 選擇hello world project產生模板，並且直接build project  
    7. 檢查switch是否使用jtag模式，檢查uart連接的pin腳，並且上電開機  
 ![](https://hackster.imgix.net/uploads/attachments/1290275/image_RXwHPazifE.png)  
    8. 使用run as將hello world燒錄在板子上，並可使用putty或是vitis本身的console去看是否有印出hello world  
    9. 若發現jtag無法連線成功，權限被拒絕，記得使用sudo 並找出vitis的路徑執行，方可正確運行  
