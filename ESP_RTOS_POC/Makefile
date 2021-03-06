#############################################################
#
# Root Level Makefile ESP8266 + RTOS
#
# (c) by CHERTS <sleuthhound@gmail.com>
#
#############################################################

BUILD_BASE	= build
FW_BASE		= firmware

TOOLS_ROOT ?= c:/Espressif

# Base directory for the compiler
XTENSA_TOOLS_ROOT ?= c:/Espressif/xtensa-lx106-elf/bin

# base directory of the ESP8266 SDK package, absolute
SDK_BASE	?= c:/Espressif/ESP8266_RTOS_SDK
SDK_TOOLS	?= c:/Espressif/utils

# esptool path and port
ESPTOOL ?= $(SDK_TOOLS)/esptool.exe
ESPPORT ?= COM5

# name for the target project
TARGET		= app

# which modules (subdirectories) of the project to include in compiling
# MODULES = third_party/freertos
MODULES	= user system appinit SmingCore RTOS_EC Wiring SmingCore/Platform SmingCore/Network $(filter %/, $(wildcard Services/*/))  $(filter %/, $(wildcard Libraries/*/))
# MODULES	= user appinit 
# MODULES	+=	$(SDK_BASE)\third_party\freertos
MODULES		+= rboot/appcode

EXTRA_INCDIR = include 
EXTRA_INCDIR += $(SDK_BASE)/include/freertos
EXTRA_INCDIR += $(SDK_BASE)/include/espressif
EXTRA_INCDIR += $(SDK_BASE)/include/espressif/esp8266 

#LIBS       = cirom gccirom hal phy pp net80211 wpa main freertos lwip udhcp
# linking libgccirom.a instead of libgcc.a causes reset when working with flash memory (ie spi_flash_erase_sector)
LIBS		= microc gcc hal phy pp net80211 wpa crypto main freertos lwip minic pwm

# compiler flags using during compilation of source files
#CFLAGS		= -g -save-temps -Os -Wpointer-arith -Wundef -Werror -Wl,-EL -fno-inline-functions -nostdlib -mlongcalls -mtext-section-literals -mno-serialize-volatile -D__ets__ -DICACHE_FLASH
CFLAGS		= -Os -g -Wpointer-arith -Wundef -Werror -Wl,-EL -nostdlib -mlongcalls -mtext-section-literals -finline-functions -fdata-sections -ffunction-sections -D__ets__ -DICACHE_FLASH -DARDUINO=106 $(USER_CFLAGS)
# CFLAGS		= -Os -g -Wpointer-arith -Wundef -Wl,-EL -nostdlib -mlongcalls -mtext-section-literals -finline-functions -fdata-sections -ffunction-sections -D__ets__ -DICACHE_FLASH -DARDUINO=106 $(USER_CFLAGS)


WIFI_SSID ?= ""
WIFI_PWD ?= ""
ifneq ($(WIFI_SSID), "")
	CFLAGS += -DWIFI_SSID=\"$(WIFI_SSID)\"
endif
ifneq ($(WIFI_PWD), "")
	CFLAGS += -DWIFI_PWD=\"$(WIFI_PWD)\"
endif

CFLAGS		+= -DNDEBUG=1
CFLAGS		+= -DLWIP_DEBUG=0
# CXXFLAGS	= $(CFLAGS) -std=c++11 -fno-rtti -fno-exceptions
CXXFLAGS	= $(CFLAGS) -fno-rtti -fno-exceptions -std=c++11 -felide-constructors -Wno-literal-suffix
# linker flags used to generate the main object file
LDFLAGS		= -nostdlib -Wl,--no-check-sections -u call_user_start -Wl,-static -Wl,-Map=$(BUILD_BASE)/firmware.map

# linker script used for the above linkier step
#LD_SCRIPT	= eagle.app.v6.ld
LD_SCRIPT = eagle.app.v6.hr.ld
# various paths from the SDK used in this project
SDK_LIBDIR	= lib
SDK_LDDIR	= ld
SDK_INCDIR	= extra_include include include/espressif include/json include/udhcp include/lwip include/lwip/lwip include/lwip/ipv4 include/lwip/ipv6 include/freertos

