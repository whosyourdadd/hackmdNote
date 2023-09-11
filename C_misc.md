
# C_Misc
記錄一些小範例

## ptr_casting
```clike=
//little endian or big endian checking
//指標轉型的時候有機會用到
int EndianChecking(unsigned char printArg){
    unsigned int x = 0x76543210;
    char *c = (char*) &x;
    if (*c == 0x10){
        if(printArg) printf ("Underlying architecture is little endian. \n");
        return 0;
    }
    else{
        if(printArg) printf ("Underlying architecture is big endian. \n");
        return 1;
    }
    return 0;
}
```
## ptrAndarray
```clike=
// 2D array 紀錄指標移動的範例
// 跟指標比較近的加號是使指標往下一個column
float farr[4][5] = {
	{  1,2,3,4,5  },
	{  6,7,8,9,10 },
	{ 11,12,13,14,15 },
	{ 16,17,18,19,20 }
};
void testprint(float *ptr){
	printf("test: %0.3f,%0.3f,%0.3f,%0.3f\n",ptr[0],ptr[1],ptr[2],ptr[3]);
}
int main(){
	testprint(*farr);
	testprint(*(farr+1));
	testprint((*farr+1));
	
	return 0;
}
//test: 1.000,2.000,3.000,4.000
//test: 6.000,7.000,8.000,9.000
//test: 2.000,3.000,4.000,5.000
```
## string_parser
```clike=
//紀錄引數用法以及scanf,sscanf,scanf遇到空格就會結束，可以用這個範例來改善問題
int checkArgMode(int argc,char **argv){
  int opt
  char *optstring = "abc:";//-a,-b都不會帶參數，-c會帶一個參數
  while((opt = getopt(argc,argv,optstring)) != -1){
      switch(opt){
          case "a":
              printf("here is -a\n");
              return 1;
          case "b":
              printf("here is -b\n");
              return 1;
          case "d":
              printf("here is -d\n");
              return 1;
    }
  }
  return 0;
}
int main(int argc,char **argv){
  int rev = checkArgMode(argc,argv);
  char buf[64];
  char cmd;
  int t1,t2;
  printf("checkArgMode: \n",rev); 
  printf("scanf test\n");
  scanf("%[\n]",buf);
  sscanf(buf,"%c %d %d\n",&cmd,&t1,&t2);
  return 0; 
}
```

# 常見的題目
## 同步問題
* 使用原則
    * 盡量減少critical section的處理時間
* spinlock\mutex\samphone差異與使用時機
* atomic
    * 在程式加入這個宣告綴詞可以讓"read-modify-write"的命令包起來，只讓一個CPU取得memory bus使用權
* KERNEL SPACE內的程式是共享的(記憶體空間也是)，在多CPU使用下只有一個thread可以進到CS
* semaphore 
    * 有人進到CS之後，其他人沒拿到KEY，會在外面SLEEP，CPU就先做其他事，KEY被釋放就會喚醒一個context出來工作
    * Down() 進到Sleep，送signal都不會結束甚至Ctrl+c or kill，一直到取得sem
    * down_interruptibale()可以被解除
    * 一般都使用down()，down_interruptibale()通常用在socket interface

    * spinlock就原地等待，保證每次都鎖的到，多核好用   
* completion
    * 呼叫wait_completion會一直sleep到呼叫completion()
    * 如果進到呼叫wait_completion前就completion()也不會鎖死
* 在irq裡面不能用sem跟completion(類sleep都不行)，所以進到irq之前要先禁止中斷