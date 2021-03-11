CC      = riscv64-unknown-elf-gcc
OBJCOPY = riscv64-unknown-elf-objcopy
OBJDUMP = riscv64-unknown-elf-objdump
READELF = riscv64-unknown-elf-readelf
ELFSIZE = riscv64-unknown-elf-size

CONTRIBDIR = $(abspath ../..)
LWIPDIR    = $(CONTRIBDIR)/../src
DEVELOPDIR = $(CONTRIBDIR)/../..
EXEDIR     = ./exe
OBJDIR     = $(EXEDIR)/obj

BOARD  ?= agrv2k_test
DEVICE ?= AgRV2K
TARGET ?= agm
EXE   = $(EXEDIR)/$(TARGET)

all:
	make TARGET=agm $(EXE).elf $(EXE).bin $(EXE).readelf $(EXE).dump
.PHONY: all

ISA    ?= rv32imafc
ABI    ?= ilp32f
CFLAGS += -march=$(ISA) -mabi=$(ABI)
CFLAGS += -Wall -DDEFINE_MALLOC -DDEFINE_FREE
CFLAGS += -fdata-sections -ffunction-sections
CFLAGS += -fsingle-precision-constant
CFLAGS += -I.  -I./test -I$(CONTRIBDIR) -I$(LWIPDIR)/include

ifeq ($(OPT), )
CFLAGS += -g3 -Og -DLWIP_DEBUG
else
CFLAGS += -O3
endif

include $(CONTRIBDIR)/Filelists.mk
include $(LWIPDIR)/Filelists.mk
LWIPFILES = $(LWIPNOAPPSFILES) $(ARCHFILES)
APPFILES  = $(CONTRIBAPPFILES) $(LWIPAPPFILES)

SRC_C = $(LWIPFILES) $(APPFILES)
SRC_C += \
  $(DEVELOPDIR)/misc/syscalls.c \
  $(wildcard $(DEVELOPDIR)/alta/*.c) \
  $(wildcard $(DEVELOPDIR)/boards/$(BOARD)/*.c) \
  $(wildcard ./*.c) \
  $(wildcard ./test/*.c)

ifneq ($(FREERTOS), )
CFLAGS += -I$(CONTRIBDIR)/ports/freertos/include
SRC_C  += $(CONTRIBDIR)/ports/freertos/*.c
endif

LOGGER ?= UART0
ifeq ($(LOGGER), RTT)
  SRC_C += $(DEVELOPDIR)/misc/segger/RTT/SEGGER_RTT.c
  CFLAGS += -DSEGGER_RTT
else ifneq (, $(findstring UART, $(LOGGER)))
  CFLAGS += -DLOGGER_UART=$(subst UART,,$(LOGGER))
endif

CFLAGS += -I$(DEVELOPDIR)/alta -I$(DEVELOPDIR)/misc -I$(DEVELOPDIR)/boards/$(BOARD)
CFLAGS += -Wno-c90-c99-compat
CFLAGS += -Wno-c++-compat
CFLAGS += -Wno-pedantic
CFLAGS += -Wno-unused-value
CFLAGS += -Wno-missing-prototypes
CFLAGS += -Wno-unused-parameter
CFLAGS += -Wno-nested-externs
CFLAGS += -Wno-pointer-arith
CFLAGS += -Wno-cast-qual
CFLAGS += -Wno-unused-label
CFLAGS += -Wno-unused-but-set-variable
CFLAGS += -Wno-unused-variable
CFLAGS += -Wno-implicit-fallthrough
CFLAGS += -Wno-switch-default
CFLAGS += -Wno-strict-aliasing

BOOT_MODE ?= FLASH
SRC_S := $(DEVELOPDIR)/misc/crt.S
LDSCRIPT := $(DEVELOPDIR)/misc/devices/$(DEVICE)_$(BOOT_MODE).ld
LDFLAGS += -march=$(ISA) -mabi=$(ABI)
LDFLAGS += $(addprefix -T, $(LDSCRIPT))
LDFLAGS += -Wl,--gc-sections -Wl,-L$(DEVELOPDIR)/misc/devices
LDFLAGS += $(SRC_S) -static -nostartfiles -lc -lm -lgcc

OBJ := $(addprefix $(OBJDIR)/, $(SRC_C:.c=.o))
OBJ_DIRS = $(sort $(dir $(OBJ)))
$(OBJ) : | $(OBJ_DIRS) # Order only prerequisites
$(OBJ_DIRS):
	mkdir -p $@

DEPS := $(OBJ:.o=.d)

-include $(DEPS)

$(OBJDIR)/%.o: %.c Makefile
	$(CC) $(CFLAGS) -c $< -o $@ -MT $@ -MMD

clean:
	rm -rf $(EXEDIR)

$(EXE).elf: $(OBJ) $(SRC_S) $(LDSCRIPT)
	$(CC) -o $@ -Wl,--start-group $(OBJ) -Wl,--end-group $(LDFLAGS)
	$(ELFSIZE) $@

%.bin: %.elf
	$(OBJCOPY) -O binary $< $@

%.dump: %.elf
	$(OBJDUMP) -d $< > $@

%.readelf: %.elf
	$(READELF) -a $< > $@

