# --- build configuration ---

# one of: img, lib
BUILD_TYPE  = lib

# one of: debug, release
BUILD_PROFILE     = debug

MCU         = atmega168
FREQUENCY   = 8000000

EXTRAINCDIRS =
EXTRALIBDIRS =

# --- build environment variables ---

TARGET = $(lastword $(subst /, ,$(CURDIR)))
ifeq ($(BUILD_TYPE),lib)
	LIBNAME = $(lastword $(subst /, ,$(CURDIR))).a
endif

SRC_DIR = ./src
INC_DIR = ./include
SRC = $(wildcard $(SRC_DIR)/*.c)

BUILD_ROOT_DIR = build
BUILD_DIR = $(BUILD_ROOT_DIR)/$(BUILD_PROFILE)

OBJ_DIR= $(BUILD_DIR)
OBJ_DUMMY = $(SRC:$(SRC_DIR)%=$(OBJ_DIR)%)
OBJ = $(OBJ_DUMMY:.c=.o)
LST = $(OBJ_DIR)/$(TARGET).lst

# --- programs variables ---

AR = avr-ar rcs
AVRDUDE = avrdude
CC = avr-gcc
NM = avr-nm
OBJCOPY = avr-objcopy
OBJDUMP = avr-objdump
SIZE = avr-size

# --- tools parameter variables ---

# optimization
OPT = s
# debug type
DEBUG = dwarf-2
# C standard
CSTANDARD = -std=gnu99
# C defines
CDEFS = -DF_CPU=$(FREQUENCY)UL

CFLAGS = -g$(DEBUG)
CFLAGS += $(CDEFS)
CFLAGS += -O$(OPT)
CFLAGS += $(CSTANDARD)
CFLAGS += -funsigned-char
CFLAGS += -funsigned-bitfields
CFLAGS += -fpack-struct
CFLAGS += -fshort-enums
CFLAGS += -Wall
CFLAGS += -Wstrict-prototypes
CFLAGS += -Wa,-adhlns=$(LST)
CFLAGS += $(patsubst %,-I%,$(EXTRAINCDIRS))

GENDEPFLAGS = -MMD -MP -MF .dep/$(@F).d

ALL_CFLAGS = -mmcu=$(MCU) -I$(INC_DIR) $(CFLAGS) $(GENDEPFLAGS)

PRINTF_LIB_MIN = -Wl,-u,vfprintf -lprintf_min
PRINTF_LIB_FLOAT = -Wl,-u,vfprintf -lprintf_flt
# using standard printf library
PRINTF_LIB = 

SCANF_LIB_MIN = -Wl,-u,vfscanf -lscanf_min
SCANF_LIB_FLOAT = -Wl,-u,vfscanf -lscanf_flt
# using standard scanf library
SCANF_LIB = 

MATH_LIB = -lm

LDFLAGS = $(strip $(PRINTF_LIB) $(SCANF_LIB) $(MATH_LIB))
LDFLAGS += -Wl,-Map=$(OBJ_DIR)/$(TARGET).map,--cref
LDFLAGS += $(patsubst %,-L%,$(EXTRALIBDIRS))

FORMAT = ihex

AVRDUDE_PROGRAMMER = usbasp
AVRDUDE_PORT = com1
AVRDUDE_WRITE_FLASH = -U flash:w:$(TARGET).hex
#AVRDUDE_WRITE_EEPROM = -U eeprom:w:$(TARGET).eep
#AVRDUDE_ERASE_COUNTER = -y
#AVRDUDE_NO_VERIFY = -V
AVRDUDE_VERBOSE = -v

AVRDUDE_FLAGS = -p $(MCU) -P $(AVRDUDE_PORT) -c $(AVRDUDE_PROGRAMMER)
AVRDUDE_FLAGS += $(AVRDUDE_NO_VERIFY)
AVRDUDE_FLAGS += $(AVRDUDE_VERBOSE)
AVRDUDE_FLAGS += $(AVRDUDE_ERASE_COUNTER)

ELFSIZE = $(SIZE) --mcu=$(MCU) --format=avr $(OBJ_DIR)/$(TARGET).elf
HEXSIZE = $(SIZE) --target=$(FORMAT) $(OBJ_DIR)/$(TARGET).hex

# --- rules ---

.DEFAULT_GOAL: all
vpath %.c $(SRC_DIR)
vpath %.h $(INC_DIR)
vpath %.o $(OBJ_DIR)

ifeq ($(BUILD_TYPE), lib)
all: build-info prepare lib
else ifeq ($(BUILD_TYPE), img)
all: build-info prepare img
else
all:
	$(error no valid build type - use either img or lib)
endif

build-info:
	@echo
	@echo "+++ build information +++"
	@echo
	@echo "   build type        -> $(BUILD_TYPE)"
	@echo "   target name       -> $(TARGET)"
	@echo "   library name      -> $(LIBNAME)"
	@echo "   compiler info     -> $(shell $(CC) --version)"
	@echo "   source directory  -> $(SRC_DIR)"
	@echo "   main source file  -> $(SRC)"
	@echo "   build directory   -> $(BUILD_DIR)"
	@echo "   objects directory -> $(OBJ_DIR)"
	@echo "   main object file  -> $(OBJ)"
	@echo "   listing file      -> $(LST)"
	@echo "   CFLAGS            -> $(CFLAGS)"
	@echo "   ALL_CFLAGS        -> $(ALL_CFLAGS)"
	@echo "   LDFLAGS           -> $(LDFLAGS)"
	@echo "   AVRDUDE_FLAGS     -> $(AVRDUDE_FLAGS)"
	@echo

prepare:
	@echo "+++ prepare build +++"
	@echo
	if [ ! -d $(BUILD_ROOT_DIR) ]; then mkdir -p $(BUILD_DIR); fi
	@echo
	@echo "+++ start build +++"
	@echo

elfsize:
	@if test -f  $(OBJ_DIR)/$(TARGET).elf; then echo "elf size:"; echo; $(ELFSIZE); \
	2>/dev/null; fi

hexsize:
	@if test -f  $(OBJ_DIR)/$(TARGET).hex; then echo "hex size: "; echo; $(HEXSIZE); \
	2>/dev/null; fi

lib: $(OBJ_DIR)/$(LIBNAME)
	
.SECONDARY : $(OBJ_DIR)/$(LIBNAME)
.PRECIOUS : $(OBJ)
$(OBJ_DIR)/$(LIBNAME): $(OBJ)
	$(AR) $@ $(OBJ)

img:  elf elfsize hex hexsize eep lss sym

elf: $(OBJ_DIR)/$(TARGET).elf

.SECONDARY : $(OBJ_DIR)/$(TARGET).elf
.PRECIOUS : $(OBJ)
$(OBJ_DIR)/$(TARGET).elf: $(OBJ)
	$(CC) $(ALL_CFLAGS) $^ --output $@ $(LDFLAGS)

hex: $(OBJ_DIR)/$(TARGET).hex

$(OBJ_DIR)/$(TARGET).hex: $(OBJ_DIR)/$(TARGET).elf
	$(OBJCOPY) -O $(FORMAT) -R .eeprom -R .fuse -R .lock -R .signature $< $@

eep: $(OBJ_DIR)/$(TARGET).eep

$(OBJ_DIR)/$(TARGET).eep: $(OBJ_DIR)/$(TARGET).elf
	-$(OBJCOPY) -j .eeprom --set-section-flags=.eeprom="alloc,load" \
	--change-section-lma .eeprom=0 --no-change-warnings -O $(FORMAT) $< $@ || exit 0

lss: $(OBJ_DIR)/$(TARGET).lss

$(OBJ_DIR)/$(TARGET).lss: $(OBJ_DIR)/$(TARGET).elf
	$(OBJDUMP) -h -S -z $< > $@

sym: $(OBJ_DIR)/$(TARGET).sym

$(OBJ_DIR)/$(TARGET).sym: $(OBJ_DIR)/$(TARGET).elf
	$(NM) -n $< > $@

$(OBJ_DIR)/%.o : %.c
	$(CC) -c $(ALL_CFLAGS) $< -o $@ 

program: all
	@echo
	@echo "+++ transfer program to device +++"
	@echo
ifeq ($(BUILD_TYPE), img)
	$(AVRDUDE) $(AVRDUDE_FLAGS) $(AVRDUDE_WRITE_FLASH) $(AVRDUDE_WRITE_EEPROM)
else
	$(error program not supported for this build type)
endif

distclean:
	@echo "+++ cleanup project +++"
	@echo
	if [ -d $(BUILD_ROOT_DIR) ]; then rm -fr $(BUILD_ROOT_DIR); fi
	if [ -d .dep ]; then rm -fr .dep; fi
	@echo
	
.PHONY: all lib img elf hex eep lss sym build-info prepare elfsize hexsize distclean

-include $(shell mkdir -p .dep) $(wildcard .dep/*)