# select which tools to use as compiler, librarian and linker
CC		:= $(XTENSA_TOOLS_ROOT)/xtensa-lx106-elf-gcc
CXX		:= $(XTENSA_TOOLS_ROOT)/xtensa-lx106-elf-g++
AR		:= $(XTENSA_TOOLS_ROOT)/xtensa-lx106-elf-ar
LD		:= $(XTENSA_TOOLS_ROOT)/xtensa-lx106-elf-gcc
OBJCOPY := $(XTENSA_TOOLS_ROOT)/xtensa-lx106-elf-objcopy
OBJDUMP := $(XTENSA_TOOLS_ROOT)/xtensa-lx106-elf-objdump

SRC_DIR		:= $(MODULES)
BUILD_DIR	:= $(addprefix $(BUILD_BASE)/,$(MODULES))

USER_LIBDIR	:= compiler/lib
SDK_LIBDIR	:= $(addprefix $(SDK_BASE)/,$(SDK_LIBDIR))
SDK_INCDIR	:= $(addprefix -I$(SDK_BASE)/,$(SDK_INCDIR))

SRC		:= $(foreach sdir,$(SRC_DIR),$(wildcard $(sdir)/*.c*))
C_OBJ		:= $(patsubst %.c,%.o,$(SRC))
CXX_OBJ		:= $(patsubst %.cpp,%.o,$(C_OBJ))
OBJ		:= $(patsubst %.o,$(BUILD_BASE)/%.o,$(CXX_OBJ))
LIBS		:= $(addprefix -l,$(LIBS))
APP_AR		:= $(addprefix $(BUILD_BASE)/,$(TARGET)_app.a)
TARGET_OUT	:= $(addprefix $(BUILD_BASE)/,$(TARGET).out)

LD_SCRIPT	:= $(addprefix -Tcompiler/ld/,$(LD_SCRIPT))

INCDIR	:= $(addprefix -I,$(SRC_DIR))
EXTRA_INCDIR	:= $(addprefix -I,$(EXTRA_INCDIR))
MODULE_INCDIR	:= $(addsuffix /include,$(INCDIR))

V ?= $(VERBOSE)
ifeq ("$(V)","1")
Q :=
vecho := @true
else
Q := @
vecho := @echo
endif

vpath %.c $(SRC_DIR)
vpath %.cpp $(SRC_DIR)

#$(vecho) "Compiler = " $(CC) 

define compile-objects
$1/%.o: %.c
	$(vecho) "CC $$<" 
	$(Q) $(CC) $(INCDIR) $(MODULE_INCDIR) $(EXTRA_INCDIR) $(SDK_INCDIR) $(CFLAGS)  -c $$< -o $$@
$1/%.o: %.cpp
	$(vecho) "C+ $$<"
	$(Q) $(CXX) $(INCDIR) $(MODULE_INCDIR) $(EXTRA_INCDIR) $(SDK_INCDIR) $(CXXFLAGS)  -c $$< -o $$@
endef

.PHONY: all checkdirs clean firmware flashonefile flash flashinit rebuild

all: checkdirs $(APP_AR)

$(APP_AR): $(OBJ)
	$(vecho) "AR $@"
	$(Q) $(AR) cru $@ $^
	$(vecho) "Installing libsming"
	$(Q) cp -r $(APP_AR) $(USER_LIBDIR)/libsming.a
	$(vecho) "Done"

spiffy: spiffy/spiffy

spiffy/spiffy:
	$(vecho) "Making spiffy utility"
	$(Q) $(MAKE) --no-print-directory -C spiffy V=$(V)
	$(vecho) "Done"

checkdirs: $(BUILD_DIR) $(FW_BASE)

$(BUILD_DIR):
	$(Q) mkdir -p $@

$(FW_BASE):
	$(Q) mkdir -p $@

rebuild: clean all

clean:
	$(Q) rm -f $(USER_LIBDIR)/libsming.a
	$(Q) rm -rf $(BUILD_BASE)
	$(Q) rm -rf $(FW_BASE)

$(foreach bdir,$(BUILD_DIR),$(eval $(call compile-objects,$(bdir))))
