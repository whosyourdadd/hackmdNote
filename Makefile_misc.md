# makefile_misc




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