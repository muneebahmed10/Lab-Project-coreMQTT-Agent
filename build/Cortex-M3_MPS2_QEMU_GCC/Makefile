OUTPUT_DIR := ./output
LIB_OUTPUT_DIR := ./output/lib
IMAGE := RTOSDemo.elf
CC = arm-none-eabi-gcc
LD = arm-none-eabi-gcc

SUB_MAKEFILE_DIR = ./library-makefiles

CFLAGS += $(INCLUDE_DIRS) -nostartfiles -ffreestanding -mthumb -mcpu=cortex-m3 \
		  -Wall -Wextra -g3 -O0 -ffunction-sections -fdata-sections \
		  -MMD -MP -MF"$(@:%.o=%.d)" -MT $@

#must be the first include to ensure the correct FreeRTOSConfig.h is used.
INCLUDE_DIRS += -I./target-specific-source
INCLUDE_DIRS += -I./target-specific-source/CMSIS
INCLUDE_DIRS += -I./../../lib/FreeRTOS/utilities/logging
INCLUDE_DIRS += -I./../../source/configuration-files
INCLUDE_DIRS += -I./../../lib/ThirdParty/mbedtls/include

#FreeRTOS specific library includes
include $(SUB_MAKEFILE_DIR)/freertos-kernel.mk
include $(SUB_MAKEFILE_DIR)/freertos-plus-tcp.mk

#Standalone libraries, with the transport interface to link the coreMQTT library
#to the FreeRTOS+TCP library.
include $(SUB_MAKEFILE_DIR)/coremqtt-agent.mk
include $(SUB_MAKEFILE_DIR)/corejson.mk
include $(SUB_MAKEFILE_DIR)/transport-interface.mk

#AWS IoT service client libraries
include $(SUB_MAKEFILE_DIR)/aws-iot-ota.mk
include $(SUB_MAKEFILE_DIR)/aws-iot-shadow.mk
include $(SUB_MAKEFILE_DIR)/aws-iot-device-defender.mk

#Utility libraries
include $(SUB_MAKEFILE_DIR)/mbedtls-utils.mk #Implementation of the Mbed TLS port layer for FreeRTOS.
include $(SUB_MAKEFILE_DIR)/backoff-algorithm.mk

#Third party libraries
include $(SUB_MAKEFILE_DIR)/tinycbor.mk
include $(SUB_MAKEFILE_DIR)/mbedtls.mk

#Application source files
APPLICATION_DIR += ./../../source
BUILD_SPECIFIC_FILES += ./target-specific-source
VPATH += $(APPLICATION_DIR) $(APPLICATION_DIR)/subscription-manager $(APPLICATION_DIR)/demo-tasks $(BUILD_SPECIFIC_FILES)
INCLUDE_DIRS += -I$(APPLICATION_DIR)/subscription-manager -I./CMSIS -I$(BUILD_SPECIFIC_FILES)
SOURCE_FILES += $(wildcard $(APPLICATION_DIR)/*.c)
SOURCE_FILES += $(wildcard $(APPLICATION_DIR)/subscription-manager/*.c)
SOURCE_FILES += $(wildcard $(APPLICATION_DIR)/demo-tasks/*.c)
SOURCE_FILES += $(wildcard $(BUILD_SPECIFIC_FILES)/*.c)

#Create a list of object files with the desired output directory path.
OBJS = $(SOURCE_FILES:%.c=%.o)
OBJS_NO_PATH = $(notdir $(OBJS))
OBJS_OUTPUT = $(OBJS_NO_PATH:%.o=$(OUTPUT_DIR)/%.o)

#Create a list of dependency files with the desired output directory path.
DEP_FILES := $(SOURCE_FILES:%.c=$(OUTPUT_DIR)/%.d)
DEP_FILES_NO_PATH = $(notdir $(DEP_FILES))
DEP_OUTPUT = $(DEP_FILES_NO_PATH:%.d=$(OUTPUT_DIR)/%.d)

all: $(OUTPUT_DIR)/$(IMAGE)

%.o : %.c
$(OUTPUT_DIR)/%.o : %.c $(OUTPUT_DIR)/%.d Makefile
	$(CC) $(CFLAGS) -c $< -o $@

$(OUTPUT_DIR)/$(IMAGE): ./mps2_m3.ld $(OBJS_OUTPUT) Makefile
	$(LD) $(OBJS_OUTPUT) $(CFLAGS) -Xlinker --gc-sections -Xlinker -T ./mps2_m3.ld \
		-Xlinker -Map=$(OUTPUT_DIR)/RTOSDemo.map -specs=nano.specs \
		-specs=nosys.specs -specs=rdimon.specs -o $(OUTPUT_DIR)/$(IMAGE)

$(DEP_OUTPUT):
include $(wildcard $(DEP_OUTPUT))

clean:
	rm -f $(OUTPUT_DIR)/$(IMAGE) $(OUTPUT_DIR)/*.o $(OUTPUT_DIR)/*.d

#use "make print-[VARIABLE_NAME] to print the value of a variable generated by
#this makefile.
print-%  : ; @echo $* = $($*)

.PHONY: all clean


