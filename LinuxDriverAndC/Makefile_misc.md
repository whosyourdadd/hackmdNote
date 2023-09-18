# makefile_misc
## 參考資料
* makefile
    * [Makefile編寫要點](https://hackmd.io/@zero6992/r1ePe1lFt)
    * [Makefile 語法和示範](/XdF11cBIR8aahe8bxU7gfw)
    * [手把手教你寫makefile](https://www.readfog.com/a/1685591356715667456)
        * gcc -o是決定輸出檔的名稱
        * gcc -c是指輸出obj未做link
        * gcc -DDEBUG是preprocessor的時候使用DEBUG這個define 


## 範例

```shell
TARGET = TDQ16_unittest
#TARGET = TDQ32_unittest
BIN_TARGET = $(BIN_DIR)/$(TARGET)

INC_DIR = ./include
BIN_DIR = ./bin
SRC_DIR = ./src
OBJ_DIR = ./obj

SRC = $(wildcard $(SRC_DIR)/*.c)      # /*/
OBJ = $(patsubst %.c, $(OBJ_DIR)/%.o, $(notdir $(SRC)))

#CC = gcc
CC = aarch64-linux-gnu-gcc
LD = -lm -pthread
CFLAGS = -Wall 
		
DEBUG ?= 0
#ifeq ($(DEBUG), 1)
#	@echo "DEBUG mod $(DEBUG)"
#else
#	@echo "DEBUG mod $(DEBUG)"
#endif


$(BIN_TARGET).elf: $(OBJ)
	$(CC) $(LD) $(OBJ) -o $@
    
 
$(OBJ_DIR)/%.o:$(SRC_DIR)/%.c
	$(CC) -I$(INC_DIR) -c -o $@ $<

.PHONY: clean
clean:
	find $(OBJ_DIR) -name *.o -exec rm -rf {} \;
	rm $(BIN_DIR)/$(TARGET).elf

#%.o: %.c
#	@echo " cross compiler "
#	$(CC) $(CFLAG) -c -o $@ $<


#TDQ16_UNITTEST.elf: $(TDQ16_UT_OBJS)
#	@echo "2 make 16_UNITTEST.elf"
#	$(CC) $(LD) -o $@ $^

#print: $(wildcard *.c)
#	ls -la $?

#.PHONY: clean Test123
#clean:
#	@echo "clean obj"
#	-rm *.o	
#	-rm *.elf
```
* 自動化變數
  * $@ 工作目標檔名
  * $< 第一個必要條件的檔名
  * $^ 所有必要條件的檔名，並以空格隔開這些檔名 (這份清單已移除重複的檔名)
  * $* 工作目標的主檔名
* % 萬用字元
* 
```shell=
#CC = gcc
CC = aarch64-linux-gnu-gcc
LD = -lm -pthread
CFLAGS = -Wall 
OS_TYPE := $(shell uname -s)

TDQ16_UT_OBJS = 	TDQ_unittest.o \
		BS_main_16.o \
		BS_RX_16.o \
		BS_formula.o
		
DEBUG ?= 0
#ifeq ($(DEBUG), 1)
#	@echo "DEBUG mod $(DEBUG)"
#else
#	@echo "DEBUG mod $(DEBUG)"
#endif


TDQ16_UNITTEST: TDQ16_UNITTEST.elf


TEMP_TEST: 	
	@echo "just for test "
	@echo $(info Detected OS type is "$(OS_TYPE)")
	@echo "DEBUG: $(DEBUG)"

%.o: %.c
	@echo " cross compiler "
	$(CC) $(CFLAG) -c -o $@ $<


TDQ16_UNITTEST.elf: $(TDQ16_UT_OBJS)
	@echo "2 make TDQ16_UNITTEST.elf"
	$(CC) $(LD) -o $@ $^

print: $(wildcard *.c)
	ls -la $?

.PHONY: clean 
clean:
	@echo "clean obj"
	-rm *.o	
	-rm *.elf

```
