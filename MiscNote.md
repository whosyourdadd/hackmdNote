# MiscNote

## 5G NR
* https://opencores.org/projects
* https://github.com/catkira
* https://github.com/catkira/open5G_phy
* https://github.com/zulfadlizainal
* https://github.com/jg-fossh/IIR_FILTER/

## UVM
* https://github.com/mjhborja/hello_world_uvm

## C_Misc

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
