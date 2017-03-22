cmake_minimum_required(VERSION 3.0)

#Project name
project(clean_project)

# Set paths & system files
set(STM_DIR STM32F4xx_DSP_StdPeriph_Lib_V1.8.0)
set(STM_SRC ${STM_DIR}/Libraries/STM32F4xx_StdPeriph_Driver/src)
set(STM_LNK ${PROJECT_SOURCE_DIR}/stm32_flash.ld)
set(STM_ASM ${PROJECT_SOURCE_DIR}/startup_stm32f4xx.s)
set(STM_SYS ${PROJECT_SOURCE_DIR}/system_stm32f4xx.c)
set(STM_PRPH
  ${STM_SRC}/stm32f4xx_adc.c
  ${STM_SRC}/stm32f4xx_can.c
  ${STM_SRC}/stm32f4xx_crc.c
  ${STM_SRC}/stm32f4xx_cryp.c
  ${STM_SRC}/stm32f4xx_dac.c
  ${STM_SRC}/stm32f4xx_dbgmcu.c
  ${STM_SRC}/stm32f4xx_dcmi.c
  ${STM_SRC}/stm32f4xx_dma.c
  ${STM_SRC}/stm32f4xx_exti.c
  ${STM_SRC}/stm32f4xx_flash.c
  ${STM_SRC}/stm32f4xx_fsmc.c
  ${STM_SRC}/stm32f4xx_hash.c
  ${STM_SRC}/stm32f4xx_gpio.c
  ${STM_SRC}/stm32f4xx_i2c.c
  ${STM_SRC}/stm32f4xx_iwdg.c
  ${STM_SRC}/stm32f4xx_pwr.c
  ${STM_SRC}/stm32f4xx_rcc.c
  ${STM_SRC}/stm32f4xx_rng.c
  ${STM_SRC}/stm32f4xx_rtc.c
  ${STM_SRC}/stm32f4xx_sdio.c
  ${STM_SRC}/stm32f4xx_spi.c
  ${STM_SRC}/stm32f4xx_syscfg.c
  ${STM_SRC}/stm32f4xx_tim.c
  ${STM_SRC}/stm32f4xx_usart.c
  ${STM_SRC}/stm32f4xx_wwdg.c
  )


# Configuration for cross compiling
set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_C_FLAGS "-O0 -std=c11 -Warray-bounds -mlittle-endian -mthumb -mcpu=cortex-m4 -mthumb-interwork -mfloat-abi=hard -mfpu=fpv4-sp-d16 --specs=rdimon.specs -ffreestanding -nostdlib")
set(CMAKE_C_COMPILER arm-none-eabi-gcc)
set(OBJ_COPY arm-none-eabi-objcopy)
set(CMAKE_SHARED_LIBRARY_LINK_C_FLAGS)
set(CMAKE_BUILD_TYPE Debug)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
# set(CMAKE_VERBOSE_MAKEFILE ON)
add_definitions(-DSTM32F40_41xxx -DUSE_STDPERIPH_DRIVER -DARM_MATH_CM4)
set(CMAKE_EXE_LINKER_FLAGS "-T ${STM_LNK}")


# Inlcude directories for ptoject
include_directories(
  ${STM_DIR}/Libraries/CMSIS/Include
  ${STM_DIR}/Libraries/CMSIS/Device/ST/STM32F4xx/Include
  ${STM_DIR}/Libraries/STM32F4xx_StdPeriph_Driver/inc
  ${PROJECT_SOURCE_DIR}
  )

# Collect source files
file(GLOB SOURCES "*.c")
set(PROJECT_SOURCES ${SOURCES} ${STM_PRPH} ${STM_ASM} ${STM_SYS})

# Build command
set_property(SOURCE ${STM_ASM} PROPERTY LANGUAGE C)
add_executable(${PROJECT_NAME}.elf ${PROJECT_SOURCES} ${STM_LNK})
add_custom_command(TARGET ${PROJECT_NAME}.elf
  COMMAND ${OBJ_COPY} --output-format=binary ${PROJECT_NAME}.elf ${PROJECT_NAME}.bin
  DEPENDS ${PROJECT_NAME}.elf
  COMMENT "Objcopying ${PROJECT_NAME}.elf to ${PROJECT_NAME}.bin")
set_directory_properties(PROPERTIES ADDITIONAL_MAKE_CLEAN_FILES ${PROJECT_NAME}.bin)

# Flash commad (st-flash required)
add_custom_target(flash COMMAND st-flash write ${PROJECT_NAME}.bin 0x8000000)
